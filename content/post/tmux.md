---
title: tmux 使用总结
date: 2019-04-17T18:24:21+08:00
description: ""
tags: ["tmux"]
categories: ["工具"]
---

tmux 是一个优秀的终端复用器类自由软件，功能类似 GNU Screen，但使用 BSD 许可>
发布。用户可以通过 tmux 在一个终端内管理多个分离的会话，窗口及面板，对于同时
使用多个命令行，或多个任务时非常方便。

<!--more-->

## 基础知识

Tmux 中的主要概念分为三个：

* Session - 默认开启Tmux的时候，就会自动新建一个会话，在这个会话中，也会给你开启一个默认的Window（也即窗口）。Tmux中可以拥有多个会话，多个会话之间可以来回无缝切换。
* Window - 一个 Session 中，可以开启多个 window。这些 window 同属一个 Session，并由其管理。
* Panel - Panel是比 Window 更小的界面元素。由 window 分割出来的单位就叫做 Panel。在同一个 window 中，通过控制光标在分割出的 Panel 中随意移动用来选定当前激活状态的 Panel。

{{% figure class="center" src="/img/tmux-arch.jpg" alt="tmux-arch" %}}

三者之间的关系是：Session -> Window -> Panel

### 安装

Mac OS 下通过 Homebrew 安装：
```sh
$ brew install tmux
```
通过以下命令查看是否安装正确以及版本号：
```sh
$ tmux -V
```
启动
```sh
$ tmux
```
退出（不推荐这种方式）
```sh
$ exit
```
### Session

**创建命名会话**
```sh
$ tmux new-session -s basic
# 简化命令
$ tmux new -s basic
```

**默认前缀快捷键**
`ctrl(^)+b`

**列出当前存在的会话**
```sh
$ tmux list-sessions
# 简化命令
$ tmux ls
```

**重新连接到已有会话**
```sh
# 如果只有一个会话
$ tmux attach
# 如果存在多个会话
$ tmux attach -t <session-name>
```

**杀死会话**
```sh
# 方式一
$ exit
# 方式二
$ tmux kill-session -t <session-name>
```

**分离会话**
```sh
PREFIX d
```

### Window

**创建并命名窗口**
```sh
# 创建窗口
prefix c
# 重命名窗口（前缀+逗号键）
prefix ,
```

**在窗口之间切换**
```sh
# 切换到下一个窗口（n=next）
prefix n
# 切换到上一个窗口（p=previous）
prefix p
# 切换到某个编号的窗口
prefix <num>
# 如果窗口数量超过 9 个，通过窗口名称查找（f=find）
prefix f
# 如果窗口数量超过 9 个，通过窗口列表查找（w=window）
prefix w
# 关闭一个窗口
exit 或者 prefix &
```

### Panel

**切割面板**
```sh
# 垂直切割
prefix %
# 水平切割
prefix "
# 面板间来回切换
prefix o
# 最大化或者恢复面板
prefix z
```

**面板布局**
```sh
# 切换面板布局
prefix <space>
```

默认布局有以下几种：

* even-horizontal 把所有面板均匀地水平排列，从左到右
* even-vertical 把所有均匀地垂直排列，从上到下
* main-horizontal 在顶部创建一个非常大的面板，其余面板变为小面板水平的放在底部
* main-vertical 在左侧创建一个非常大的面板，其余面板变为小面板水平的放在底部
* tiled 把所有面板大小均等地在屏幕上显示

**关闭面板**
```sh
# 大写的 X
exit 或者 prefix X
```

### 命令模式

**进入命令模式**
```sh
prefix :
```

**创建新窗口**
```sh
new-window -n console
```

**创建新窗口并执行命令**
```sh
new-window -n processes "top"`。
```
执行后，如果按下 q 键，程序会随窗口一起被关闭

### 快捷键表

| 命令 | 描述 |
| :-: | :-: |
| tmux new-session | 创建一个未命名的会话。可以简写为 tmux new 或者 tmux |
| tmux new -s development | 创建一个名为 development 的会话 |
| tmux new -s development -n editor | 创建一个名为 development 的会话，并将第一个窗口命名为 editor |
| tmux attach -t development | 连接到名为 development 的会话 |

| 快捷键 | 描述 |
| :-: | :-: |
| PREFIX d | 从会话中分离，让该会话在后台运行 |
| PREFIX : | 进入命令模式 |
| PREFIX c | 在当前会话中创建一个新窗口 |
| PREFIX 0...9 | 根据窗口编号选择窗口 |
| PREFIX w | 显示当前会话中所有窗口的可选列表 |
| PREFIX , | 显示一个提示符来重命名窗口 |
| PREFIX & | 杀死当前窗口，有确认提示 |
| PREFIX % | 将当前窗口垂直分割 |
| PREFIX " | 将当前窗口水平分割 |
| PREFIX o | 在以打开的面板之间循环移动当前焦点 |
| PREFIX q | 短暂地显示每个面板的编号 |
| PREFIX X | 关闭当前面板，有确认提示 |
| PREFIX SPACE | 循环使用 tmux 的默认面板布局 |

## 配置tmux

默认情况下，tmux 会在两个位置查找配置文件。首先查找 `/etc/tmux.conf` 作为系统配置，然后在当前用户的目录下查找（`~/.tmux.conf`）文件，后者优先级更高。

### 设置前缀键

```sh
set -g prefix C-a
unbind C-b
# 发送前缀键到其它程序
bind C-a send-prefix
```

### 修改默认延时

```sh
set -sg escape-time 1
```

### 设置窗口和面板索引

```sh
# 让窗口索引从 1 开始
set -g base-index 1
# 让面板索引从 1 开始
setw -g pane-base-index 1
```

### 创建重新加载配置的快捷键

设置快捷键为：`prefix r`
```sh
bind r source-file ~/.tmux.conf\; display "Reloaded!"
```
使用 display 能让 tmux 在状态栏输出一个消息。
在多个命令之间添加 \; 符号可以使一个键绑定执行多个命令。

然后通过 `prefix :` 进入命令模式，执行以下命令重新加载配置文件：
```sh
source-file ~/.tmux.conf
```

### 分割面板

```sh
# 水平分割
bind | split-window -h
# 垂直分割
bind - split-window -v
```

### 调整移动键和调整面板大小

```sh
# 调整移动键
bind h select-pane -L # 左
bind j select-pane -D # 下
bind k select-pane -U # 上
bind l select-pane -R # 右
# 选择窗口
bind -r C-h select-window -t :-
bind -r C-l select-window -t :+
# 调整面板大小
bind -r H resize-pane -L 5
bind -r J resize-pane -D 5
bind -r K resize-pane -U 5
bind -r L resize-pane -R 5
```

### 处理鼠标

```sh
set -g mode-mouse on
```

### 配置颜色

测试终端色彩模式

```sh
$ wget http://www.vim.org/scripts/download_script.php?src_id=4568 -O colortest
$ perl colortest -w
```

如果是 Linux 系统，需要在 `~/.bashrc` 中加入 `[ -z "$TMUX"] && export TERM=xterm-256color`。

```sh
set -g default-terminal "screen-256color"
```

**设置状态栏的颜色**
```sh
set -g status-fg white
set -g status-bg black
```

**设置窗口列表的颜色**
```sh
setw -g window-status-fg cyan
setw -g window-status-bg default
setw -g window-status-attr dim
```

**设置活动窗口的颜色**
```sh
setw -g window-status-current-fg white
setw -g window-status-current-bg red
setw -g window-status-current-attr bright
```

**设置命令行或消息的颜色**
```sh
set -g message-fg white
set -g message-bg black
set -g message-attr bright
```

### 开启活动通知

```sh
setw -g monitor-activity on
set -g visual-activity on
```

### 设置状态栏

```sh
# 设置状态栏左侧的内容和颜色
set -g status-left-length 40
set -g status-left "[fg=green]Session: #S #[fg=yellow]#I #[fg=cyan]#P"
set -g status-utf8 on

# 设置状态栏右侧的内容和颜色
# 15% | 28 Nov 18:15
set -g status-right "#(~/battery Discharging) | #[fg=cyan]%d %b %R"

# 每 60 秒更新一次状态栏
set -g status-interval 60
```

**状态栏变量**

| 变量 | 描述 |
| :-: | :-: |
| #H | 本地主机的主机名 |
| #h | 本地主机的主机名，没有 domain |
| #F | 当前窗口的标签 |
| #I | 当前窗口的索引 |
| #P | 当前面板的索引 |
| #S | 当前会话的名称 |
| #T | 当前窗口的标题 |
| #W | 当前窗口的名称 |
| ## | 一个 # 符号 |
| #(shell-command) | shell 命令的第一行输出 |
| #[attributes] | 要改变的颜色或属性 |

### 窗口列表居中显示

```sh
set -g status-justify centre
```

### 启用 vi 模式

```sh
setw -g mode-keys vi
```

### 配置表

| 命令 | 描述 |
| :-: | :-: |
| set -g prefix C-a | 设置前缀键为 Ctrl-a |
| set -sg escape-time n | 设置 tmux 等待前缀键和命令键之间的时间间隔（毫秒）|
| source-file [file] | 加载一个配置文件。重新加载当前配置文件或以后加入附加配置选项 |
| bind C-a send-prefix | 两次按下 PREFIX 键后向 tmux 发送组合键 |
| bind-key [key] [command] | 新建一个快捷键，执行指定的 command，可简写为 bind |
| bind-key -r [key] [command] | 新建一个可重复的快捷键，就是说只需要按下一次 PREFIX 键之后就可以重复地按下命令键 |
| unbind-key [key] | 移除一个定义的快捷键，将它绑定到其它命令 |
| display-message 或 display | 在状态消息里显示给定的文字 |
| set-option [flags] [option] [value] | 配置会话选项。使用 -g 可作为全局配置 |
| set-window-option [option] [value] | 配置窗口选项，比如活动通知、光标移动等 |
| set -a | 将值添加到当前选项而不是替换选项的值 |

## 脚本定制 tmux 环境

### 项目配置脚本

创建脚本并设置可执行
```sh
$ touch development
$ chmod +x development
```
脚本内容如下：
```sh
# 判断会话是否存在
tmux has-session -t development
if [ $? != 0 ]
then
    # 创建一个新的会话并分离
    tmux new-session -s development -n editor -d
    # 变更工作目录，需要事先创建好（C-m 表示回车）
    tmux send-keys -t development 'cd ~/prj/' C-m
    # 打开编辑器
    tmux send-keys -t development 'vim' C-m
    # 分割主编辑窗口
    tmux split-window -v -t development
    # 调整主编辑窗口布局
    tmux select-layout -t development main-horizontal
    # 在分割后的第二个面板切换路径
    tmux send-keys -t development:1.2 'cd ~/prj' C-m
    # 创建第二个窗口并切换路径
    tmux new-window -n console -t development
    tmux send-keys -t development:2 'cd ~/prj' C-m
    # 切回第一个窗口
    tmux select-window -t development:1
fi
tmux attach -t development
```
然后执行脚本即可。

### 使用 tmux 配置

创建一个名为 `app.conf` 的配置文件，内容如下：
```sh
source-file ~/.tmux.conf
# 创建一个新的会话并分离
new-session -s development -n editor -d
# 变更工作目录，需要事先创建好（C-m 表示回车）
send-keys -t development 'cd ~/prj/' C-m
# 打开编辑器
send-keys -t development 'vim' C-m
# 分割主编辑窗口
split-window -v -t development
# 调整主编辑窗口布局
select-layout -t development main-horizontal
# 在分割后的第二个面板切换路径
send-keys -t development:1.2 'cd ~/prj' C-m
# 创建第二个窗口并切换路径
new-window -n console -t development
send-keys -t development:2 'cd ~/prj' C-m
# 切回第一个窗口
select-window -t development:1
```

然后使用该配置文件，必须传入 `-f` 参数，并且需要使用 attach，因为 tmux 启动总是调用 new-session，如果不使用 attch 就会创建两个会话。
```sh
$ tmux -f app.conf attach
```

### 使用 tmuxinator 管理配置

前提：需要配置环境变量 $EDITOR。

首先创建一个新的 tmuxinator 项目，这里称为 development。
```sh
$ tmuxinator open development
```

然后修改配置文件内容：
```sh
name: development
root: ~/prj
windows:
  - editor:
      layout: main-horizontal
      panes:
        - vim
        - #empty, will just run plain bash
  - console: #empty
```

保存配置后，执行以下命令，tmuxinator 就会自动加载原始 `.tmux.conf` 配置文件并应用设置。
```sh
$ tmuxinator development
```

### 脚本化 tmux 命令

| 命令 | 描述 |
| :-: | :-: |
| tmux new-session -s development -n editor | 创建一个名为 development 的会话并将第一个窗口命名为 editor |
| tmux attach -t development | 连接到名为 development 的会话 |
| tmux send-keys -t development '[keys]' C-m | 向 development 会话的活动窗口或面板发送键盘命令，C-m 相当于按下回车键 |
| tmux send-keys -t development:1.0 '[keys]' C-m | 向 development 会话的第1个窗口和第1个面板发送键盘命令，C-m 相当于按下回车键 |
| tmux select-window -t development:1 | 选择 development 会话的第1个窗口为活动窗口 |
| tmux split-window -v -p 10 -t development | 在 development 会话里垂直分割当前窗口并将设置高度为总窗口大小的 10% |
| tmux select-layout -t development main-horizontal | 设置 development 的布局为 main-horizontal |
| tmux -f app.conf attach | 加载配置文件 app.conf，并连接到该会话 |
| tmuxinator open [name] | 在项目名为 name 的路径下使用默认编辑器打开配置文件，如果不存在就创建 |
| tmuxinator [name] | 对指定项目加载 tmux 会话 |
| tmuxinator list | 列出当前的项目 |
| tmuxinator copy [source] [destination] | 复制一个项目配置文件 |
| tmuxinator delete [name] | 删除指定项目 |
| tmuxinator implode | 删除当前所有的项目 |
| tmuxinator doctor | 使用 tmuxinator 和系统配置查找问题 |

## 文本和缓冲区

按下 `PREFIX [` 键会进入复制模式。如果要离开复制模式，按下回车键即可。

### 在缓冲区中快速移动

使用 `Ctrl b` 向上翻滚一屏或者使用 `Ctrl f` 向下翻滚一屏，使用 `g` 键跳转到缓冲区历史的最顶部，或者使用 `G` 键跳转到缓冲区历史的最底部。

### 复制和粘贴文本

按下 `PREFIX ]` 键会切换到粘贴模式。

**捕捉面板**

按下 `PREFIX :` 键进入命令模式然后输入：
```sh
capture-pane
```

**显示并保存缓存区**

在命令模式中输入 `show-buffer` 或者通过 `tmux show-buffer` 可以显示粘贴缓存区的内容。

通过 `save-buffer` 命令可以把缓存区保存到一个文件里。比如可以捕捉当前缓存区并保存到一个文本文件：
```sh
$ tmux capture-pane && tmux save-buffer buffer.txt
```
或者在命令模式中执行 `capture-pane; save-buffer buffer.txt` 命令。

通过 `list-buffers` 命令可以查看所有缓存区。然后输入 `choose-buffer` 来选择一个特定的缓存区将它放到焦点面板中。

### 按键表

**快捷键**

| 命令 | 描述 |
| :-: | :-: |
| PREFIX [ | 进入复制模式 |
| PREFIX ] | 粘贴当前缓存区的内容 |
| PREFIX = | 列出所有粘贴缓存区并粘贴选中的缓存内容 |

**复制模式**

| 命令 | 描述 |
| :-: | :-: |
| h,j,k,l | 移动光标，分别表示向左，向下，向上，向右 |
| w | 以一个单词为单位向前移动光标 |
| b | 以一个单词为单位向后移动光标 |
| f 后跟随任意字符 | 移动光标到指定字符的下一个匹配位置 |
| F 后跟随任意字符 | 移动光标到指定字符的前一个匹配位置 |
| CTRL b | 向上翻滚一个屏幕的位置 |
| CTRL f | 向下翻滚一个屏幕的位置 |
| g | 跳转到缓存区的顶部 |
| G | 跳转到缓存区的底部 |
| ? | 在缓存区内向后查找 |
| / | 在缓存区内向前查找 |

**命令模式**

| 命令 | 描述 |
| :-: | :-: |
| show-buffer | 显示当前缓存区的内容 |
| capture-pane | 捕捉指定面板的可视内容并复制到一个新的缓存区 |
| list-buffers | 列出所有的粘贴缓存区 |
| choose-buffer | 显示粘贴缓存区并粘贴选择的缓存区内的内容 |
| save-buffer [filename] | 保存缓存区的内容到指定文件 |

## 工作流

### 高效使用面板和窗口

**把面板变为窗口**

按下 `PREFIX !` 键就会根据当前面板创建一个新的窗口。

**把窗口变为面板**

假设现在有一个带有两个窗口的 tmux 会话：
```sh
$ tmux new-session -s panes -n first -d
$ tmux new-window -t panes -n second
$ tmux attach -t panes
```

然后进入命令模式，输入：
```sh
# 详细格式为 join-pane -t [session_name]:[window].[pane]
join-pane -s 1
```
这样会取出窗口 1 并添加到当前窗口，因为没有指定目标窗口。

### 在同一目录下打开新面板

> 根据个人喜好选择使用该功能。

通过使用 `send-keys` 命令来调用一个脚本把当前路径保存到环境变量中，然后这个脚本回调 `send-keys` 向拆分的窗口中发送命令，再把路径更改为环境变量中保存的那个路径。

首先在主目录下创建一个 `~/tmux-panes` 的新文件，写入：
```sh
TMUX_CURRENT_DIR=`pwd`
tmux split-window $1
tmux send-keys "cd $TMUX_CURRENT_DIR;clear"
```

然后编辑 `.tmux.conf` 来调用这个文件做分割。这里使用 `PREFIX v` 和 `PREFIX n` 键，防止覆盖已有的分割快捷键。
```sh
unbind v
unbind n
bind v send-keys "~/tmux-panes -h" C-m
bind n send-keys "~/tmux-panes -v" C-m
```

最后给 `tmux-panes` 脚本添加可执行权限：
```sh
$ chmod +x ~/tmux-panes
```

### 会话间移动

按下 `PREFIX (` 键会进入前一个会话，按下 `PREFIX )` 键会进入下一个会话，按下 `PREFIX s` 键会显示一个会话列表，这样可以快速跳转。

### 会话间移动窗口

默认快捷键为 `PREFIX .`，按下后，在显示的命令行输入会话名称，就可以将当前窗口移动到输入的会话中。也可以通过 shell 命令来完成这个功能。比如：
```sh
# 把 proesses 会话的第 1 个窗口移动到 editor 会话中
$ tmx move-window -s processes:1 -t editor
```

### 设置 shell

```sh
set -g default-command /bin/zsh
set -g default-shell /bin/zsh
```

### 将程序输出记录到日志

可以通过 `pipe-pane` 命令选择打开或者关闭这个功能。要激活该功能可以在命令模式下输入 `pipe-pane -o "mylog.txt"`，再次输入相同的命令可以将这个功能关闭。

为了便于使用，对该功能绑定一个快捷键：
```sh
bind P pipe-pane -o "cat >>~/prj/logs/#W.log" \; display "Toggled logging to ~/prj/logs/#W.log"
```

然后我们就可以通过 `PREFIX P` 命令来控制打开日志功能。

## 个人完整配置参考

```sh
# -----------------
# Tmux2.8 自定义配置
# -----------------

# 备注
# 移除前缀绑定键
# unbind C-b

# 修改 Prefix 组合键为 ⌃-a
set -g prefix C-a
unbind C-b
bind C-a send-prefix

# 设置延时
set  -sg escape-time     1 # 修改默认延时，设置为1ms，增加响应

# 设置窗口面板和索引
set  -g  base-index      1 # 让窗口索引从 1 开始
setw -g  pane-base-index 1 # 让面板索引从 1 开始

# 选择面板快捷键
bind h select-pane -L # 左
bind j select-pane -D # 下
bind k select-pane -U # 上
bind l select-pane -R # 右

# 选择窗口快捷键
bind -r C-h select-window -t :-
bind -r C-l select-window -t :+

# 调节面板大小快捷键
bind -r H resize-pane -L 5
bind -r J resize-pane -D 5
bind -r K resize-pane -U 5
bind -r L resize-pane -R 5

# 开启鼠标模式
set -g mouse on

# 修改默认的分割panel快捷键
bind | split-window -h
bind - split-window -v

# 加载tmux配置文件的快捷键: prefix r
bind r source-file ~/.tmux.conf\; display "Reloaded!"

# --------
# 颜色配置
# --------

# 以256色显示内容
set -g default-terminal "screen-256color"

# 设置状态栏的颜色
set -g status-fg white
set -g status-bg black

# 设置窗口列表颜色
setw -g window-status-fg cyan
setw -g window-status-bg default
setw -g window-status-attr dim

# 设置活动窗口的颜色
setw -g window-status-current-fg white
setw -g window-status-current-bg red
setw -g window-status-current-attr bright

# 设置命令行或消息的颜色
set -g message-fg white
set -g message-bg black
set -g message-attr bright

# 开启活动通知
setw -g monitor-activity on
set -g visual-activity on

# 设置状态栏左侧的内容和颜色
set -g status-left-length 40
set -g status-left "#[fg=green]Session: #S #[fg=yellow]#I #[fg=cyan]#P"

# 设置状态栏右侧的内容和颜色
# 28 Nov 18:15
set -g status-right "#[fg=cyan]%d %b %R"

# 每 60 秒更新一次状态栏
set -g status-interval 60

# 开启 vi 按键
setw -g mode-keys vi
```

更加便捷的方法可以直接使用别人配置好的，这里推荐使用 [.tmux](https://github.com/gpakosz/.tmux/blob/master/.tmux.conf)。

## 参考文章

* [tmux-Productive-Mouse-Free-Development_zh](https://legacy.gitbook.com/book/aquaregia/tmux-productive-mouse-free-development_zh/details)
