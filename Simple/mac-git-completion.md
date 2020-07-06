# Mac终端Git自动补齐配置

**Git通过bash-completion软件包实现命令自动补齐，在Mac下通过Homebrew安装**

## 步骤

* 打开终端(Terminal)，使用命令：`/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`来安装Homebrew(一款软件包管理器)
* 使用命令：`brew install bash-completion`来安装bash-completion
* 安装完成后会提示将`[ -f /usr/local/etc/bash_completion ] && . /usr/local/etc/bash_completion`复制到.bash_profile的文件中.
* 使用命令：`open ~/.bash_profile`来打开文件，然后将之前复制的语句存入其中保存。
* 若没有此文件，则
* 使用命令：`cd ~`进入用户目录
* 使用命令：`touch .bash_profile`创建文件
* 使用命令：`open -e .bash_profile`打开并将之前复制的语句存入其中保存
* 使用命令：`source .bash_profile`更新

## 添加Git补全支持

* 使用命令：`cd /usr/local/opt/bash-completion/etc/bash_completion.d`
* 使用命令：`curl -L -O http https://raw.github.com/git/git/master/contrib/completion/git-completion.bash`
* 使用命令：`brew unlink bash-completion`
* 使用命令：`brew link bash-completion`
* 重启终端，在终端输入部分Git命令并按tab键就能看到效果了，例如：git com + tab
