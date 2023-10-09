# 背景

我目前有两个 gitee 账户，一个公，一个私。

因为 gitee 限制了，一个 ssh key 只能用于一个 gitee 账户。

所以我只能各自生成一个 ssh key。

我需要提交代码分别到两个 git 仓库，这就导致了私人账户的 git 仓库，除非每次用 https 来提交代码，用 git 方式提交代码我都不知道怎么玩。

# 解决

用 `ssh-keygen` 生成一个新的ssh key:

```text
 ~/.ssh/id_rsa_haha
```

vi ~/.ssh/config:

```text
Host hahagitee
  HostName gitee.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_rsa_haha
  User root
```

然后 git clone的时候：

```text
 git clone git@newgitee:miehaha/MyGit.git 
```

这样子就可以顺畅的git push, git pull了，和第一个 gitee 账户完全隔离.