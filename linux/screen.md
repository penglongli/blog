## Linux 工具之 screen 的使用

screen 工具主要为了在后台运行命令，概念之类的不介绍了，仅介绍常用的。

## screen --help
```
$ screen --help
Use: screen [-opts] [cmd [args]]
 or: screen -r [host.tty]

Options:
-4            Resolve hostnames only to IPv4 addresses.
-6            Resolve hostnames only to IPv6 addresses.
-a            Force all capabilities into each window's termcap.
-A -[r|R]     Adapt all windows to the new display width & height.
-c file       Read configuration file instead of '.screenrc'.
-d (-r)       Detach the elsewhere running screen (and reattach here).
-dmS name     Start as daemon: Screen session in detached mode.
-D (-r)       Detach and logout remote (and reattach here).
-D -RR        Do whatever is needed to get a screen session.
-e xy         Change command characters.
-f            Flow control on, -fn = off, -fa = auto.
-h lines      Set the size of the scrollback history buffer.
-i            Interrupt output sooner when flow control is on.
-l            Login mode on (update /var/run/utmp), -ln = off.
-ls [match]   or -list. Do nothing, just list our SockDir [on possible matches].
-L            Turn on output logging.
-m            ignore $STY variable, do create a new screen session.
-O            Choose optimal output rather than exact vt100 emulation.
-p window     Preselect the named window if it exists.
-q            Quiet startup. Exits with non-zero return code if unsuccessful.
-r [session]  Reattach to a detached screen process.
-R            Reattach if possible, otherwise start a new session.
-s shell      Shell to execute rather than $SHELL.
-S sockname   Name this session <pid>.sockname instead of <pid>.<tty>.<host>.
-t title      Set title. (window's name).
-T term       Use term as $TERM for windows, rather than "screen".
-U            Tell screen to use UTF-8 encoding.
-v            Print "Screen version 4.01.00devel (GNU) 2-May-06".
-wipe [match] Do nothing, just clean up SockDir [on possible matches].
-x            Attach to a not detached screen. (Multi display mode).
-X            Execute <cmd> as a screen command in the specified session.
```
简单的 Options 介绍：
* -S 会话 Session 的名字
* -ls 查看所有会话
* -d 分离会话
* -r 重新连接断掉的会话
* -D 退出会话

## 简单使用
``` bash
$ screen -ls   # 查看当前所有会话
No Sockets found in /var/run/screen/S-root.
$ screen -S test_screen # 新建一个名为 test_screen 的会话

这时候，如果键入了 Enter 键，那么就会进入到 test_screen 的会话了，我们接下来要运行一个命令：ping www.baidu.com

$ ping www.baidu.com
PING www.a.shifen.com (119.75.213.51) 56(84) bytes of data.
64 bytes from 119.75.213.51: icmp_seq=1 ttl=52 time=3.46 ms
64 bytes from 119.75.213.51: icmp_seq=2 ttl=52 time=3.30 ms
64 bytes from 119.75.213.51: icmp_seq=3 ttl=52 time=3.28 ms
(由于我们是 Linux/Unix 系统，这个 ping 的信息会一直打印出来，如果我们 Ctrl + C，则会结束)

现在我们有一个需求，这个 ping 我想让他一直在打印信息，不结束。
我们现在键入如下命令：
Ctrl + A，然后输入 d
```

此时，我们的 test_screen 会话就已经是分离状态了：
[detached from 18717.test_screen]

接下来我们运行如下命令：
```
$ screen -ls
There is a screen on:
	18717.test_screen	(07/11/2017 12:28:15 AM)	(Detached)
1 Socket in /var/run/screen/S-root.
$ screen -r 18717 （重新连接会话）
```
我们会发现，ping www.baidu.com 的信息又多了很多条，由此可见我们的命令在后台正常运行状态。

### 一些小问题

Q&A:

Q1: 如果 screen -ls 的时候查看到一个会话是 attached 状态，要怎么办？
A1: 这是因为这个会话正在被另外一个人登录（或者使用者原本意外退出），执行 `screen -D -r SessionId` 就能重新连接上了。

Q2: 想要结束会话怎么办？
A2: 两种办法：
  * 第一种：screen -S SessionId -X quit
  * 第二种：重连会话，在会话中执行 exit


