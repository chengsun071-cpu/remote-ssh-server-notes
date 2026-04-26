# VS Code Remote-SSH 连接超算服务器配置记录

## 一、文档说明

本文记录在 Windows 本地电脑上，使用 VS Code Remote-SSH 远程连接超算服务器的完整过程，包括：

- SSH 连接信息获取
- VS Code Remote-SSH 插件配置
- 密钥登录配置
- 远程打开服务器项目目录
- 常见问题排查
- VS Code Server 版本兼容问题解决
- Remote-SSH 与容器实例的区别

使用 Remote-SSH 后，可以直接在本地 VS Code 中编辑服务器上的代码文件，避免每次手动上传代码。

---

## 二、使用场景

传统远程开发方式通常是：

```text
本地改代码
↓
手动上传服务器
↓
服务器运行
↓
再下载日志或结果
```

这种方式容易出现以下问题：

- 本地代码和服务器代码不同步
- 忘记上传最新代码
- 覆盖服务器已有代码
- 实验目录和日志管理混乱
- 修改代码后不确定服务器是否已经更新

使用 VS Code Remote-SSH 后，流程变为：

```text
本地 VS Code
↓
Remote-SSH 连接服务器
↓
直接打开服务器项目目录
↓
直接修改服务器代码
↓
Ctrl + S 保存
↓
服务器直接运行最新代码
```

也就是说，VS Code 显示在本地电脑上，但实际打开和修改的是服务器上的文件。

---

## 三、准备工作

本地电脑需要安装：

- Visual Studio Code
- VS Code 插件：Remote - SSH
- Windows 自带 OpenSSH 客户端

服务器侧需要准备：

- SSH 登录地址
- SSH 端口
- 用户名
- 密钥文件或登录密码

示例连接信息格式如下：

```text
主机：<server_host>
端口：<ssh_port>
用户名：<username>
```

对应 SSH 命令格式为：

```bash
ssh <username>@<server_host> -p <ssh_port>
```

注意：  
不要在公开仓库中暴露自己的真实密码、密钥内容或验证码。

---

## 四、安装 Remote-SSH 插件

打开 VS Code，进入左侧扩展页面，搜索：

```text
Remote - SSH
```

安装 Microsoft 官方插件。

安装完成后，可以通过 VS Code 的命令面板进行远程连接配置。

打开命令面板的方法：

```text
Ctrl + Shift + P
```

命令面板可以理解为 VS Code 的“功能搜索框”。  
例如，要连接远程服务器，就搜索：

```text
Remote-SSH: Connect to Host
```

---

## 五、配置 SSH 连接

在 VS Code 中按：

```text
Ctrl + Shift + P
```

搜索并选择：

```text
Remote-SSH: Open SSH Configuration File
```

然后选择当前 Windows 用户的 SSH 配置文件：

```text
C:\Users\<Windows用户名>\.ssh\config
```

不要优先选择系统级配置文件：

```text
C:\ProgramData\ssh\ssh_config
```

在配置文件中添加如下内容：

```ssh-config
Host scnet
    HostName <server_host>
    User <username>
    Port <ssh_port>
    IdentityFile C:/Users/<Windows用户名>/.ssh/scnet_key
    IdentitiesOnly yes
```

配置项说明如下：

| 配置项 | 含义 |
|---|---|
| Host | 给服务器连接起一个别名 |
| HostName | 服务器主机地址 |
| User | 服务器用户名 |
| Port | SSH 端口 |
| IdentityFile | 本地私钥文件路径 |
| IdentitiesOnly yes | 指定只使用该密钥登录 |

如果平台提供密钥登录，建议将下载的密钥文件放到：

```text
C:\Users\<Windows用户名>\.ssh\
```

并改名为：

```text
scnet_key
```

这样配置文件更清晰。

---

## 六、连接服务器

SSH 配置保存后，在 VS Code 中按：

```text
Ctrl + Shift + P
```

搜索：

```text
Remote-SSH: Connect to Host
```

选择刚才配置的主机别名：

```text
scnet
```

首次连接时，VS Code 可能会提示是否信任该主机，选择：

```text
Continue
```

连接过程中，VS Code 会在服务器上安装远程组件：

```text
VS Code Server
```

该组件通常安装在服务器用户目录下：

```bash
~/.vscode-server
```

它的作用是让本地 VS Code 能够：

- 读取服务器文件
- 编辑服务器代码
- 打开服务器终端
- 在服务器环境中执行命令

---

## 七、打开服务器项目目录

连接成功后，VS Code 左下角会显示类似：

```text
SSH: scnet
```

此时说明 VS Code 已经连接到服务器。

然后点击：

```text
文件 → 打开文件夹
```

输入服务器项目路径，例如：

```bash
/public/home/<username>/Sunbc/dual_relation_ustc/v1
```

打开后，左侧文件树中显示的文件就是服务器上的真实文件，例如：

```text
train.py
dataset.py
model.py
flow_graph_builder.py
logs/
runs_*/
```

此时修改代码后，按：

```text
Ctrl + S
```

就是直接保存到服务器，不需要再手动上传。

---

## 八、确认当前是否处于服务器环境

在 VS Code 中打开终端：

```text
终端 → 新建终端
```

输入：

```bash
pwd
hostname
whoami
```

如果输出类似：

```text
/public/home/<username>/Sunbc/dual_relation_ustc/v1
login09
<username>
```

说明当前已经在服务器环境中。

如果显示的是：

```text
C:\Users\...
```

说明当前仍然是本地环境，并没有进入服务器。

---

## 九、常用服务器命令

进入项目目录：

```bash
cd /public/home/<username>/Sunbc/dual_relation_ustc/v1
```

激活 Conda 环境：

```bash
conda activate sun
```

查看当前路径：

```bash
pwd
```

查看当前目录文件：

```bash
ls
```

查看当前用户作业：

```bash
squeue -u <username>
```

查看日志：

```bash
tail -f logs/xxx.out
```

提交作业：

```bash
sbatch submit_all_ablation.sh
```

取消作业：

```bash
scancel <job_id>
```

查看 GPU：

```bash
nvidia-smi
```

---

## 十、问题记录：新版 VS Code Server 启动失败

连接过程中，可能出现 VS Code Server 下载完成但启动失败的问题。

日志中可能出现类似错误：

```text
GLIBC_2.27 not found
GLIBC_2.28 not found
GLIBCXX_3.4.21 not found
CXXABI_1.3.9 not found
```

这种情况说明：

```text
本地 VS Code 版本较新
↓
自动安装的 VS Code Server 也较新
↓
服务器登录节点系统库版本较旧
↓
VS Code Server 无法正常启动
```

这不是 SSH 地址错误，也不是密钥错误，而是本地 VS Code 版本与服务器系统库不兼容。

---

## 十一、解决方法：安装旧版 VS Code

如果服务器系统库较旧，可以安装旧版 VS Code，例如：

```text
VS Code 1.85.x
```

下载 Windows 版本时，需要选择：

```text
Windows x64 User Installer
```

不要选择：

```text
Windows ARM64
```

如果误下载 ARM64 版本，安装时会提示：

```text
这个程序只能在 ARM64 的 Windows 版本中进行安装
```

安装旧版 VS Code 后，建议关闭自动更新。

关闭方法：

```text
文件 → 首选项 → 设置
搜索 update mode
Update: Mode 设置为 none
```

这样可以避免 VS Code 自动升级后再次出现远程组件不兼容的问题。

---

## 十二、清理服务器上的旧 VS Code Server

更换本地 VS Code 版本后，需要清理服务器上之前安装失败的远程组件。

在服务器终端或网页 E-Shell 中执行：

```bash
rm -rf ~/.vscode-server ~/.vscode-server-cli
```

该命令只会删除 VS Code 远程组件，不会删除实验代码。

清理完成后，再重新使用旧版 VS Code 进行 Remote-SSH 连接。

---

## 十三、Remote-SSH 与容器实例的区别

超算平台通常还提供容器实例，例如：

```text
vscode-pytorch
jupyterlab-pytorch
```

需要区分 Remote-SSH、容器实例和 SLURM 作业。

| 类型 | 作用 | 是否需要排队 | 是否占用 GPU |
|---|---|---|---|
| Remote-SSH | 连接服务器、编辑代码、查看文件 | 通常不需要 GPU 排队 | 基本不占 |
| 容器实例 | 创建完整运行环境，如 VS Code/Jupyter | 需要排队 | 会占用 CPU/GPU/内存 |
| SLURM 作业 | 正式训练实验 | 需要排队 | 会占用 CPU/GPU/内存 |

Remote-SSH 更适合：

- 日常改代码
- 查看日志
- 管理项目文件
- 提交作业脚本

容器更适合：

- 图形化调试
- Jupyter 实验
- 临时测试代码

但是容器本身也需要申请资源。  
如果容器还在排队或未启动，就无法打开容器内的 VS Code 或 Jupyter。

---

## 十四、安全注意事项

不要把以下内容上传到 GitHub：

```text
密码
验证码
私钥内容
密钥文件
服务器登录凭证
```

以下文件也不要上传到公开仓库：

```text
*.pem
*.key
id_rsa
id_rsa.pub
scnet_key
*_rsakey*
```

可以在 `.gitignore` 中加入：

```gitignore
# SSH keys
*.pem
*.key
id_rsa
id_rsa.pub
scnet_key
*_rsakey*
```

技术文档中建议使用占位符：

```text
<username>
<server_host>
<ssh_port>
<key_path>
<Windows用户名>
```

不要直接公开自己的真实登录信息。

---

## 十五、最终工作流程

最终形成的远程开发方式为：

```text
本地 Windows
↓
VS Code Remote-SSH
↓
连接超算服务器
↓
打开服务器项目目录
↓
直接修改服务器代码
↓
Ctrl + S 保存
↓
使用 srun / sbatch 运行实验
↓
查看 logs 和 runs 结果
```

这种方式相比手动上传代码更加方便，也能减少本地代码和服务器代码不一致的问题。

---

## 十六、总结

VS Code Remote-SSH 的核心作用是：

```text
让本地 VS Code 直接操作服务器上的文件
```

它不是容器，也不是正式训练作业。  
它更像是一个远程编辑工具。

在实际使用中，需要注意：

- SSH 地址、端口、用户名要正确
- 密钥路径要配置正确
- 本地 VS Code 版本可能需要适配服务器系统环境
- 如果 VS Code Server 启动失败，可以尝试旧版 VS Code
- 不要把密钥、密码等敏感信息上传到 GitHub

通过 Remote-SSH，可以更方便地进行服务器代码开发、实验脚本管理和日志查看。
