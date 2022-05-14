brew install安装的所有包都是在==/usr/local/Cellar/==这个路径下

换源

* [中科大](https://mirrors.ustc.edu.cn/)

* [清华](https://mirrors.tuna.tsinghua.edu.cn/)

* [阿里](https://developer.aliyun.com/special/mirrors/notice)

  ```shell
  ############清华源
  # 替换brew.git源
  git -C "$(brew --repo)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git
  # 替换 homebrew-core.git源
  git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
  # 替换 homebrew-cask.git源
  git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask.git
  
  ############中科大的源
  # brew.git源
  git -C "$(brew --repo)" remote set-url origin https://mirrors.ustc.edu.cn/brew.git
  # homebrew-core.git源
  git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
  # homebrew-cask.git源
  git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git
  
  ############查看
  # 查看brew.git源
  git -C "$(brew --repo)" config --get remote.origin.url
  # 查看homebrew-core.git源
  git -C "$(brew --repo homebrew/core)" config --get remote.origin.url
  # 查看homebrew-cask.git源
  git -C "$(brew --repo homebrew/cask)" config --get remote.origin.url
  
  ##########官方源
  # brew.git源
  git -C "$(brew --repo)" remote set-url origin https://github.com/Homebrew/brew.git
  # homebrew-core.git源
  git -C "$(brew --repo homebrew/core)" remote set-url origin https://github.com/Homebrew/homebrew-core
  # homebrew-cask.git源
  git -C "$(brew --repo homebrew/cask)" remote set-url origin https://github.com/Homebrew/homebrew-cask
  ```

  > 指令

```shell
https://www.jianshu.com/p/d2f8ed1127a3
查看所有包 brew list
安装包路径 /usr/local/Cellar/
//更新包
brew update
brew outdated         #会安装新版本的包，但旧版本仍然会保留。
brew upgrade             # 更新所有的包
brew upgrade $FORMULA    # 更新指定的包
brew cleanup             # 清理所有包的旧版本
brew cleanup $FORMULA    # 清理指定包的旧版本
brew cleanup -n          # 查看可清理的旧版本包，不执行实际操作
//锁定不想更新的包
brew pin $FORMULA      # 锁定某个包
brew unpin $FORMULA    # 取消锁定
brew info $FORMULA    # 显示某个包的信息
brew info             # 显示安装了包数量，文件数量，和总占用空间
brew deps           #可以显示包的依赖关系，我常用它来查看已安装的包的依赖，然后判断哪些包是可以安全删除的。

#切换版本
brew unlink opencv@4
brew link opencv@3
opencv_version
```

