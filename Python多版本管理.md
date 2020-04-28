



> **pyenv** 是一款特别好用的Python版本管理器，程序员可以建立不同的目录，在不同的目录里分别运行不同版本的Python， 并且互不影响，安装的包也互不影响。github项目地址：[https://github.com/yyuu/pyenv](https://link.jianshu.com?t=https://github.com/yyuu/pyenv)



> **pyenv-virtualenv** 是pyenv的一个plugin（插件），可以用来创建基于不同Python版本的干净的虚拟环境。github项目地址：[https://github.com/yyuu/pyenv-virtualenv](https://link.jianshu.com?t=https://github.com/yyuu/pyenv-virtualenv)



安装：

```shell
brew install pyenv

brew install pyenv-virtualenv
# 按照安装完后Caveats的提示要添加两条环境变量到~/.bash_profile文件里：

if which pyenv-virtualenv-init > /dev/null; then eval "$(pyenv virtualenv-init -)"; fi

# 需要使环境变量生效，运行命令exec "$SHELL"，如果还没有效果，就退出终端，重新打开。

# 查看当前版本
pyenv version

# 查看当前系统有哪些版本 *表示当前使用的版本
pyenv versions 


# 查看python版本列表
pyenv install --list
➜  ~ pyenv install --list
Available versions:
  2.1.3
  2.2.3
  2.3.7
  2.4.0
  2.4.1
  2.4.2
  2.4.3
  2.4.4
  2.4.5
  2.4.6
  2.5.0
  .........
  
# 安装某个版本
pyenv install 3.x.x

# 卸载某个版本
pyenv uninstall 3.x.x

#安装/卸载完成后必须rehash database
pyenv rehash



# # 
# https://www.jianshu.com/p/37576a6de65b
# https://www.jianshu.com/p/1842a363257c
# #



```

