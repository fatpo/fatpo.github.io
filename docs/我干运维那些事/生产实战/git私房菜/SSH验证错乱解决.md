# 背景

之前说过了，我有两个 gitee ssh 账户。

但是今天发现我在账户 2 提交代码后，在账户 1 这边 git pull 就提示验证失败。

不能理解：

```text
[session-9423a8af] Auth error: Access denied
致命错误：无法读取远程仓库。

请确认您有正确的访问权限并且仓库存在。
```

# 验证

```text
ssh -T git@gitee.com
```

需要更多信息：

```text
ssh -T -v git@gitee.com
```

# 解决

我确定我的ssh 配置是对的。

如果你仍然遇到问题，可能是因为SSH代理（SSH Agent）中仍然包含了错误的密钥。你可以使用以下命令清除SSH代理中的所有密钥：

```text
ssh-add -D
```

然后，重新添加正确的密钥：

```text
ssh-add ~/.ssh/id_rsa
ssh-add ~/.ssh/id_rsa_haha
```

再次验证 ok.
