如何在本地配置多个GitHub账号

在平时的工作中，经常会遇到需要使用两个GitHub账户，一个自己的账户，一个工作的账户。 所以接下来讲解如何在本地配置两个GitHub账号，保证可以同时使用。

1. 创建ssh密钥，在GitHub上绑定密钥
有两个不同的GitHub账号，所以需要有两个ssh密钥。

1.1 分别使用两个GitHub 的邮箱，生成两个ssh密钥文件，需要给不同的账号取不同的名字。在执行命令的过程中需要一直回车就行。

ssh-keygen -t rsa -f ~/.ssh/id_rsa -C "personal@outlook.com"
ssh-keygen -t rsa -f ~/.ssh/id_rsa_work -C "work@outlook.com"
1.2 生成的密钥存放在 /Users/name/.ssh 路径下，打开公钥文件 id_rsa.pub/id_rsa_work.pub，
复制所有内容，在GitHub上打开Setting -> SSH and GPG keys -> add SSH key，将复制的内容粘贴在里边，保存。

2. 创建Config文件，配置多个账号
2.1 在/Users/name/.ssh路径下，创建一个config文件，添加下边代码在config文件中。

Host: 在Clone仓库的时候使用，默认的使用github.com， 所以要给工作的GitHub账号起一个不一样的Host
IdentityFile: 填写对应账号的私钥的文件路径
# Personal account
Host github.com
   HostName github.com
   User git
   IdentityFile ~/.ssh/id_rsa
   
# Work account
Host workgithub.com  
   HostName github.com
   User git
   IdentityFile ~/.ssh/id_rsa_work
2.2 执行以下命令来开启 ssh 服务

ssh-add ~/.ssh/id_rsa
ssh-add ~/.ssh/id_rsa_work
2.3 验证ssh服务是否开启，需要注意的是，-T 后边跟的内容就是 git@"config中Host的内容"，也就是clone仓库的内容。

ssh -T git@github.com
ssh -T git@workgithub.com
如果开启成功，就会看到如下的内容，测试成功之后就可以工作了。

3. 使用的Git命令
3.1 Clone代码：

git@github.com 是 git@"config中Host的内容"，所以工作的账户就是git@workgithub.com
test/test.git 指的就是仓库的地址
git clone git@github.com:test/test.git
git clone git@workgithub.com:test/test.git 
3.2 在对应的仓库路径下配置user和email

git config user.email "personal@outlook.com"
git config user.name "personal"
git config user.email "work@outlook.com"
git config user.name "work"

作者：爱sa笑的菇凉
链接：https://www.jianshu.com/p/b15f2b5d87c6
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。