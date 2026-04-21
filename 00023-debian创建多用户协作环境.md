---

## 📂 Debian 多用户协作与共享目录配置指南

### 1. 账号管理

创建新用户并赋予管理员权限。

* **创建用户**：
```bash
/usr/sbin/adduser claude

```


*注：交互过程中，密码必填，其余用户信息（房间号等）可直接按回车跳过。*
* **授权 Sudo 权限**：
```bash
/usr/sbin/usermod -aG sudo claude

```


*注：执行后，新用户 `claude` 即可使用 `sudo` 执行 root 命令。*

---

### 2. 公共共享目录设置

创建一个所有用户都能读写、且新建文件自动互通的目录。

* **创建与基础授权**：
```bash
mkdir /home/common_share
chmod 777 /home/common_share

```


* **配置 ACL (高级访问控制)**：
使用 ACL 确保该目录下**未来新建**的文件也对所有人开放读写。
```bash
# 设置目录当前的权限
setfacl -m u::rwx,g::rwx,o::rwx /home/common_share

# 设置默认权限（继承给以后新建的文件/文件夹）
setfacl -d -m u::rwx,g::rwx,o::rwx /home/common_share

```



---

### 3. 核心命令速查表

| 任务 | 命令 | 说明 |
| --- | --- | --- |
| **切换用户** | `su - username` | 切换到指定用户环境（`-` 表示加载用户环境变量）。 |
| **查看权限** | `getfacl /path/to/dir` | 查看比 `ls -l` 更详细的 ACL 权限信息。 |
| **路径修复** | `export PATH=$PATH:/usr/sbin` | 解决 `adduser` 等命令找不到的问题。 |
| **删除用户** | `deluser --remove-home username` | 删除用户及其家目录。 |

---

### ⚠️ 运维小贴士

1. **安全建议**：由于 `/home/common_share` 设置了 `777` 权限，任何用户都可以删除该目录下他人的文件。如果你希望**只能删除自己创建的文件**，请执行：
```bash
chmod +t /home/common_share

```


2. **环境变量**：你在操作中遇到了 `bash: adduser: 未找到命令`，这是因为 root 的 `PATH` 缺少了 `/usr/sbin`。建议检查 `/etc/profile` 或 `~/.bashrc` 确保路径完整。

