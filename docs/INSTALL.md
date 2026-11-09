# 安装、升级与卸载

## 1. 确认已有 Agent 启动方式

面板会复用既有 Agent 连接启动器。当前实现调用方式为：

```text
Python3 launch.py <Agent安装目录> serve --pure --hostname 127.0.0.1 --port 0
```

因此 Agent 安装目录必须包含 `launch.py`、可执行的 `opencode`，且配套启动器支持上述服务启动方式。开发环境使用 `xunchi-agent-0.1.2-chatgpt.1`；单独一个旧版 TUI 命令或公开仓库中的历史 r1 运行包不等同于这个配套安装。

接收者先确认自己的 TUI 已能正常连接模型，然后确定它的实际安装目录。模型连接使用接收者已有配置。

## 2. 检查 Python 与桌面

```bash
python3 --version
echo "$DISPLAY"
/usr/bin/python -c 'import gi; gi.require_version("Gtk","3.0"); gi.require_version("GdkX11","3.0"); from gi.repository import Gtk,GdkX11; print("GTK OK")'
```

- Python3 至少为 3.6。CentOS 7 可能没有名为 `python3` 的命令，可使用已有 Python3 的绝对路径执行安装器，并将同一路径赋给 `XUNCHI_VIRTUOSO_PYTHON3`。
- `DISPLAY` 应指向当前可工作的 X11 桌面；使用平时能正常启动 Virtuoso 的图形会话。
- GUI 使用的是 `/usr/bin/python` 的 Gtk 绑定，不是 Python3 的 Gtk。缺少依赖时由本机环境管理员按发行版软件源补齐。
- 现有验证环境为 IC618。其他 Cadence 版本、窗口管理器和远程桌面环境需要现场核对。

## 3. 下载和校验

从本私有仓库的 `packages/` 或 Releases 下载这两个文件：

```text
xunchi-virtuoso-0.5.2.tar.gz
SHA256SUMS
```

在文件所在目录执行：

```bash
sha256sum -c SHA256SUMS
tar -xzf xunchi-virtuoso-0.5.2.tar.gz
```

预期校验结果为 `xunchi-virtuoso-0.5.2.tar.gz: OK`。已有同名解压目录时先选择空目录，避免混用文件。

## 4. 首次安装

```bash
python3 xunchi-virtuoso-0.5.2/install.py install
```

默认安装目录：

```text
~/.local/share/xunchi-virtuoso/0.5.2/
```

安装器会备份已有用户 `.cdsinit`，增加受管加载块。重复安装会核对安装内容并返回 `already_installed`；检测到已安装文件被修改时会保留现场并停止。

### 升级已有面板

从0.5.1升级示例：

```bash
python3 xunchi-virtuoso-0.5.2/install.py install \
  --upgrade-from "$HOME/.local/share/xunchi-virtuoso/0.5.1"
```

其他旧版本将路径改为实际旧安装目录。升级器校验旧清单，替换受管加载块，保留旧版文件；若有用户修改，先按报错审阅实际差异。

### 使用项目自己的初始化文件

如果项目或站点 `.cdsinit` 优先加载，且没有读取用户 `~/.cdsinit`，可指定自己维护且确实会被 Virtuoso 加载的文件：

```bash
python3 xunchi-virtuoso-0.5.2/install.py install \
  --cdsinit /absolute/path/to/user-maintained/.cdsinit
```

按实际情况选择首次安装、升级或指定初始化文件命令，不要连续重复安装到不同启动文件。安装路径要求 ASCII；初始化文件若是符号链接，指定其真实目标。

## 5. 设置环境并正常启动

在 Bash 中设置，路径按接收者电脑修改：

```bash
export XUNCHI_AGENT_HOME="$HOME/.local/opt/xunchi-agent-0.1.2-chatgpt.1"
export XUNCHI_VIRTUOSO_PYTHON3="$(command -v python3)"

ls -l "$XUNCHI_AGENT_HOME/launch.py" "$XUNCHI_AGENT_HOME/opencode"
"$XUNCHI_VIRTUOSO_PYTHON3" --version

virtuoso &
```

这两项变量必须在启动 Virtuoso 前设置，或放入其 Bash 启动脚本；从桌面图标、另一终端或其他 Shell 启动时，要确认继承了相同配置。V0.5.2 未设置变量时的 Python3 默认路径属于开发机，换电脑时务必显式指定。

正常重启 Virtuoso 后新插件才会加载。`-nocdsinit` 会跳过初始化配置，不能使用该选项启动日常面板会话。

打开设计，再选择 **XunChi Agent → Open Chat Panel**。首次打开会启动专属本机 Agent 服务；其他已运行的终端 TUI 不需要退出。

## 6. 首次验收

依次确认：

1. 设计窗口出现 XunChi Agent 菜单，面板可打开、隐藏和重开。
2. 面板连接就绪，输入中文后能得到真实模型回复。
3. 发送“请查询当前 library、cell、view、访问模式及选中对象，不要修改设计”，返回与实际窗口一致。
4. 如使用 Layout XL，切换 L→XL 后核对菜单和面板是否仍可使用。

连接失败时使用面板“重连”，并参考[故障排查](TROUBLESHOOTING.md)。

## 7. 卸载

```bash
python3 "$HOME/.local/share/xunchi-virtuoso/0.5.2/install.py" uninstall
```

自定义安装路径时使用对应目录内的安装器。卸载会核对文件、移除未修改的插件及受管加载块，保留其他配置和会话记录。正在运行的 Virtuoso 需要正常重启，才能卸载已载入的会话内菜单。
