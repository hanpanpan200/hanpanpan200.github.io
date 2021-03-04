title: 通过 ssh config 配置 Git 多账户 SSH 登录
date: 2019-10-14 16:22:46
category: Git
---

当我们使用 git 时，常用的授权通常有两种方式 HTTPS 和 SSH。当选用 HTTPS 时，每次代码的增删改查都需要输入一串密码进行身份验证，影响代码提交效率，因此大部分情况下我们会使用 SSH 免密登录。那么，在使用 SSH 免密登录过程中，你有遇到过什么问题吗？

<!-- more -->

## SSH 工作原理

>SSH(Secure Shell) 是一种加密的网络传输协议，可以在不安全的网络中为网络服务提供安全的传输环境。SSH 以非对称加密实现身份验证，通过C/S(client-server)模式来实现。

具体交互见下图：

<img src="/images/setup-multiple-git-accounts-by-ssh-config/ssh-working-model.png" alt="Working Model" />

在我们上传代码到 github 服务器的过程中，github server 就会作为 SSH server 来接受客户端的请求，并返回 public key（即公钥） 给客户端。而客户端将会进行 public key 和 private key（即私钥）的匹配校验，最终建立连接。

在明白了 SSH 的原理之后，你可能会有点疑惑，这个公钥和私钥是怎么来的呢？下面我们就来解答这些疑问。

## 基本的 Github SSH 配置

Git SSH 的 公钥和私钥都是在客户端手动生成的。一般我们会将私钥保存在客户端，公钥配置在服务端。此处以 github 为例，我们一起来进行 SSH 的配置。

### 配置 SSH 秘钥

- 根据 [github 文档](https://help.github.com/en/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#adding-your-ssh-key-to-the-ssh-agent)，在客户端创建SSH公钥和私钥对，并将私钥添加到客户端的 ssh-agent 中。
- [将公钥配置在 github 的相应账户中](https://help.github.com/en/articles/adding-a-new-ssh-key-to-your-github-account)

- 测试配置是否成功

```
ssh -T git@github.com
```

如果看到如下结果，则证明配置成功，否则需要根据错误消息进行修改。
> Hi yourusername! You've successfully authenticated, but GitHub does not provide shell access

## 配置多个 github 账户的 SSH

在完成了如上的配置之后，你愉快的用起来了 SSH ，代码提交效率大幅提高了，git pull , git push 的时候再也不需要输入密码了。你对当前的效果甚是满意。

然而过了不久，你切换到了另一个项目/公司，公司给你分配了一个企业邮箱，需要你在工作时使用这个企业邮箱进行代码提交，于是你用公司邮箱创建了另外一个 Github 账户。但是你的 SSH 配置怎么办？于是你想了一下，哦，对，我就把原来的公钥再配置到这个 github 账户里吧！于是你尝试着将本地的 id_rsa.pub 文件中的内容拷贝到公司 github 账户的 SSH settings 里，当你尝试保存的时候得到一个错误消息:

<img src="/images/setup-multiple-git-accounts-by-ssh-config/error.png" />

从原来的个人 github 账户中删除 SSH 配置，然后将 SSH 配置到公司 github 账户中？你觉得不行，你时不时还需要往自己的个人 github 账户中提交代码呢，总不能每次提交都把 SSH 配置挪来挪去吧！

哦，你想起来，SSH 还提供了一个 SSH Config，可以在这个 config 中定义不同的 host，可以在创建多对公钥和私钥，可以把不同的公钥配置在不同的 github 账户里，然后通过脚本去连接到不同的host，完成不同 git 账户的身份验证。

### 配置多个账户的私钥和公钥对，用客户端的 ssh-agent 进行 key 的切换

本例中，你的个人 github 账户用户名为 personal， 公司 github 账户用户名为 company。

#### 1. 配置多对私钥和公钥

重复 [配置 SSH]() 中的步骤，分别为个人和公司的 github 账户创建对应的私钥和公钥，此时执行  `ls ~/.ssh` 将会看到如下结果：

```
id_rsa_github_personal
id_rsa_github_personal.pub
id_rsa_github_company
id_rsa_github_company.pub
```

将 `id_rsa_github_personal.pub` 和 `id_rsa_github_company.pub` 分别添加到个人和公司 github 账户的 SSH 配置中。

##### 2. 访问不同 github 账户的 repo 时，手动重置 SSH 客户端的默认私钥

当访问公司 github 账户对应的 repo 时，执行：

```
ssh-add -D
ssh-add ~/.ssh/id_rsa_github_company
```

当访问个人 github 账户对应的 repo 时，执行：

```
ssh-add -D
ssh-add ~/.ssh/id_rsa_github_personal
```

这样，你通过这种半自动化的方式配置好了两个 github 账户的 SSH，在实际使用过程中，勉强够用，但是你经常忘记切换，常常是在拉代码收到错误提示之后，才想起来要手动做这个切换，这确实有点痛苦。你在想，如果拉代码时，SSH 可以自动匹配私钥和公钥该多好？

### 让 SSH 自动识别应该用哪个私钥和公钥进行匹配

本例中，个人 github 仓库的 ssh 地址为：
> git@github.com:personal-username/repo1.git

公司 github 仓库的ssh 地址为：
> git@github.com:company-username/repo2.git

在以上的 ssh-url 中，**github.com** 是 ssh server 的 host name。当连接这个 ssh url 时，客户端的 ssh-agent 会在本地的 ssh config 中寻找相应的配置。接下来，我们来查看本地的ssh config。

```
cat ~/.ssh/config
```

默认配置为：

```
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_rsa
```

- Host：标识一个配置区间，此处 `*` 意为匹配所有的 host
- IdentityFile: 指定秘钥认证使用的私钥文件路径，此处默认为 `~/.ssh/id_rsa`

我们可以为不同 github 账户来配置多个 host，分别指向其对应的私钥。

#### 1. 创建 id_rsa_github_personal 的私钥和公钥
#### 2. 创建 id_rsa_github_company 的私钥和公钥
#### 3. 将私钥添加到 ssh-agent 并保存在 keychain 中

```bash
ssh-add -K ~/.ssh/id_rsa_github_personal
ssh-add -K ~/.ssh/id_rsa_github_company
```

#### 4. 更新 ssh config

将 config 修改为:

```
Host personal-username.github.com
  HostName github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_rsa_github_personal
Host company-username.github.com
  HostName github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_rsa_github_company
```

#### 5. 拉代码时配置对应的 host

github 默认的 ssh host 名字为 **github.com** ，我们已经在上述的配置中，根据不同的 github 账户，将 **github.com** 这个 host 区分为了 **personal-username.github.com** 和 **company-username.github.com**， 因此，当我们在访问对应仓库时，对应的 ssh host 应当作出对应的修改。

当使用个人 github 账户的 SSH 认证访问仓库： git@github.com:personal-username/ssh-config-demo.git

- 如果 repo 已经 clone 到本地，则需要修改 `./git/config` 中的 `remote origin` 配置。具体的 diff 如下：

<img src="/images/setup-multiple-git-accounts-by-ssh-config/git-config.png" alt="Git Config" />

- 如果 repo 未 clone 到本地，则需要修改 clone 命令：

<img src="/images/setup-multiple-git-accounts-by-ssh-config/clone-command.png" alt="Clone Command" />

按照如上进行配置之后，只需要在 clone 代码处注意一下 ssh host 的处理，便可以后顾无忧了！你可以放心的在各个 github 账户间切换自如。

## 再添加一个 bitbucket 账户的 SSH 配置

又有那么一天，你又需要 [bitbucket](https://bitbucket.org) 了，可能是某个项目需要，也有可能是你的私有仓库需要大于3个的 collaborator，但是无论如何，你的确是，又需要配置 bitbucket 的 SSH 了。不过，你现在已经可以如法炮制了！

- 为 bitbucket 账户创建一个秘钥对 `id_rsa_bitbucket_personal` 和 `id_rsa_bitbucket_personal.pub`
- 将公钥 `id_rsa_bitbucket_personal.pub` 添加到 bitbucket 的账户设置中
- 修改 ssh config，新增一个 host

```
Host personal-username.github.com
  HostName github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_rsa_github_personal
Host company-username.github.com
  HostName github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_rsa_github_company
Host personal-username.bitbucket.org
  HostName bitbucket.org
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_rsa_bitbucket_personal
```
- clone repo 时，修改 bitbucket 默认的 host，比如，从  

```
git clone git@bitbucket.org:personal-username/demo.git
```

修改为

```
git clone git@personal-username.bitbucket.org:personal-username/demo.git
```

- 测试是否配置成功

```
ssh -T git@bitbucket.org
```


## 码云的 SSH ？gitlab 的 SSH ？

剩下的就不用我多说什么啦，所有基于 git 的代码托管平台都可以以此类推了！
