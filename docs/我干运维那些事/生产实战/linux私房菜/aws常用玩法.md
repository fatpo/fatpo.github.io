## aws入门教程

https://docs.aws.amazon.com/zh_cn/cli/latest/userguide/cli-services-s3-commands.html

最核心无非就是列表、上传、下载。

### ls 列表

s3 ls 示例
以下示例列出您的所有 Amazon S3 存储桶。

```text
$ aws s3 ls
2018-12-11 17:08:50 my-bucket
2018-12-14 14:55:44 my-bucket2
```

下面的命令列出一个存储桶中的所有对象和前缀。在本示例输出中，前缀 example/ 有一个名为 MyFile1.txt 的文件。

```text
$ aws s3 ls s3://bucket-name
PRE example/
2018-12-04 19:05:48          3 MyFile1.txt
```

您可以通过在命令中包含特定前缀将输出筛选到此前缀。下面的命令列出 bucket-name/example/ 中的对象（即 bucket-name 中按前缀
example/ 筛选出的对象）。

```text
$ aws s3 ls s3://bucket-name/example/
2018-12-06 18:59:32          3 MyFile1.txt
```

### cp 上传和下载

以下示例将所有对象从 s3://bucket-name/example 复制到 s3://my-bucket/。

```text
$ aws s3 cp s3://bucket-name/example s3://my-bucket/
```

以下示例使用 s3 cp 命令，将本地文件从当前工作目录复制到 Amazon S3 存储桶。

```text
$ aws s3 cp filename.txt s3://bucket-name
```

以下示例将文件从 Amazon S3 存储桶复制到当前工作目录，其中 ./ 指定当前的工作目录。

```text
$ aws s3 cp s3://bucket-name/filename.txt ./
```

以下示例使用 echo 将文本“hello world”流式传输到 s3://bucket-name/filename.txt 文件。

```text
$ echo "hello world" | aws s3 cp - s3://bucket-name/filename.txt
```

以下示例将 s3://bucket-name/filename.txt 文件流式传输到 stdout，并将内容输出到控制台。

```text
$ aws s3 cp s3://bucket-name/filename.txt -
hello world
```

以下示例将 s3://bucket-name/pre 的内容流式传输到 stdout，使用 bzip2 命令压缩文件，并将名为 key.bz2 的新压缩文件上传到 s3:
//bucket-name。

```text
$ aws s3 cp s3://bucket-name/pre - | bzip2 --best | aws s3 cp - s3://bucket-name/key.bz2
```

## 实战多环境

因为项目需要，有多个环境，live，test，uat，staging 等，这里用 live 和 test 来演示。

### 初始化配置

配置两个快捷命令： vi ~/.zshrc

```text
alias awstest="export AWS_PROFILE=test"
alias awslive="export AWS_PROFILE=live"
```

激活一下：

```text
source ~/.zshrc
```

配置aws的 config, cat ~/.aws/config

```text
[default]
region = live
output = text

[profile live]
region = live
output = text

[profile test]
region = test
output = text
```

配置aws的 credentials, cat ~/.aws/credentials

```text
[test]
aws_access_key_id = 123
aws_secret_access_key = hahahaha
region = test

[live]
aws_access_key_id = 456
aws_secret_access_key = heiheihei
region = live

[default]
aws_access_key_id = xxx
aws_secret_access_key = eeee
```

### 切换环境

切到 test 环境，命令行输入：awstest
切到 live 环境，命令行输入：awslive

### 命令

测试环境

```text
列表  aws s3 ls haha-search-test  --endpoint-url=http://proxy.uss.s3.test.haha.io
下载  aws s3 cp s3://haha-search-test/query_rewrite_offline_res_vn_test_base.txt ./ --endpoint-url=http://proxy.uss.s3.test.haha.io
上传  aws s3 cp 123.txt s3://haha-search-test --endpoint-url=http://proxy.uss.s3.test.haha.io
```

生产环境

```text
列表  aws s3 ls  haha-search-live --endpoint-url=http://proxy.uss.s3.sz.haha.io
下载  aws s3 cp s3://haha-search-live/query_rewrite_offline_data_vn_live.txt ./   --endpoint-url=http://proxy.uss.s3.sz.haha.io
上传  aws s3 cp 123.txt s3://haha-search-live --endpoint-url=http://proxy.uss.s3.sz.haha.io
```
