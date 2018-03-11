##TensorFlow集群组成
TensorFlow集群就是一组任务task，每个任务就是一个服务service。每个服务由两个部分组成，第一部分是master，用于创建session，第二部分是worker，用于执行图中具体的计算。

TensorFlow一般将任务task分为两类job：一类叫参数服务器，parameter server，简称为ps，用于存储tf.Variable；一类就是普通任务，称为worker，用于执行具体的计算。

每个任务task通常运行在不同的机器上。

Master service：是一个用于**与一系列远端的分布式设备进行交互**的RPC服务。Master Service实现了Session接口, 用来**协调多个worker service**。
Worker service：一个执行TensorFlow graph计算的RPC服务。
Client：一个典型的客户端一般会构建一个TensorFlow的图并使用Session来完成与集群的交互。


###ps和worker的分工
ps：参数的存储；更新参数。
worker：计算参数梯度。

###集群和任务的创建
在每个任务中，做以下操作：
1.创建一个 tf.train.ClusterSpec描述集群中的所有任务。这个描述对于每一个任务都应该是相同的。是为了让每个任务都知道自己所在集群中还有其他那些任务，以便与其他任务进行通信。
> 
# in task 0:
cluster = tf.train.ClusterSpec({"local": ["localhost:2222", "localhost:2223"]})  

2.创建一个 tf.train.Server, 通过 tf.train.ClusterSpec的构造函数，确定本地任务的工作名称和任务index。
> server = tf.train.Server(cluster, job_name="worker", task_index=0)  

3.选择创建session使用的master
> 
 with tf.Session("grpc://localhost:2222:8080") as sess:

Tensorflow会自动的在不同的job之间（即ps与worker之间）传输数据。在某个worker task上也可以指定将其参数放在别的job task上：

	with tf.device("/job:ps/task:1"):
  		weights_2 = tf.Variable(...)
  		biases_2 = tf.Variable(...)
	with tf.device("/job:worker/task:7"):
 		 input, labels = ...
 	 	layer_1 = tf.nn.relu(tf.matmul(input, weights_1) + biases_1)
 	 	logits = tf.nn.relu(tf.matmul(layer_1, weights_2) + biases_2)
 	 	# ...
  		train_op = ...
	with tf.Session("grpc://worker7:2222") as sess:
	 	 for _ in range(10000):
  	  		sess.run(train_op)

也可以利用tensorflow自带的tf.train.replica_device_setter函数，帮我们自动把参数放在参数服务器（若有多个ps，则轮流分配到不同的ps上）。

数据量小，各节点计算能力较均衡，用同步模型。

数据量大，各机器计算性能参差不齐，用异步模式。

##并行训练方法
###模型并行
切分模型，模型的不同部分在不同的device上被计算
###数据并行
输入数据被切分，不同节点采用相同的graph，相同的模型参数。
- in-graph
一个client，一个session，同步更新
所有操作都在**同一个graph**中，只有一个节点有client，它需要将训练**数据分发**到其他节点上，其他节点只需要join等待接收数据，一旦接受到即可开始训练。缺点在于数据分发严重影响并发速度。

- between-graph（常用）
多个client，多个session，可同步或异步更新
每个节点都有各自的client和session，**各自创建一个graph**，配置复杂了点，但是**数据不用分发**，数据分片的保存在各个计算节点，各个计算节点算完后把要更新的参数告诉参数服务器，参数服务器更新参数。（可以异步更新或者同步更新）

##一个demo
https://github.com/caicloud/tensorflow-demo/blob/master/distributed/mnist_cnn.py


