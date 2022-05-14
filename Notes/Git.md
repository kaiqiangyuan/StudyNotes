> git远程强制覆盖本地

```shell
# 拉取所有更新，不同步；
git fetch --all
# 本地代码同步线上最新版本(会覆盖本地所有与远程仓库上同名的文件)；
git reset --hard origin/master
# 再更新一次（其实也可以不用，第二步命令做过了其实）
git pull
```

配置远程连接

```shell
cd .git 
vim config
[receive]    
	denyCurrentBranch = ignore
```

> 配置远程远程服务器Git

1. 安装Git

   ```shell
   sudo apt-get install git
   git config --global user.name "用户名"
   git config --global user.email "邮箱"
   ```

2. 创建一个Git用户

```shell
useradd git //创建用户，用户名为git 
userdel git //删除用户
passwd git //设置git用户密码
```

3. 创建一个git文件夹，方便将所有git项目放入其中

```shell
cd /home //路径根据自己喜好，此处是在根目录的home里创建 
mkdir git //创建文件夹
Chmod 777 git //设置git文件夹访问权为 所有用户都可以访问
```

4. 使用git用户进入git文件夹创建git仓库

   ```
   su git //进入git用户
   cd git //进入git文件夹
   mkdir BoltStructure //创建项目文件夹，此处以BoltStructure为例 cd BoltStructure //进入文件夹
   git init //git化此文件夹，此时会在该文件夹里生成三个文件
   cd .git //进入.git这个文件中对git仓库进行配置
   vim config //打开config文件进行编译(由于git默认拒绝push操作,所以要修改config文件进行设置)
   [receive]    
   	denyCurrentBranch = ignore
   ```

5. 创建本地仓库

   ```shell
   #进入本地创建的项目
   git init #git化文件夹，使其变为git仓库
   git remote add origin git@49.234.179.217:/home/git/BoltStricture #绑定远程的git库
   git remote -v #查看绑定
   git remote remove origin # 若绑定错误，则删除绑定
   git add .     #将当前目录下所有文件交由git仓库管理
   git commit -m "123" #commit更新本地git库
   git push origin master #push上传到远程git库
   git branch mybranch    # 创建分支
   git checkout mybranch   # 切换分支
   git checkout -b mybranch   # -b为 branch和checkout的合并命令
   git branch -d mybranch   # 删除分支
   git branch -D mybranch   #  强制删除分支
   git branch -a           #   查看远程分支
   git branch            	#   查看本地分支
   git branch -v           #  查看当前分支最后一次提交
   git rebase master       # 更新master主线上的东西到该分支上
   git merge A             # 合并两个分支，A分支的内容merge到当前分支上面
   ```
   
   **·**  git add -A  提交所有变化
   
   **·** git add -u  提交被修改(modified)和被删除(deleted)文件，不包括新文件(new)
   
   **·** git add .  提交新文件(new)和被修改(modified)文件，不包括被删除(deleted)文件

> merge和rebase

**marge 特点**：自动创建一个新的commit如果合并的时候遇到冲突，仅需要修改后重新commit
优点：记录了真实的commit情况，包括每个分支的详情
缺点：因为每次merge会自动产生一个merge commit，所以在使用一些git 的GUI tools，特别是commit比较频繁时，看到分支很杂乱。

**rebase 特点**：会合并之前的commit历史
优点：得到更简洁的项目历史，去掉了merge commit
缺点：如果合并出现代码问题不容易定位，因为re-write了history

**选择**

如果你想要一个干净的，没有merge commit的线性历史树，那么你应该选择git rebase
如果你想保留完整的历史记录，并且想要避免重写commit history的风险，你应该选择使用git merge

