# 1、diff 和 patch

## 1.1、用 diff 创建 patch
首先准备 2 个文件：
```text
# cat source.txt                                                                       
source
123
456
# cat target.txt                                                                       
target
123
456
```

创建从 source 到 target 的 patch:
```text
diff -nN source.txt target.txt < 1.patch
```

我们看看 `1.patch` 长什么样子：
```text
# cat 1.patch                                                                          
--- source.txt	2023-02-16 16:38:32.000000000 +0800
+++ target.txt	2023-02-16 16:43:41.000000000 +0800
@@ -1,3 +1,3 @@
-source
+target
 123
 456
```

## 1.2、用 patch 完成 source 转换成 target
接下来我们用 1.patch 把 source.txt 变成 target.txt： 
```text
# patch -p0 < 1.patch                                                                    
# cat source.txt                                                                             
target
123
456
```

## 1.3、用 patch -RE 回滚
接下来我们用 1.patch 把 source.txt 回滚为最初的文本：
```text
# patch -RE -p0  < 1.patch                                                                
patching file source.txt
# cat source.txt                                                                         
source
123
456
```
## 1.4、文件夹亦如此
```text
diff  -urN source_dir target_dir > 2.patch
patch -p1     < 2.patch
patch -p1 -PE < 2.patch
```
其中的 `-r` 表示递归目录。


## 1.5、参数介绍
diff:
```text
-c                 显示相异处的前后部分内容，并标出不同之处。
-p                 若比较的文件是C语言文件，则列出差异所在的函数。
-r                 递归地比较目录下的所有文件。
-u                 以统一格式输出其差异（默认是3行，但是可以通过“-u  NUM”或"--unified[=NUM]” 来改变）。
-i, --ignore-case  忽略大小写的差异（即大小写视为相同）。
-a, --text         把所有文件都当作文件文件来看待（即可以包含二进制文件）。
-x  FILE/DIR       不比较指定的名为FILE的目录或名为DIR的目录。
-X file            把file中列出的文件和目录都忽略掉，不进行diff。在file中，每行一个文件和目录。
-N, --new-file     把不存在的文件作为一个空文件，把不存在的目录作为一个空目录。
```
patch；
```text
-d  DIR                  在进行任何操作前先切换到目录DIR中
-E, --remove-empty-file  移除空文件/空目录（与diff中的-N选项对应）
-R, --reverse            变换新旧文件（即把新文件还原成旧文件）
-pN, --strip=N           N为数字，代表补丁文件名左边目录的层数（即“/”的个数）。该选项表示把补丁文件名中前N层目录去掉。
-I  patchfile, --input=patchfile  指定补丁文件的位置，如果patchfile为“-”，则从标准输入读取。这是默认选项。
```

# 2、bindiff

直接上链接吧，就当是一个知识扩展： [BINDIFF二进制比较简介](http://blog.nsfocus.net/bindiff/)

进入是一个软件，看起来是跑在 windows 上，用于静态代码分析，能快速定位到两个 二进制之间的 diff，进而帮助早点定位到感兴趣的代码。



# 3、原文搬运
* [diff 和 patch](https://github.com/xgfone/snippet/blob/master/snippet/docs/tool/diff%26patch.md)
* [BINDIFF二进制比较简介](http://blog.nsfocus.net/bindiff/)