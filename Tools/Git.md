##### 常用的配置个人的用户名称和电子邮件：

  ⚡ git config --list 查看当前配置信息 

  ⚡ git config --global user.name "用户名"

  ⚡  git config --global user.email 用户邮箱地址



##### 修改项目提交地址

  ⚡ git    remote rm  origin  移除已有地址

  ⚡ git remote add origin git仓库地址 添加已有地址

  ⚡ git push --all  origin       一次提交所有分支

  ⚡ git push origin --tags     一次提交所有tag

  ⚡ git rm -r --cached *.class   忽略所有的 class文件



##### 测试SSH配置链接

  ⚡  ssh -T remotesource    测试配置

##### mac下配置多个git账号并进行账号切换

1. 打开[GitHub](https://link.jianshu.com/?t=https://github.com) 注册账号

2. 进入Finder目录 `~/.ssh`

3. 会看到有三个文件config、id_rsa、id_rsa.pub（我配置过两个账号，是五个文件）

4. 打开终端 

   输入第一行命令：

   ```
   ssh-keygen -t rsa -C "maile@maile.com"
   ```

   这时会提示你创建的文件是否使用默认的文件名

   ```
   Enter file in which to save the key (/Users/用户名/.ssh/id_rsa):
   ```

   如果第一次使用了id_rsa，为了避免覆盖第一次的账号配置，现在需要修改名为id_rsa_wangming如果不需要密码，接下来可以直接两次回车

5. 这时再看.ssh文件夹下多了两个文件：

   ```
   id_rsa_wangming
   
   id_rsa_wangming.pub
   ```

   

6. 用记事本打开id_rsa_wangming.pub，复制里面全部内容

7. 回到GitHub网页，找到设置里的SSH配置

8. 把刚刚复制的内容粘贴到key里面，起个名字，直接Add SSH key就OK了
   到此，已经配置完了新的git账号，接下来是如何切换两个git账号

9. 再回到.ssh文件夹，用记事本打开config文件将原来的配置信息改为新账号绑定的配置信息

   ```shell
   Host OSChina
         HostName git.oschina.net
         User git
         IdentityFile ~/.ssh/id_rsa
   Host GitHub
         HostName github.com
         User git
         IdentityFile ~/.ssh/id_rsa_wangming
   ```

   

10. 再找到和.ssh文件夹同一层级下的.gitconfig文件，用记事本打开
      将原来的账号信息改为新账号
      [user]
      name = wangming-github
      email = myoneray@outlook.com
      到此已经完成了新git账户的配置和切换

    使用命令：ssh -T git@gitlab.alibaba-inc.com 测试
