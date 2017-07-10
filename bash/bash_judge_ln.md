## Bash 脚本判断文件是否增加软链接

<b>解决问题：</b> 判断某个目录下某个文件是否是软链接

### 首先初始化文件目录

```
$ cd ~/tmp
$ mkdir ln_test && cd ln_test
$ touch test.sh
$ mkdir dir1 && touch dir1/dir.sh
$ mkdir dir2 && touch dir2/dir.sh
$ mkdir dir3 && touch dir3/dir.sh
$ ln -sf test.sh dir3/dir.sh
```

### 使用 tree 命令查看当前文件目录

```
$ tree .
.
├── dir1
│   └── dir.sh
├── dir2
│   └── dir.sh
├── dir3
│   └── dir.sh -> test.sh
└── test.sh

3 directories, 4 files
```

可以发现 `dir3/dir.sh` 文件已经建立了 `test.sh` 文件的软链接了

> <b>简单补充一下软链接命令的使用：</b>
>
> ln [OPTIONS] TARGET LINK_NAME（TARGET 是要软链的目标文件或目录，LINK_NAME 是要被软链的目录文件或目录）
>
> -s, --symbolic              make symbolic links instead of hard links（符号链接）
> -f, --force                 remove existing destination files（强制执行）

ln 的使用可参考文章：http://www.cnblogs.com/peida/archive/2012/12/11/2812294.html

### 写脚本

<b>明确目标：</b>如果 `dir` 目录下的 `dir.sh` 没有软链接的话，要加上

```shell
#!/bin/bash
set -o pipefail

DIR=$PWD
ABS_PATH=$DIR/test.sh

add_ln() {
  for d1 in $1/*
  do
    # 如果 d1 不是目录，则跳过此次循环
    if [ ! -d $d1 ]
    then
      continue
    fi

    # 如果 d1/dir.sh 不存在，则添加软链
    if [ ! -f $d1/dir.sh ]
    then
      ln -sf $ABS_PATH $d1/dir.sh
      continue
    fi

    # 如果 d1/dir.sh 存在，则对比链接的绝对路径是否相同
    LINK_PATH=`readlink -f $d1/dir.sh`
    if [ $ABS_PATH = $LINK_PATH ]
    then
      echo "Equals"
    else
      ln -sf $ABS_PATH $d1/dir.sh
    fi
  done
}

add_ln $DIR

unset DIR
unset ABS_PATH
unset LINK_PATH

echo "Completed!"
```

其实写这么麻烦是为了演示，如果真要实现这个的话，检查都不用检查，直接一梭子把 dir*/dir.sh 文件强制重新刷新软链就行，顶多再加一个判断 dir*/dir.sh 的绝对路径是不是正确的。
