$0   文本当前行的全部内容
$1   文本的第1列
$2   文件的第2列
$3   文件的第3列，依此类推
NR   文件当前行的行号
NF   文件当前行

```bash
$0   文本当前行的全部内容
$1   文本的第1列
$2   文件的第2列
$3   文件的第3列，依此类推
NR   文件当前行的行号
NF   文件当前行的列数（有几列）
FS   字段分隔符，默认是空格
FNR  各文件分别记行号
FILENAME  文件名
awk 'BEGIN{ commands } pattern{ commands } END{ commands }'
BEGIN{ }    行前处理，读取文件内容前执行，指令执行1次
  { }         逐行处理，读取文件过程中执行，指令执行n次
  END{ }     行后处理，读取文件结束后执行，指令执行1次
  
```

awk 'BEGIN{ commands } pattern{ commands } END{ commands }'

