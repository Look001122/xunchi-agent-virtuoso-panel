# 常见问题

## 私有仓库打不开或下载显示404

先由仓库所有者在 **Settings → Collaborators** 邀请接收者。接收者接受邀请，并登录对应 GitHub 账号后再下载。公开分享网址不会自动授予私有仓库访问权。

## 安装成功，但没有 XunChi Agent 菜单

检查：

1. 安装后是否正常重启 Virtuoso。
2. 启动参数是否带了 `-nocdsinit`。
3. 是否已经打开一个 schematic 或 layout 窗口。
4. 项目/站点初始化文件是否绕过了被安装器修改的用户 `.cdsinit`。
5. CIW 是否报告插件加载错误；结合实际加载顺序选择 `--cdsinit`。

## 面板打开，但提示依赖缺失

最常见的是以下路径不正确：

```bash
export XUNCHI_AGENT_HOME="/absolute/path/to/your-agent"
export XUNCHI_VIRTUOSO_PYTHON3="/absolute/path/to/python3"
```

检查 Agent 目录是否有 `launch.py` 和可执行的 `opencode`，Python3 是否至少为3.6，且配套启动器支持服务模式。仅有旧版 TUI 可执行文件时，需要先补齐兼容的 Agent 安装。

环境变量要在启动 Virtuoso 之前设置。已启动的进程不会继承之后才在终端中设置的变量。

## Gtk、GI 导入失败或面板无法显示

使用[安装说明](INSTALL.md#2-检查-python-与桌面)中的 `/usr/bin/python` 检查命令。GUI 需要系统解释器的 GI/Gtk3/GdkX11；另装 Python3 的 Gtk 绑定不能直接满足此要求。确认当前 X11 图形会话能正常显示 Virtuoso。

## TUI 能回答，面板连接失败

面板启动的是独立 Agent 服务。确认面板继承了正确的 Agent 目录、Python3 路径和既有连接环境，再查看实际日志。UI显示出来并不代表 Agent 服务已就绪。

会话日志位于：

```text
~/.local/state/xunchi-virtuoso/session-*/
```

常用文件：

| 文件 | 用途 |
| --- | --- |
| `agent_state.json` | Agent 连接状态和错误码 |
| `agent_server.log` | Agent 服务启动输出 |
| `agent_transport.log` | 请求与通信错误 |
| `chat.log` | 面板运行错误 |
| `tool_calls.jsonl` | 实际设计工具调用记录 |
| `skill-output-*.log` | SKILL 执行输出 |

确认对应的会话目录后查看本次时间附近的日志，再使用“重连”。提交问题时提供版本、操作步骤和相关错误片段即可。

## 返回的不是我正在看的设计

面板保持显式关联，不会随焦点自动切换。先检查顶部下拉框，使用“关联当前”重新关联，再发送实时查询。不同 Virtuoso 进程有各自的面板和会话。

## 修改报错、只改了一部分或停止后仍在执行

先核对实际数据库和工具日志。同步 SKILL 进入 Virtuoso 后可能继续运行；异常不会自动回滚此前表达式。确认原生操作已结束及设计状态后再重试，避免重复执行写操作。

## 中文输入或滚动异常

先在当前桌面的普通文本编辑器确认中文输入法可用，再在面板验证。向上滚动或选择历史文字会暂停自动跟随，点击“最新”恢复。当前嵌入实现已在开发机的 CentOS 7/X11 环境验证，其他窗口管理器和远程桌面需现场核对。

## 升级提示文件被修改

升级器会保护已修改的安装文件及启动配置。保留报错现场，检查对应文件的实际差异，确认修改来源后再决定升级方式。使用安装器生成的备份追溯初始化配置。
