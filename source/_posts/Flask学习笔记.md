date: 2022-06-13 18:00:12

categories: 2022年6月

tags: [Python]

---

Flask学习笔记

<!-- more -->

[TOC]

学习教程参考[^1]

# 命令行启动notebook

python3 -m IPython notebook

-m参数 "mod"是“module”的缩写，即“-m”选项后面的内容是 module（模块），其作用是把模块当成脚本来运行。[^4]


# 内置变量：__name__ 

## 不同情况下的 __name__ 的值[^2]

首先我们需要知道 __name__ 在不同情况下会有不同值，它的值取决于我们是如何执行脚本的.

有时候，我们用 Python 写了一个脚本，当我们既希望这个脚本可以单独运行，同样希望它可以在其他的脚本中发挥作用. 这个时候就需要考虑使用 __name__ 了. 


# 运行遇到的错误及解决方案

## zmq.error.ZMQError: Address already in use [^3]

    app.run(debug = True)

开启debug模式后端口报被占用

之前用的是http://localhost:5000

sudo lsof -i:5000

sudo kill PID

## 查找指定名称（例如python）的进程并显示进程详细信息

ps -ef  | grep python

但是如果关掉flask的debug模式，就没有问题

6、原因是：flask的debug模式会额外开启一个进程，这个进程负责监控代码是否发生变化，如果发生变化，会自动重启应用，使新修改代码立即自动生效；因此我猜想，是这个进程破坏了rpc服务的启动。

DEBUG模式下flask多开一个线程来监视项目的变化。

The first thing it does is start the main function in a new thread so it can monitor >the source files and restart the thread when they change.

参考自这篇文章http://stackoverflow.com/questions/9276078/whats-the-right-approach-for-calling-functions-after-a-flask-app-is-run。

如果你想要避免加载两次，应该设置

    app.run(debug=True, use_reloader=False)


## 文件上传报错：FileNotFoundError: [Errno 2] No such file or directory:

根据代码内容，需要在代码的根目录下创建一个文件夹upload才能保存文件

## Flask WTF : ImportError: cannot import name 'ContactForm' from 'forms' 



这个ContactForm是自己建的forms.py文件中的类

## ImportError: cannot import name 'TextField' from 'wtforms'

原因是版本更新问题

    from wtforms import StringField

使用StringField即可


## AttributeError: module 'wtforms.validators' has no attribute 'Required'

looks like wtforms.validators split the Required into DataRequired and InputRequired in version version 1.0.2

新版本将Required换成InputRequired


## Install 'email_validator' for email validation support.

还是版本兼容问题

## TypeError: __init__() takes 1 positional argument but 5 were given

在 Flask SQLAlchemy中 students 的class初始化函数如下：

    def __init__(self, name, city, addr,pin):

调用的时候赋值：

     student = students(request.form['name'], request.form['city'],
        request.form['addr'], request.form['pin'])

报错：

改成下列调用方式即可

    student = students(name = request.form['name'], city = request.form['city'],
                               addr = request.form['addr'], pin = request.form['pin'])



[^1]:https://www.w3cschool.cn/flask/flask_application.html

[^2]:https://zhuanlan.zhihu.com/p/157439994

[^3]:https://www.cnblogs.com/shengulong/p/8207179.html

[^4]:https://zhuanlan.zhihu.com/p/91120727

