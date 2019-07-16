# 总相关

* 首先你要有一个速度还不错的网络 :p

# http(s) clone 提速

* 备注: 你使用http clone仓库或者pull. 缺点 不设置配置文件 你总是要验证密码和用户. 优点 较ssh方便 容易弄.
1. 首先有一个ss
2. 然后配置本地git的代理config。使用下面指令或者直接修改本地git的config，Mac环境在~/.gitconfig
```
git config --global http.proxy socks5://127.0.0.1:1080   // 注意这里的1080端口每个人不一样 你得看你的ss socks5走的哪个端口 修改成对应的
git config --global https.proxy socks5://127.0.0.1:1080
```

3. 修改本地host 
  * 通过https://www.ipaddress.com 查询github.global.ssl.fastly.net和github.com的ip地址
  * 输入:
  ```
  sudo vi /etc/hosts
  ```
  * 修改hosts里面内容 新增github.global.ssl.fastly.net和github.com的映射。例如
  ```
  151.101.185.194 github.global.ssl.fastly.net
  140.82.113.4 github.com
  ```
4. clone http对应的仓库已经没有问题了


# ssh clone 提速

* 备注: 你使用ssh clone仓库或者pull。缺点 比较麻烦。 优点 不用总输入密码和用户名了 emmmmm

1. 完成http(s) clone 提速部分的步骤。（其实可以不弄 但懒的挑内容了 就弄完上面4步吧 总不会错的）
2. 首先你要生产ssh-key 并加到github的设置里面。自行谷歌
3. 增加sshconfig文件。Mac环境 ssh client有两个配置文件，/etc/ssh/ssh_config和~/.ssh/config，前者是对所有用户，后者是针对某个用户，两个文件的格式是一样的。
4. 个人选择在~/.ssh/config配置，config文件开始时没有 自行创建。然后配置进内容
```
Host github.com
User git
ProxyCommand /usr/bin/nc -x 127.0.0.1:1086 %h %p    // 注意这里的1086端口 每个人不一样 你得看你的ss socks5走的哪个端口 修改成对应的
IdentityFile ~/.ssh/id_rsa
```
5. 上面代码里的注释记得删。好了 应该没问题了。如果还不行 那我也不知道了 恭喜你踩了个更深的坑 :p
