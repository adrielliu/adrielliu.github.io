---
layout: post
title:  "iterm2出现pyenv:11: command not found: pyenv，prompt_status:5: command not found: wc!"
date:   2018-09-19 12:31:21 +0800
tags: mac iterm2 pyenv wc
color: rgb(255,90,90)
cover: '../assets/iterm2.jpg'
subtitle: 'Iterm2 command not found!'
---
早上想跑一个go的项目，所以在.zshrc里面添加PATH=$GOPATH/bin，结果就出现了以下问题：
```shell
pyenv:11: command not found: pyenv
prompt_status:5: command not found: wc
env_default:1: command not found: env
env_default:1: command not found: grep
env_default:1: command not found: env
env_default:1: command not found: grep
virtualenvwrapper_derive_workon_home:12: command not found: grep
virtualenvwrapper_derive_workon_home:21: command not found: egrep
virtualenvwrapper_initialize:12: command not found: mkdir
virtualenvwrapper_mktemp:1: command not found: mktemp
virtualenvwrapper_tempfile:6: command not found: touch
ERROR: virtualenvwrapper could not create a temporary file name.
```
结果发现mac的添加path的规则是PATH=$PATH:(需要添加的path)，这样才是添加新path到系统path中，不然系统path就被覆盖了，其他命令自然用不了

修改为export PATH="\$PATH:\$GOPATH/bin"即可