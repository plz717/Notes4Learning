# AlexNet

1. 8层
2. 大量数据:ImageNet；
3. GPU的并行计算能力使得模型可以变复杂；
4. 算法的改进，包括网络变深、数据增强(水平翻转  随机裁剪、平移变换   颜色、光照变换)、ReLU、Dropout , Local Response Normalization等。

-  Local Response Normalization

  加入这个机制是为了仿造生物学上**活跃的神经元对相邻神经元的抑制现象**（侧抑制），使得其中响应比较大的值变得相对更大，并抑制其他反馈较小的神经元, 提高网络的泛化能力。

  在NN里“神经元”对应的是“feature map”，因此LRN实现的就是**pixel-wise**、**跨feature map的归一化**。

  由下图可以看出, 每个神经元的新的响应值都与其周围(跨feature map)的所有神经元有关

  !img](http://leix.me/images/lrn.png)

  ![img](https://qph.ec.quoracdn.net/main-qimg-c1dade99f98a8d843e235ca836b7b51e.webp)

- Overlapping Pooling

  pooling size大于stride

  相比传统的Polling, 减少信息丢失。

![lrn](http://leix.me/images/lrn.png)



# VGG

一个字:深;  两个字:更深

![VGG-19](https://www.52ml.net/wp-content/uploads/2016/08/vgg19.png)



#GoogLeNet

没有最深，只有更深。

创新:

Inception-----Network In Network, 即原来的结点也是一个网络



#ResNet

没有最深，只有更深（152层）。目前层数已突破一千。

创新:

残差网络, 解决了层次比较深的时候无法训练的问题

> 更深的网络, 意味着更强的特征拟合能力, 但是实际上更深的网络会带来:
>
> 1.梯度爆炸或消失     
>
> ​	这个可以通过batch normalization解决
>
> 2.层数很深时, 在训练集上的误差反而更大了, 即出现了退化现象
>
> ​	通过"浅层网络  +  等同映射"构造深层模型，发现深层模型并没有比浅层网络有"等同或更低的错误率"
>
> ​	推断:  退化问题可能是因为----->很难利用多层网络拟合同等函数

###如何解决退化问题?

shortcut使得**输入可以直达输出**(避免过拟合)，而优化的目标由原来的    **拟合**  输出H(x) = x
变成   **输出和输入的差H(x)-x**，其中H(X)是某一层原始的的期望映射输出，x是输入, 只要H(x) - x = 0, 即F(x) = 0 , 就能拟合恒等映射了

![residual](https://www.52ml.net/wp-content/uploads/2016/08/residual.png)

### 为什么残差学习更容易?

-  原来想学习恒等映射:

  ​	输入x, 输出F(x), 为了使 输出等于输入, 需要找到合适的权重使得  F(x) = x

-  现在的恒等映射:

  ​	输入x, 输出F(x)+x, 为了使 输出等于输入, 需要找到合适的权重使得   F(x) = 0 

找到使函数等于零的参数 显然比找到使函数等于一个常数的参数更容易

首先残差单元可以表示为：

![\begin{align} & {{y}_{l}}=h({{x}_{l}})+F({{x}_{l}},{{W}_{l}}) \\ & {{x}_{l+1}}=f({{y}_{l}}) \\ \end{align} ](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D+%26+%7B%7By%7D_%7Bl%7D%7D%3Dh%28%7B%7Bx%7D_%7Bl%7D%7D%29%2BF%28%7B%7Bx%7D_%7Bl%7D%7D%2C%7B%7BW%7D_%7Bl%7D%7D%29+%5C%5C+%26+%7B%7Bx%7D_%7Bl%2B1%7D%7D%3Df%28%7B%7By%7D_%7Bl%7D%7D%29+%5C%5C+%5Cend%7Balign%7D+) 

其中 ![x_{l}](https://www.zhihu.com/equation?tex=x_%7Bl%7D) 和 ![x_{l+1}](https://www.zhihu.com/equation?tex=x_%7Bl%2B1%7D) 分别表示的是第 ![l](https://www.zhihu.com/equation?tex=l) 个残差单元的输入和输出，注意每个残差单元一般包含多层结构。 ![F](https://www.zhihu.com/equation?tex=F) 是残差函数，表示学习到的残差，而 ![h(x_{l})=x_{l}](https://www.zhihu.com/equation?tex=h%28x_%7Bl%7D%29%3Dx_%7Bl%7D) 表示恒等映射， ![f](https://www.zhihu.com/equation?tex=f) 是ReLU激活函数。基于上式，我们求得从浅层 ![l](https://www.zhihu.com/equation?tex=l) 到深层 ![L](https://www.zhihu.com/equation?tex=L) 的学习特征为：

![{{x}_{L}}={{x}_{l}}+\sum\limits_{i-l}^{L-1}{F({{x}_{i}}},{{W}_{i}})](https://www.zhihu.com/equation?tex=%7B%7Bx%7D_%7BL%7D%7D%3D%7B%7Bx%7D_%7Bl%7D%7D%2B%5Csum%5Climits_%7Bi-l%7D%5E%7BL-1%7D%7BF%28%7B%7Bx%7D_%7Bi%7D%7D%7D%2C%7B%7BW%7D_%7Bi%7D%7D%29) 

利用链式规则，可以求得反向过程的梯度：

![\frac{\partial loss}{\partial {{x}_{l}}}=\frac{\partial loss}{\partial {{x}_{L}}}\cdot \frac{\partial {{x}_{L}}}{\partial {{x}_{l}}}=\frac{\partial loss}{\partial {{x}_{L}}}\cdot \left( 1+\frac{\partial }{\partial {{x}_{L}}}\sum\limits_{i=l}^{L-1}{F({{x}_{i}},{{W}_{i}})} \right)](https://www.zhihu.com/equation?tex=%5Cfrac%7B%5Cpartial+loss%7D%7B%5Cpartial+%7B%7Bx%7D_%7Bl%7D%7D%7D%3D%5Cfrac%7B%5Cpartial+loss%7D%7B%5Cpartial+%7B%7Bx%7D_%7BL%7D%7D%7D%5Ccdot+%5Cfrac%7B%5Cpartial+%7B%7Bx%7D_%7BL%7D%7D%7D%7B%5Cpartial+%7B%7Bx%7D_%7Bl%7D%7D%7D%3D%5Cfrac%7B%5Cpartial+loss%7D%7B%5Cpartial+%7B%7Bx%7D_%7BL%7D%7D%7D%5Ccdot+%5Cleft%28+1%2B%5Cfrac%7B%5Cpartial+%7D%7B%5Cpartial+%7B%7Bx%7D_%7BL%7D%7D%7D%5Csum%5Climits_%7Bi%3Dl%7D%5E%7BL-1%7D%7BF%28%7B%7Bx%7D_%7Bi%7D%7D%2C%7B%7BW%7D_%7Bi%7D%7D%29%7D+%5Cright%29) 

式子的第一个因子 ![\frac{\partial loss}{\partial {{x}_{L}}}](https://www.zhihu.com/equation?tex=%5Cfrac%7B%5Cpartial+loss%7D%7B%5Cpartial+%7B%7Bx%7D_%7BL%7D%7D%7D) 表示的损失函数到达 ![L](https://www.zhihu.com/equation?tex=L) 的梯度，小括号中的1表明短路机制可以无损地传播梯度，而另外一项残差梯度则需要经过带有weights的层，梯度不是直接传递过来的。残差梯度不会那么巧全为-1，而且就算其比较小，有1的存在也不会导致梯度消失。所以残差学习会更容易。