# XunChi Agent · Virtuoso 面板

在 Cadence Virtuoso 内使用中文聊天面板，查询当前设计，并通过原生 SKILL 接口执行设计操作。

XunChi Agent 由苏州迅驰信息科技有限公司定制，基于 OpenCode。本仓库交付 Virtuoso 面板插件及安装说明，配合接收者已有的 XunChi Agent 使用。

**当前版本：V0.5.2（2026-11-09）。**

## 下载

- [最新版安装包](packages/xunchi-virtuoso-0.5.2.tar.gz)
- [SHA256 校验文件](packages/SHA256SUMS)
- [版本记录](CHANGELOG.md)
- [GitHub Releases](https://github.com/Look001122/xunchi-agent-virtuoso-panel/releases)

这是私有仓库。接收者需要先接受仓库邀请，并登录被邀请的 GitHub 账号，才能查看和下载。下载面板安装包；GitHub 自动生成的 **Source code.zip / tar.gz** 是整个仓库快照，不是同一个插件安装包。

## 适用环境

| 项目 | 要求 |
| --- | --- |
| 系统 | Linux x86-64、X11 桌面；已验证开发环境为 CentOS 7 |
| Virtuoso | 已有可正常工作的 Cadence Virtuoso；已验证版本 IC618 |
| Agent | 已有可用的 XunChi Agent 及模型连接；当前启动方式要求安装目录含 `launch.py` 和可执行的 `opencode` |
| Python3 | Python 3.6+，用于安装器及既有 Agent 连接启动器 |
| GUI Python | `/usr/bin/python` 能导入 GI、Gtk3、GdkX11；CentOS 7 开发环境使用系统 Python2.7 |
| 其他 | Bash、curl；jq 已随插件提供 |

当前默认 Agent 目录为 `~/.local/opt/xunchi-agent-0.1.2-chatgpt.1/`。如果只装过旧版 TUI 或使用其他打包形式，请先核对[完整安装说明](docs/INSTALL.md)中的启动器要求。

## 快速安装

以下命令在接收者自己的 **Bash 终端**中执行。先下载安装包和 `SHA256SUMS`，放在同一个目录。

### 1. 检查依赖

```bash
python3 --version

/usr/bin/python -c 'import gi; gi.require_version("Gtk","3.0"); gi.require_version("GdkX11","3.0"); from gi.repository import Gtk,GdkX11; print("GTK OK")'
```

### 2. 校验并安装

在下载文件所在目录执行。解压目录已存在时先选择新的空目录。

```bash
sha256sum -c SHA256SUMS
tar -xzf xunchi-virtuoso-0.5.2.tar.gz
python3 xunchi-virtuoso-0.5.2/install.py install
```

安装器默认写入 `~/.local/share/xunchi-virtuoso/0.5.2/`，并备份、更新用户 `.cdsinit` 中的插件加载块。已有旧版插件时使用[升级命令](docs/INSTALL.md#升级已有面板)。

### 3. 设置本机路径并启动

```bash
# 改成接收者自己的 Agent 安装目录。
export XUNCHI_AGENT_HOME="$HOME/.local/opt/xunchi-agent-0.1.2-chatgpt.1"

# 必须指向接收者电脑上可用的 Python3.6+。
export XUNCHI_VIRTUOSO_PYTHON3="$(command -v python3)"

ls -l "$XUNCHI_AGENT_HOME/launch.py" "$XUNCHI_AGENT_HOME/opencode"
"$XUNCHI_VIRTUOSO_PYTHON3" --version

# 按平时方式准备好 Cadence 环境后，在同一终端启动。
virtuoso &
```

路径检查应能找到两个文件，`opencode` 应具有执行权限，Python3 应能输出版本。任一报错时先修正路径，再启动 Virtuoso，详见[故障排查](docs/TROUBLESHOOTING.md)。

**两项环境变量必须由 Virtuoso 进程继承。** V0.5.2 的 Python3 默认值指向开发机的 Calibre 路径，接收者应显式设置自己的 Python3 路径。验证成功后，将这两项配置加入自己常用的 Virtuoso 启动脚本。

### 4. 打开并验证

正常启动 Virtuoso，打开 schematic 或 layout，点击：

**XunChi Agent → Open Chat Panel**

第一次发送：

> 请查询实际设计，告诉我当前关联的 library、cell、view、访问模式及选中对象数量，不要修改设计。

返回结果应与实际窗口一致。面板显示成功、模型连接成功、实时设计查询成功是三项分别需要确认的结果。

## 使用与注意事项

- **Enter 发送，Shift+Enter 换行**；拼音预编辑时 Enter 优先确认输入法。
- 顶部下拉框选择关联设计，点击“关联当前”可重新关联；切换焦点不会自动改绑。
- “Hide Panel”隐藏后可从菜单重开，同一会话的历史保持；“清空显示”保留 Agent 会话。
- 当前面板具备 SKILL 和 Shell 设计操作能力。执行修改前核对关联设计及实际读写模式；文件锁、Linux 权限和许可证仍由现有环境决定。
- “停止”会取消 Agent 等待，已进入 Virtuoso 的同步 SKILL 可能继续运行；操作报错可能已经修改部分对象，先核对结果再重试。
- DRC、LVS 和仿真使用接收者自己的工具、PDK、规则及工程配置。结论以匹配输入版本的真实报告为准。
- 同一 Virtuoso 进程保留一个面板，可显式关联同进程的其他设计窗口；整台 Virtuoso 退出后的自动聊天恢复尚未实现。

完整步骤、升级及卸载见 [INSTALL.md](docs/INSTALL.md)；常见问题见 [TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)。

## 验证范围

前版已在 CentOS 7 / IC618 上完成面板嵌入、中文交互、原生设计查询及 Layout L→XL 菜单恢复验证。V0.5.2 新增的执行状态修复通过8项轮询逻辑回归、9项安装器测试；本版本尚未进行真实 Virtuoso 界面验收，也未在接收者电脑上验证。

## 第三方组件

插件随包提供 jq 1.8.1 Linux x86-64 运行时，相关说明与许可证见安装包内 `THIRD_PARTY.md`、`JQ_COPYING`、`ONIGURUMA_COPYING`。Cadence、PDK、Agent 引擎和模型连接由接收者已有环境提供。
