# Tips All You Need to Work Decently

面向日常开发与科研计算的实用指南：SSH 远程连接、tmux 会话管理、CPU / GPU 资源检查、Git 协作与 Slurm 作业管理。

## 目录

- [使用前须知](#使用前须知)
- [Remote Server 使用指南](#remote-server-使用指南)
- [CPU Machine 使用指南](#cpu-machine-使用指南)
- [GPU Machine 使用指南](#gpu-machine-使用指南)
- [VSCode Git 使用指南](#vscode-git-使用指南)
- [Slurm 使用指南](#slurm-使用指南)
- [参考资料与贡献](#参考资料与贡献)

## 使用前须知

- 命令主要面向 Linux / macOS 终端；Windows 可使用 OpenSSH、Git 和 WSL，具体工具需单独安装。
- 示例中的 `username`、`server.example.com`、`feature-name` 等均需替换为实际值；尖括号占位符不能原样粘贴到终端。
- 遇到问题可借助 AI 排查，但应核对官方文档。删除文件、终止任务、重置提交前，先确认影响范围并做好备份。
- 不要提交密码、访问令牌、私钥或包含敏感信息的截图。共享服务器应遵守管理员的资源使用规定。

## Remote Server 使用指南

### SSH

#### 登录

```bash
ssh username@server.example.com
# 非默认端口（默认端口为 22）
ssh -p 2222 username@server.example.com
```

首次连接时，应向管理员核对主机密钥指纹，再接受连接。遇到主机密钥变更警告时，不要直接删除记录或禁用检查。

#### 密钥对管理

在本机生成密钥，并将公钥安装到远程服务器：

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
ssh-copy-id -i ~/.ssh/id_ed25519.pub -p 2222 username@server.example.com
```

- 生成时建议设置密钥口令；若目标文件已存在，不要覆盖仍在使用的密钥。
- `ssh-keygen` 不需要服务器端口参数；它的 `-p` 表示修改现有私钥的口令。
- `ssh-copy-id` 需在本机安装，且通常需要已有的远程登录方式。Windows 用户可在 WSL 中使用，或请管理员安装公钥。
- Windows 与 WSL 的主目录和密钥默认不共享；应确认 VS Code 实际使用的 SSH 客户端及密钥路径。

> **安全提醒：** `.pub` 文件是公钥，可交给管理员；没有 `.pub` 后缀的对应文件是私钥，绝不能分享或上传到仓库。

#### 结合 VS Code 使用

1. 安装 Microsoft 的 **Remote - SSH** 扩展。
2. 打开 Remote Explorer，点击 New Remote，输入 SSH 登录命令。

![在 Remote Explorer 中添加 SSH 服务器](figs/SSH_add_remote_server.png)

3. 点击 Open SSH Config File，选择当前客户端使用的配置文件。

![打开 SSH 配置文件](figs/SSH_config.png)

例如，在 `~/.ssh/config` 中添加：

```sshconfig
Host work-server
    HostName server.example.com
    Port 2222
    User username
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
```

`Host` 是自定义别名，此后可运行 `ssh work-server`，也可在 VS Code 中选择该别名连接。Windows OpenSSH 通常使用 `%USERPROFILE%\.ssh\config`。

### tmux

tmux 可以管理多个会话、窗口和面板。建议在**远程服务器上**启动 tmux 后运行长任务，以便 SSH 断开后重新连接。

> tmux 不防止服务器重启、进程崩溃或管理员清理任务，也不替代 Slurm 等作业调度器。

#### 常用命令

| 命令 | 用途 |
| --- | --- |
| `tmux ls` | 查看会话列表；没有会话时可能提示没有运行中的服务器 |
| `tmux new -s work` | 创建名为 work 的会话 |
| `tmux attach -t work` | 重新连接会话 |
| `tmux kill-session -t work` | 销毁会话，可能导致其中运行的任务终止 |

#### 常用快捷键

先按下 `Ctrl+b`，松开两个键，再按表中的后续键。

| 后续键 | 用途 |
| --- | --- |
| `d` | 分离会话，任务继续运行（不是暂停任务） |
| `c` | 新建窗口 |
| `,` | 重命名窗口 |
| `%` | 左右分屏 |
| `"` | 上下分屏 |
| 方向键 | 切换面板 |
| `?` | 查看快捷键帮助 |

离开时优先使用 `Ctrl+b` → `d`；仅在确认任务结束或不再需要时销毁会话。

## CPU Machine 使用指南

以下命令主要适用于 Linux；`htop` 可能需要额外安装。

| 命令 | 用途 |
| --- | --- |
| `top` / `htop` | 查看 CPU、内存和进程状态 |
| `free -h` | 查看内存和交换空间 |
| `df -h` | 查看文件系统剩余空间 |
| `du -sh path/to/directory` | 查看指定目录占用空间 |
| `ps -u "$USER" -f` | 查看当前用户的进程 |

共享服务器上应限制并行线程数，避免占满 CPU 或内存。终止进程前核对用户、PID 和命令；不要结束他人的任务。

## GPU Machine 使用指南

以下命令面向 NVIDIA GPU，需要相应驱动；`nvcc` 还需要安装 CUDA Toolkit。

| 命令 | 用途 |
| --- | --- |
| `nvidia-smi` | 查看 GPU、显存、驱动和进程信息 |
| `watch -n 1 nvidia-smi` | 每秒刷新一次；按 `Ctrl+c` 停止监控 |
| `nvcc --version` | 查看当前 PATH 中 CUDA 编译器的版本 |

> `nvidia-smi` 显示的 CUDA Version 表示驱动支持的 CUDA 版本上限，不等于已安装的 CUDA Toolkit 或深度学习框架使用的运行时版本。

- 启动任务前检查显存与资源分配，降低 OOM（out of memory）风险；空闲显存不等于获得使用权限。
- Slurm 集群上应通过调度器申请 GPU，不要仅凭 `nvidia-smi` 输出直接占用设备。
- 任务结束后检查资源是否释放。SSH 断开不代表后台任务结束；计划继续运行的任务无需因下线而停止。

## VSCode Git 使用指南

### 基本概念与命令

Git 的基本流程是：工作区修改 → 暂存（`add`）→ 本地提交（`commit`）→ 推送（`push`）。**本地提交不会自动上传到远程仓库。**

| 命令 | 用途 |
| --- | --- |
| `git status` | 查看当前分支、暂存和未暂存改动 |
| `git diff` | 查看尚未暂存的改动 |
| `git diff --staged` | 查看即将提交的改动 |
| `git add README.md` | 暂存指定文件 |
| `git commit -m "docs: update guide"` | 提交暂存区内容 |
| `git remote -v` | 查看远程仓库地址 |
| `git fetch origin` | 更新远程跟踪信息，不合并到当前分支 |
| `git pull --ff-only` | 拉取并仅允许快进更新；分支分叉时停止 |
| `git push` | 将本地提交推送到已配置的上游分支 |
| `git log --oneline -n 10` | 查看最近十条提交 |

首次提交前，可按需为当前仓库设置身份（添加 `--global` 则影响所有仓库的默认设置）：

```bash
git config user.name "Your Name"
git config user.email "your_email@example.com"
```

#### 获取或创建仓库

已有远程仓库时，优先克隆：

```bash
git clone https://github.com/USERNAME/REPOSITORY.git
cd REPOSITORY
```

从本地新建并连接**空的远程仓库**：

```bash
git init -b main
git add README.md
git commit -m "docs: add README"
git remote add origin https://github.com/USERNAME/REPOSITORY.git
git push -u origin main
```

上述示例假设本地已有 `README.md`，远程仓库已创建且没有提交。若已有 `origin`，先查看地址，不要重复添加。HTTPS 推送需配置凭据管理器或令牌；不要把令牌写入命令、URL 或文档。

#### 分支协作

```bash
git switch -c feature-name
# 编辑文件，再检查、暂存并提交
git add README.md
git commit -m "docs: improve guide"
git push -u origin feature-name
```

可通过 Pull Request 合入主分支；本地合并的基本步骤为：

```bash
git switch main
git pull --ff-only
git merge feature-name
git push
```

切换、拉取或合并前，先检查工作区并妥善保存改动。`--ff-only` 失败说明历史可能已分叉，应查看日志后决定合并或变基，不要直接强制推送。

#### 撤销与恢复

| 命令 | 效果与注意事项 |
| --- | --- |
| `git restore --staged README.md` | 取消暂存，保留工作区修改（仓库需已有提交） |
| `git revert <commit-id>` | 创建新提交撤销指定提交，适合已共享的普通提交；可能产生冲突 |
| `git reset --soft <commit-id>` | 移动当前分支指针，保留索引和工作区；会改写本地历史 |
| `git reset --hard <commit-id>` | 重置分支、索引和工作区，丢弃已跟踪文件的未提交修改；还可能删除阻碍检出的未跟踪文件 |

> `reset --hard` 具有破坏性，不作为日常撤销的默认方式。已推送的提交优先使用 `revert`；撤销合并提交需要额外选择主线，不能盲目套用普通提交的操作。

### VS Code Git 可视化操作

界面名称可能随版本或语言变化；以下保留原有截图作为参考。

#### 初始化仓库

打开项目文件夹，在 Source Control（源代码管理）中点击 **Initialize Repository**，相当于 `git init`。

![VS Code 初始化 Git 仓库](figs/初始化界面.png)

#### 暂存

点击文件旁的加号暂存指定文件，并检查差异。

![点击加号暂存文件](figs/暂存.png)

![文件进入暂存区后的状态](figs/暂存效果.png)

通常提交只包含暂存区内容。暂存区为空时，VS Code 可能依据 Smart Commit 设置提示暂存所有改动；建议显式选择文件，避免误提交。

#### 提交（本地）

输入提交信息后点击 **Commit**。也可在弹出的提交信息编辑器中填写内容并确认。

![填写信息并提交暂存内容](figs/提交（本地）方法1.png)

![在编辑器中填写提交信息](figs/提交（本地）方法2.png)

#### 推送与同步

只上传本地提交时，使用菜单中的 **Push**。**Sync Changes** 通常先拉取再推送，并非单纯的 Push；远程有新提交时，可能需要处理合并或冲突。

![VS Code 的 Sync Changes 操作入口](figs/向远程仓库提交.png)

![状态栏中的同步按钮](figs/同步远程仓库信息.png)

仅更新远程跟踪信息时使用 **Fetch**；需要将远程更新整合到当前分支时使用 **Pull**。

![通过菜单拉取远程更新](figs/拉取远程仓库更新.png)

#### 分支管理与冲突处理

通过状态栏分支名称或命令面板创建、切换分支。

![分支管理入口](figs/分支管理1.png)

![选择需要切换的分支](figs/分支管理2.png)

合并前确认当前分支是接收修改的目标分支，再选择待合并分支。

![选择要合并的分支](figs/分支合并.png)

发生冲突时，点击 **Resolve in Merge Editor**。

![打开合并冲突编辑器](figs/分支合并冲突处理1.png)

逐处比较当前版本和传入版本，可以保留任一侧、组合两侧或手动编辑；不要不加检查地全部接受某一侧。

![比较冲突两侧并编辑合并结果](figs/分支合并冲突处理2.png)

点击 **Complete Merge** 后检查文件、运行必要测试，并确认暂存与合并提交是否完成；该按钮不代表修改已推送。

![确认合并结果](figs/分支合并冲突处理3.png)

命令行合并期间如需放弃，可使用 `git merge --abort`。它只适用于正在进行的合并；合并前应先保存已有改动。

### 忽略文件与大文件

`.gitignore` 中填写的是**需要忽略的路径或匹配规则**，不是文件内容。规则仅作用于未跟踪文件，不会自动停止跟踪已提交的文件。

```gitignore
# 本地配置与临时文件
.env
.env.*
!.env.example
*.swp
.DS_Store
```

若要停止跟踪已提交文件，应先确认影响，再运行 `git rm --cached path/to/file` 并提交删除记录；本地文件会保留。该操作不会清除历史中的敏感信息：若泄露凭据，应立即撤销或轮换凭据。

大型数据集、模型权重和生成产物不宜直接放入普通 Git 历史；可按需要使用 Git LFS 或外部存储。

## Slurm 使用指南

Slurm 用于集群资源分配和任务排队。以下仅为通用入门示例，分区、账户、QoS、GPU 申请方式和环境初始化均以本集群文档为准；不要在登录节点直接运行重型任务。

### 常用命令

| 命令 | 用途 |
| --- | --- |
| `sinfo` | 查看分区和节点状态 |
| `squeue -u "$USER"` | 查看自己的排队和运行作业 |
| `sbatch job.sh` | 提交批处理作业，返回作业 ID |
| `scontrol show job <job-id>` | 查看作业详情和等待原因 |
| `sacct -j <job-id> --format=JobID,State,Elapsed,ExitCode` | 查看作业记录；需要集群启用记账功能 |
| `scancel <job-id>` | 取消指定作业；先核对 ID |

### 最小批处理示例

将以下内容保存为 `job.sh`。该示例只运行 `hostname`，可在验证后替换为实际任务：

```bash
#!/usr/bin/env bash
#SBATCH --job-name=hello
#SBATCH --output=slurm-%j.out
#SBATCH --time=00:05:00
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem=1G

set -euo pipefail

# 按集群要求在此加载软件环境
srun hostname
```

在脚本所在目录提交并检查：

```bash
sbatch job.sh
squeue -u "$USER"
```

- `#SBATCH` 指令需放在第一条实际执行命令之前；其中的 shell 变量不会自动展开。
- 示例使用集群默认分区与账户；无默认配置时需按管理员要求添加对应指令。
- `%j` 会替换为作业 ID。作业从队列消失不一定代表成功，还需检查日志、状态和退出码。
- GPU 作业需额外申请 GPU；常见写法包括 `--gres=gpu:1` 或 `--gpus=1`，支持情况由集群配置决定，不应盲目同时添加。
- `sbatch` 提交成功后通常无需保持 SSH 连接；不要通过 tmux 绕过资源调度。

## 参考资料与贡献

- [OpenSSH 手册](https://www.openssh.com/manual.html)
- [VS Code Remote - SSH](https://code.visualstudio.com/docs/remote/ssh)
- [tmux 使用入门](https://github.com/tmux/tmux/wiki/Getting-Started)
- [Git 命令参考](https://git-scm.com/docs)
- [VS Code 版本控制](https://code.visualstudio.com/docs/sourcecontrol/overview)
- [NVIDIA System Management Interface](https://docs.nvidia.com/deploy/nvidia-smi/index.html)
- [Slurm 快速入门](https://slurm.schedmd.com/quickstart.html)

文档集中在 `README.md`，操作截图位于 `figs/`。欢迎通过 Issue 或 Pull Request 提交更正；新增示例请注明适用环境，并检查图片路径与命令的安全性。
