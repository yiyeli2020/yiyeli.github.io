---
title: TensorFlow入门-Mnist手写数字识别
date: 2018-11-22 16:30:34
categories: 2018年11月
tags: [TensorFlow]


---
#  TensorFlow入门-Mnist手写数字识别
记录一下Mnist手写数字识别的练习过程，网上已经有很多类似的教程比如(https://blog.csdn.net/simple_the_best/article/details/75267863，https://blog.csdn.net/cqrtxwd/article/details/79028264）， 但会有些许问题，所以做修改后整理一下
当然最好的入门教程还是TensorFlow中文社区（www.tensorfly.cn)

<!-- more -->



## 准备工作：
 可在http://yann.lecun.com/exdb/mnist/下载mnist数据集，包含

	Training set images: train-images-idx3-ubyte.gz (9.9 MB, 解压后 47 MB, 包含 60,000 个样本)
	Training set labels: train-labels-idx1-ubyte.gz (29 KB, 解压后 60 KB, 包含 60,000 个标签)
	Test set images: t10k-images-idx3-ubyte.gz (1.6 MB, 解压后 7.8 MB, 包含 10,000 个样本)
	Test set labels: t10k-labels-idx1-ubyte.gz (5KB, 解压后 10 KB, 包含 10,000 个标签）
MNIST 数据集来自美国国家标准与技术研究所, National Institute of Standards and Technology (NIST). 训练集 (training set) 由来自 250 个不同人手写的数字构成, 其中 50% 是高中学生, 50% 来自人口普查局 (the Census Bureau) 的工作人员. 测试集(test set) 也是同样比例的手写数字数据.
图片是以字节的形式进行存储, 我们需要把它们读取到 NumPy array 中, 以便训练和测试算法,载入数据
	
	import tensorflow as tf
	from tensorflow.examples.tutorials.mnist import input_data
	mnist=input_data.read_data_sets("MNIST_data/",one_hot=True)
	import os
	import struct
	import numpy as np
	def load_mnist(path,kind='train'):
	    labels_path=os.path.join(path,'%s-labels-idx1-ubyte'%kind)
	    images_path=os.path.join(path,'%s-images-idx3-ubyte'%kind)
	    with open(labels_path,'rb') as lbpath: 
	        magic,n=struct.unpack('>II',lbpath.read(8))
	        labels=np.fromfile(lbpath,dtype=np.uint8)
	    
	    with open(images_path,'rb') as imgpath:
	        magic,num,rows,cols=struct.unpack('IIII',imgpath.read(16))
	        images=np.fromfile(imgpath,dtype=np.uint8).reshape(len(labels),784)
	        
	    return images,labels

load_mnist 函数返回两个数组, 第一个是一个 n x m 维的NumPy array(images), 这里的 n 是样本数(行数), m 是特征数(列数). 训练数据集包含 60,000 个样本, 测试数据集包含 10,000 样本. 在 MNIST 数据集中的每张图片由 28 x 28 个像素点构成, 每个像素点用一个灰度值表示. 在这里, 我们将 28 x 28 的像素展开为一个一维的行向量, 这些行向量就是图片数组里的行(每行 784 个值, 或者说每行就是代表了一张图片). load_mnist 函数返回的第二个数组(labels) 包含了相应的目标变量, 也就是手写数字的类标签(整数 0-9).这里对图片的读取方式做一些解释：

	magic, n = struct.unpack('>II', lbpath.read(8))
	labels = np.fromfile(lbpath, dtype=np.uint8)
看一下 MNIST 网站上对数据集的介绍:
	
	TRAINING SET LABEL FILE (train-labels-idx1-ubyte):
	
	[offset] [type]          [value]          [description] 
	0000     32 bit integer  0x00000801(2049) magic number (MSB first) 
	0004     32 bit integer  60000            number of items 
	0008     unsigned byte   ??               label 
	0009     unsigned byte   ??               label 
	........ 
	xxxx     unsigned byte   ??               label
	The labels values are 0 to 9.

我们首先读入 magic number, 它是一个文件协议的描述, 也是在我们调用 fromfile 方法将字节读入 NumPy array 之前在文件缓冲中的 item 数(n). 作为参数值传入 struct.unpack 的 >II 有两个部分:

>: 这是指大端(用来定义字节是如何存储的); 如果你还不知道什么是大端和小端, Endianness 是一个非常好的解释. (关于大小端, 更多内容可见<<深入理解计算机系统 – 2.1 节信息存储>>)
I: 这是指一个无符号整数.

通过执行下面的代码, 我们将会从刚刚解压 MNIST 数据集后的 mnist 目录下加载 60,000 个训练样本和 10,000 个测试样本.


## 数据预览
此时要将文件解压后放在MNIST_data文件夹下，可视化处理. 从 feature matrix 中将 784-像素值 的向量 reshape 为之前的 28*28 的形状, 然后通过 matplotlib 的 imshow 函数进行绘制:

	import matplotlib.pyplot as plt
	X_train,y_train=load_mnist("MNIST_data/",kind='train')
	
	fig, ax = plt.subplots(
	    nrows=2,
	    ncols=5,
	    sharex=True,
	    sharey=True, )
	
	ax = ax.flatten()
	for i in range(10):
	    img = X_train[y_train == i][0].reshape(28, 28)
	    ax[i].imshow(img, cmap='Greys', interpolation='nearest')
	
	ax[0].set_xticks([])
	ax[0].set_yticks([])
	#plt.tight_layout()
	plt.show()

可以看到一个 2*5 的图片, 里面分别是 0-9 单个数字的图片.还可以绘制某一数字的多个样本图片, 来看一下这些手写样本到底有多不同:
	
	fig,ax=plt.subplots(
	    nrows=5,
	    ncols=5,
	    sharex=True,
	    sharey=True,
	)
	ax=ax.flatten()
	for i in range(25):
	    img=X_train[y_train==7][i].reshape(28,28)
	    ax[i].imshow(img,cmap='Greys',interpolation='nearest')
	    
	ax[0].set_xticks([])
	ax[0].set_yticks([])
	plt.show()

我们也可以选择将 MNIST 图片数据和标签保存为 CSV 文件, 这样就可以在不支持特殊的字节格式的程序中打开数据集. 但是, 有一点要说明, CSV 的文件格式将会占用更多的磁盘空间, 如下所示:
	
	train_img.csv: 109.5 MB
	train_labels.csv: 120 KB
	test_img.csv: 18.3 MB
	test_labels: 20 KB
如果我们打算保存这些 CSV 文件, 在将 MNIST 数据集加载入 NumPy array 以后, 我们应该执行下列代码:
	
	np.savetxt('train_img.csv', X_train,
	           fmt='%i', delimiter=',')
	np.savetxt('train_labels.csv', y_train,
	           fmt='%i', delimiter=',')
	np.savetxt('test_img.csv', X_test,
	           fmt='%i', delimiter=',')
	np.savetxt('test_labels.csv', y_test,
	           fmt='%i', delimiter=',')

一旦将数据集保存为 CSV 文件, 我们也可以用 NumPy 的 genfromtxt 函数重新将它们加载入程序中:
	
	X_train = np.genfromtxt('train_img.csv',
	                        dtype=int, delimiter=',')
	y_train = np.genfromtxt('train_labels.csv',
	                        dtype=int, delimiter=',')
	X_test = np.genfromtxt('test_img.csv',
	                       dtype=int, delimiter=',')
	y_test = np.genfromtxt('test_labels.csv',
	                       dtype=int, delimiter=',')
不过, 从 CSV 文件中加载 MNIST 数据将会显著发给更长的时间, 因此如果可能的话, 还是建议维持数据集原有的字节格式.

## 神经网络结构
流程分为三步：
1.构建卷积神经网络结构
2.构建loss function，配置寻优器
3.训练，测试
在源码中使用了两个卷积层+池化层，最后接上两个全连接层。
第一层卷积使用了32个3x3x1的卷积核，步长为1，边界处理方式为“SAME”（卷积的输入和输出保持相同尺寸），激活函数为ReLu，后接一个2x2的池化层，方式为最大化池化；
第二层卷积使用50个3x3x32的卷积核，步长为1，边界处理方式为“SAME”，激活函数为ReLu，后接一个2x2的池化层，方式为最大化池化；
第一层全连接层：使用1024个神经元，激活函数为ReLu。
第二层全连接层：使用10个神经元，激活函数为Softmax,用于输出结果

## 源码
	
	import tensorflow as tf
	from tensorflow.examples.tutorials.mnist import input_data
	#读取数据
	mnist=input_data.read_data_sets('MNIST_data/',one_hot=True)
	
	sess=tf.InteractiveSession()
	#构建卷积神经网络结构
	#自定义卷积函数
	def conv2d(x,w):
	    return tf.nn.conv2d(x,w,strides=[1,1,1,1],padding='SAME')
	#自定义池化函数
	def max_pool_2x2(x):
	    return tf.nn.max_pool(x,ksize=[1,2,2,1],strides=[1,2,2,1],padding='SAME')
	#设置占位符，尺寸为样本输入和输出的尺寸
	x=tf.placeholder(tf.float32,[None,784])
	y_=tf.placeholder(tf.float32,[None,10])
	x_img=tf.reshape(x,[-1,28,28,1])
	#设置第一个卷积层和池化层
	w_conv1=tf.Variable(tf.truncated_normal([3,3,1,32],stddev=0.1))
	b_conv1=tf.Variable(tf.constant(0.1,shape=[32]))
	h_conv1=tf.nn.relu(conv2d(x_img,w_conv1)+b_conv1)
	h_pool1=max_pool_2x2(h_conv1)
	
	#设置第二个卷积层和池化层
	w_conv2=tf.Variable(tf.truncated_normal([3,3,32,50],stddev=0.1))
	b_conv2=tf.Variable(tf.constant(0.1,shape=[50]))
	h_conv2=tf.nn.relu(conv2d(h_pool1,w_conv2)+b_conv2)
	h_pool2=max_pool_2x2(h_conv2)
	
	#设置第一个全连接层
	w_fc1=tf.Variable(tf.truncated_normal([7*7*50,1024],stddev=0.1))
	b_fc1=tf.Variable(tf.constant(0.1,shape=[1024]))
	h_pool2_flat=tf.reshape(h_pool2,[-1,7*7*50])
	h_fc1=tf.nn.relu(tf.matmul(h_pool2_flat,w_fc1)+b_fc1)
	
	#dropout(随机权重失活)
	
	keep_prob=tf.placeholder(tf.float32)
	h_fc1_drop=tf.nn.dropout(h_fc1,keep_prob)
	
	#设置第二个全连接层
	w_fc2=tf.Variable(tf.truncated_normal([1024,10]))
	b_fc2=tf.Variable(tf.constant(0.1,shape=[10]))
	y_out=tf.nn.softmax(tf.matmul(h_fc1_drop,w_fc2)+b_fc2)
	
	#建立loss function,为交叉熵
	loss=tf.reduce_mean(-tf.reduce_sum(y_*tf.log(y_out),reduction_indices=[1]))
	#配置Adam优化器，学习速率为1e-4
	train_step=tf.train.AdamOptimizer(1e-4).minimize(loss)
	
	#建立正确率计算表达式
	correct_prediction=tf.equal(tf.argmax(y_out,1),tf.argmax(y_,1))
	accuracy=tf.reduce_mean(tf.cast(correct_prediction,tf.float32))
	
	#训练
	tf.global_variables_initializer().run()
	for i in range(20000):
	    batch = mnist.train.next_batch(50)
	    if i%100==0:
	        train_accuracy=accuracy.eval(feed_dict={x:batch[0],y_:batch[1],keep_prob:1})
	        print("ste %d,train_accuracy=%g" %(i,train_accuracy))
	    train_step.run(feed_dict={x:batch[0],y_:batch[1],keep_prob:0.5})
	#训练后用测试集进行测试，输出最终结果
	print("test_accuracy= %h" %accuracy.eval(feed_dict={x:mnist.test.images,y_:mnist.test.labels,keep_prob:1}))

## 代码详解：


	#自定义卷积函数
	def conv2d(x,w):
	    return tf.nn.conv2d(x,w,strides=[1,1,1,1],padding='SAME')
	#自定义池化函数
	def max_pool_2x2(x):
	    return tf.nn.max_pool(x,ksize=[1,2,2,1],strides=[1,2,2,1],padding='SAME')
卷积步长为1，如果要改为步长为2则strides=[1,2,2,1],对于二维图来说只有中间两个是有效的，使用‘SAME’的padding方法（即输出与输入保持相同尺寸，边界处少一两个像素则自动补上）；池化层的设置也类似，池化尺寸为2x2

	#设置占位符，尺寸为样本输入和输出的尺寸
	x=tf.placeholder(tf.float32,[None,784])
	y_=tf.placeholder(tf.float32,[None,10])
	x_img=tf.reshape(x,[-1,28,28,1])

设置输入输出的占位符，占位符是向一个会话中喂数据的入口，因为在TensorFlow的使用中通过构建计算图来设计网络，而网络的运行计算则在会话中启动，这个过程我们无法直接介入，需要通过placeholder来对一个会话进行数据输入。
占位符设置好之后将x变形为28x28是矩阵形式（tf.reshape()函数)

	#设置第一个卷积层和池化层
	w_conv1=tf.Variable(tf.truncated_normal([3,3,1,32],stddev=0.1))
	b_conv1=tf.Variable(tf.constant(0.1,shape=[32]))
	h_conv1=tf.nn.relu(conv2d(x_img,w_conv1)+b_conv1)
	h_pool1=max_pool_2x2(h_conv1)

	#设置第二个卷积层和池化层
	w_conv2=tf.Variable(tf.truncated_normal([3,3,32,50],stddev=0.1))
	b_conv2=tf.Variable(tf.constant(0.1,shape=[50]))
	h_conv2=tf.nn.relu(conv2d(h_pool1,w_conv2)+b_conv2)
	h_pool2=max_pool_2x2(h_conv2)
	
第一层卷积使用3x3x1的卷积核，一共有32个卷积核，权值使用方差为0.1的截断正态分布（最大值不超过方差两倍的分布）来初始化，偏置的初值设定为常数0.1。
第二层卷积核第一层类似，卷积核尺寸为3x3x32（32是通道数，因为上一层使用32个卷积核，所以这一层的通道数就变成了32），这一层一共使用了50个卷积核，其他设置与上一层相同。
每一层卷积完之后接上一个2x2的最大化池化操作
	
	#设置第一个全连接层
	w_fc1=tf.Variable(tf.truncated_normal([7*7*50,1024],stddev=0.1))
	b_fc1=tf.Variable(tf.constant(0.1,shape=[1024]))
	h_pool2_flat=tf.reshape(h_pool2,[-1,7*7*50])
	h_fc1=tf.nn.relu(tf.matmul(h_pool2_flat,w_fc1)+b_fc1)
	
	#dropout(随机权重失活)
	
	keep_prob=tf.placeholder(tf.float32)
	h_fc1_drop=tf.nn.dropout(h_fc1,keep_prob)
	
	#设置第二个全连接层
	w_fc2=tf.Variable(tf.truncated_normal([1024,10]))
	b_fc2=tf.Variable(tf.constant(0.1,shape=[10]))
	y_out=tf.nn.softmax(tf.matmul(h_fc1_drop,w_fc2)+b_fc2)
	
卷积层之后就是两个全连接层，第一个全连接层有1024个神经元，现将卷积层得到的2x2输出展开成一长条，使用ReLu激活函数得到输出，输出为1024维。

Dropout：在这一层使用权值随机失活，对一些神经元突触连接进行强制的置零，这个trick可以防止神经网络过拟合。这里的dropout的保留比例是0.5，即随机的保留一半权值，删除另一半。dropout比例通过placeholder来设置，因为训练过程中需要dropout，但是在最后的测试过程中，我们有希望使用全部的权值，随意dropout的比例要能够改变，这里使用placeholder。

第二个全连接层有10个神经元，分别对应0-9这19个数字，和之前每层不同，这里使用的激活函数是softmax，softmax是以指数函数作为核函数的归一化操作，与一般归一化操作不同的是，指数函数能够放大一个分布内各个数值的差异，两级分布现象会更明显。

	#建立loss function,为交叉熵
	loss=tf.reduce_mean(-tf.reduce_sum(y_*tf.log(y_out),reduction_indices=[1]))
	#配置Adam优化器，学习速率为1e-4
	train_step=tf.train.AdamOptimizer(1e-4).minimize(loss)
	
	#建立正确率计算表达式
	correct_prediction=tf.equal(tf.argmax(y_out,1),tf.argmax(y_,1))
	accuracy=tf.reduce_mean(tf.cast(correct_prediction,tf.float32))
	
建立loss function很重要，这里使用交叉熵作为loss，交叉熵用来衡量两个分布的相似程度。两个分布越接近，则交叉熵越小。
使用Adam优化器来最小化loss，配置学习速率为1e-4，然后建立正确率的计算表达式，tf.argmax(y_,1)函数用来返回其中最大的值的下标，tf.equal用来计算两个值是否相等，tf.cast()函数用来实现数据类型转换，tf.reduce_mean()用来求平均（得到正确率）


	#训练
	tf.global_variables_initializer().run()
	for i in range(20000):
	    batch = mnist.train.next_batch(50)
	    if i%100==0:
	        train_accuracy=accuracy.eval(feed_dict={x:batch[0],y_:batch[1],keep_prob:1})
	        print("ste %d,train_accuracy=%g" %(i,train_accuracy))
	    train_step.run(feed_dict={x:batch[0],y_:batch[1],keep_prob:0.5})

对网络进行训练，首先使用`tf.global_variables_initializer().run()`初始化所有数据，从mnist训练数据集中集中一次取50个样本作为一组训练，共进行20000组训练，每100次就输出一次该组数据上的正确率，进行训练计算的方式是`train_step.run(feed_dict={x:batch[0],y_:batch[1],keep_prob:0.5})`
,通过feed_dict来对会话输送训练数据（以及其他一些想在计算过程中实时调整的参数，比如dropout比例）
这段代码中可以看到，训练时dropout的保留比例是0.5，测试时的保留比例是1.

	#训练后用测试集进行测试，输出最终结果
	print("test_accuracy= %h" %accuracy.eval(feed_dict={x:mnist.test.images,y_:mnist.test.labels,keep_prob:1}))

最后输入测试集进行测试验证
