## Bash 脚本读取文件，根据空格分割行

简单的按行读取文件，然后根据空格分割行

### 创建文件
```
$ touch list
$ cat list
Hello World
```

### 写脚本

```bash
#/bin/bash
set -o pipefail

while read LINE
do
    read -a ITEMS <<< $LINE
    for ITEM in ${ITEMS[@]}
    do
        echo $ITEM
    done
done < list
```

ITEMS 是一个 Array 数组：
* 如果根据索引检出单个值，则用：${array[0]}  
* 取得全部值，则用：${array[@]} 

<b>Attention:</b> $ITEMS 只会取到第一个值
