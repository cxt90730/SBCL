# SBCL(Steel Bank Common Lisp)脚本启动笔记

作者：安静1337

链接：https://www.jianshu.com/p/dfebe7b299cd

Steel Bank Common Lisp

当sbcl以脚本形式(sbcl --script)运行，它不会加载任何额外文件。当我们的脚本中有其他依赖时，就会变得很棘手。以下，以我自己实际经验(一个签到的小功能)介绍怎么配置sbcl的脚本环境。

首先，需要一个调用脚本，文件名称是sign.lisp。里面的内容很简单，但是需要依赖一个其他的系统
sign.lisp:

(in-package :cl-user)
(require :cl-163-music)

(multiple-value-bind (res0 res1)
 (cl-163-music:daily-sign "hehe@sbcl.com" "sbwy")
 (format t "~A ~% ~A ~%" res0 res1))
然后用shell调用sbcl加载sign.lisp

#!/usr/bin/env sh
PATH="/Users/nero/devel/shell/163.music.sign/"
SBCL_PATH="/Users/nero/tanshuai/sbcl/bin/"

${SBCL_PATH}sbcl --noinform --core ${PATH}core --script ${PATH}sign.lisp
shell也很简单，但是里面有一个--core选项。好了，重点来了，core文件是什么？怎么生成的？

core文件就是一个环境镜像文件，它的作用就是解决脚本的依赖，提供一个运行时环境包，含了当时运行时刻所有状态，我们可以先加载所有的必须条件后，用sb-ext:save-lisp-and-die生成出的镜像文件。

由于cl-163-music符合asdf结构，此处我用了lisp的三方管理包quicklisp(类似于python的pip，nodejs的npm等等，至于怎么安装的此处不再叙述)，quicklisp会自动管理asdf项目，解决cl-163-music的依赖。cl-163-music是自己的本地项目，并且sign.lisp里(require :cl-163-music)的需要，所以生成镜像前需要(push #P"/Users/nero/devel/lisp/cl-163-music/" asdf:*central-registry*)。
到此，所有工作都完成了，这里有个小插曲，当我运行的时候，报错了：

ASDF could not load cl-163-music because
Don't know how to REQUIRE sb-rotate-byte.
See also:
  The SBCL Manual, Variable *MODULE-PROVIDER-FUNCTIONS*
  The SBCL Manual, Function REQUIRE.
Unhandled SB-INT:EXTENSION-FAILURE in thread #<SB-THREAD:THREAD
                                               "main thread" RUNNING
                                                {10033AEC13}>:
  Don't know how to REQUIRE sb-rotate-byte.
See also:
  The SBCL Manual, Variable *MODULE-PROVIDER-FUNCTIONS*
  The SBCL Manual, Function REQUIRE
... ...
...
解决办法就是，再次进入sbcl环境，运行下(require "sb-rotate-byte")，然后重新生成镜像。

ps：缺点就是镜像文件太大了，大约50多MB。


来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
