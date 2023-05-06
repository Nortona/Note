### 3.1 新建会话

第一个启动的 Tmux 窗口，编号是`0`，第二个窗口的编号是`1`，以此类推。这些窗口对应的会话，就是 0 号会话、1 号会话。

使用编号区分会话，不太直观，更好的方法是为会话起名。
 
 ```bash
 $ tmux new -s <session-name>
```

上面命令新建一个指定名称的会话。

### 3.2 分离会话

在 Tmux 窗口中，按下`Ctrl+b d`或者输入`tmux detach`命令，就会将当前会话与窗口分离。

```bash
$ tmux detach
```

上面命令执行后，就会退出当前 Tmux 窗口，但是会话和里面的进程仍然在后台运行。

`tmux ls`命令可以查看当前所有的 Tmux 会话。

```bash
$ tmux ls
# or
$ tmux list-session
```

### 3.3 接入会话

`tmux attach`命令用于重新接入某个已存在的会话。

```bash
# 使用会话编号
$ tmux attach -t 0

# 使用会话名称
$ tmux attach -t <session-name>
```

### 3.4 杀死会话

`tmux kill-session`命令用于杀死某个会话。

```bash
# 使用会话编号
$ tmux kill-session -t 0
# 使用会话名称
$ tmux kill-session -t <session-name>
```

### 3.5 切换会话

`tmux switch`命令用于切换会话。

```bash
# 使用会话编号
$ tmux switch -t 0

# 使用会话名称
$ tmux switch -t <session-name>
```

### 3.6 重命名会话

`tmux rename-session`命令用于重命名会话。

```bash
$ tmux rename-session -t 0 <new-name>
```

上面命令将0号会话重命名。

### 3.7 会话快捷键

下面是一些会话相关的快捷键。

> -   `Ctrl+b d`：分离当前会话。
> -   `Ctrl+b s`：列出所有会话。
> -   `Ctrl+b $`：重命名当前会话。
> -   `Ctrl+b [`:   进入翻页模式。