当一个高尔夫球员刚开始学习打高尔夫时，他们通常会在挥杆的练习上花费大多数时间。慢慢地他们才会在基本的挥杆上通过变化发展其他的击球方式，学习低飞球、左曲球和右曲球。类似的，我们现在仍然聚焦在反向传播算法的理解上。这就是我们的“基本挥杆”——神经网络中大部分工作学习和研究的基础。本章，我会解释若干技术能够用来提升我们关于反向传播的初级的实现，最终改进网络学习的方式。

本章涉及的技术包括：更好的代价函数的选择——[交叉熵](http://neuralnetworksanddeeplearning.com/chap3.html#the_cross-entropy_cost_function) 代价函数；四中规范化方法（L1 和 L2 规范化，dropout 和训练数据的人工扩展），这会让我们的网络在训练集之外的数据上更好地泛化；更好的[权重初始化方法](http://neuralnetworksanddeeplearning.com/chap3.html#weight_initialization)；还有[帮助选择好的超参数的启发式想法](http://neuralnetworksanddeeplearning.com/chap3.html#how_to_choose_a_neural_network's_hyper-parameters)。同样我也会再给出一些简要的[其他技术介绍](http://neuralnetworksanddeeplearning.com/chap3.html#other_techniques)。这些讨论之间的独立性比较大，所有你们可以随自己的意愿挑着看。另外我还会在代码中实现这些技术，使用他们来提高在第一章中的分类问题上的性能。

当然，我们仅仅覆盖了大量已经在神经网络中研究发展出的技术的一点点内容。此处我们学习深度学习的观点是想要在一些已有的技术上入门的最佳策略其实是深入研究一小部分最重要那些的技术点。掌握了这些关键技术不仅仅对这些技术本身的理解很有用，而且会深化你对使用神经网络时会遇到哪些问题的理解。这会让你们做好在需要时快速掌握其他技术的充分准备。

# 交叉熵代价函数
---
我们大多数人觉得错了就很不爽。在开始学习弹奏钢琴不久后，我在一个听众前做了处女秀。我很紧张，开始时将八度音阶的曲段演奏得很低。我很困惑，因为不能继续演奏下去了，直到有个人指出了其中的错误。当时，我非常尴尬。不过，尽管不开心，我们却能够因为明显的犯错快速地学习到正确的东西。你应该相信下次我再演奏肯定会是正确的！相反，在我们的错误不是很好的定义的时候，学习的过程会变得更加缓慢。

理想地，我们希望和期待神经网络可以从错误中快速地学习。在实践中，这种情况经常出现么？为了回答这个问题，让我们看看一个小例子。这个例子包含一个只有一个输入的神经元：

![](http://upload-images.jianshu.io/upload_images/42741-54bcf1d440d6de82.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们会训练这个神经元来做一件非常简单的事：让输入 $$1$$ 转化为 $$0$$。当然，这很简单了，手工找到合适的权重和偏差就可以了，不需要什么学习算法。然而，看起来使用梯度下降的方式来学习权重和偏差是很有启发的。所以，我们来看看神经元如何学习。

为了让事情确定化，我会首先将权重和偏差初始化为 $$0.6$$ 和 $$0.9$$。这些就是一般的开始学习的选择，并没有任何刻意的想法。一开始的神经元的输出是 $$0.82$$，所以这离我们的目标输出 $$0.0$$ 还差得很远。点击右下角的“运行”按钮来看看神经元如何学习到让输出接近 $$0.0$$ 的。注意到，这并不是一个已经录好的动画，你的浏览器实际上是正在进行梯度的计算，然后使用梯度更新来对权重和偏差进行更新，并且展示结果。设置学习率  $$\eta=0.15$$ 进行学习一方面足够慢的让我们跟随学习的过程，另一方面也保证了学习的时间不会太久，几秒钟应该就足够了。代价函数是我们前面用到的二次函数，$$C$$。这里我也会给出准确的形式，所以不需要翻到前面查看定义了。注意，你可以通过点击 “Run” 按钮执行训练若干次。

![1](http://upload-images.jianshu.io/upload_images/42741-752fcc2176bf9b8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2](http://upload-images.jianshu.io/upload_images/42741-99457432d0a54542.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![3](http://upload-images.jianshu.io/upload_images/42741-1eedcdca670ee566.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![4](http://upload-images.jianshu.io/upload_images/42741-b2e1647ff9c88555.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 我们这里是静态的例子，在原书中，使用的动态示例，所以为了更好的效果，请参考[原书的此处动态示例](http://neuralnetworksanddeeplearning.com/chap3.html)。

正如你所见，神经元快速地学到了使得代价函数下降的权重和偏差，给出了最终的输出为 $$0.09$$。这虽然不是我们的目标输出 $$0.0$$，但是已经挺好了。假设我们现在将初始权重和偏差都设置为 $$2.0$$。此时初始输出为 $$0.98$$，这是和目标值的差距相当大的。现在看看神经元学习的过程。点击“Run” 按钮：

![1](http://upload-images.jianshu.io/upload_images/42741-95ceb66b3451dbb7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2](http://upload-images.jianshu.io/upload_images/42741-68d123092f32aed6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![3](http://upload-images.jianshu.io/upload_images/42741-bff381a28137c4b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![4](http://upload-images.jianshu.io/upload_images/42741-83d1aa344e862ab9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![5](http://upload-images.jianshu.io/upload_images/42741-d16017b9a247c13f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

虽然这个例子使用的了同样的学习率（$$\eta=0.15$$），我们可以看到刚开始的学习速度是比较缓慢的。对前 $$150$$ 左右的学习次数，权重和偏差并没有发生太大的变化。随后学习速度加快，与上一个例子中类似了，神经网络的输出也迅速接近 $$0.0$$。

> 强烈建议参考[原书的此处动态示例](http://neuralnetworksanddeeplearning.com/chap3.html)感受学习过程的差异。

这种行为看起来和人类学习行为差异很大。正如我在此节开头所说，我们通常是在犯错比较明显的时候学习的速度最快。但是我们已经看到了人工神经元在其犯错较大的情况下其实学习很有难度。而且，这种现象不仅仅是在这个小例子中出现，也会再更加一般的神经网络中出现。为何学习如此缓慢？我们能够找到缓解这种情况的方法么？

为了理解这个问题的源头，想想神经元是按照偏导数（$$\partial C/\partial w$$ 和 $$\partial C/\partial b$$）和学习率（$$\eta$$）的乘积来改变权重和偏差的。所以，我们在说“学习缓慢”时，实际上就是说这些偏导数很小。理解他们为何这么小就是我们面临的挑战。为了理解这些，让我们计算偏导数看看。我们一直在用的是二次代价函数，定义如下

![](http://upload-images.jianshu.io/upload_images/42741-689311744b65e8ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中 $$a$$ 是神经元的输出，其中训练输入为 $$x=1$$，$$y=0$$ 则是目标输出。显式地使用权重和偏差来表达这个，我们有 $$a=\sigma(z)$$，其中 $$z=wx+b$$。使用链式法则来求偏导数就有：

![](http://upload-images.jianshu.io/upload_images/42741-fa97259e24bc11d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中我已经将 $$x=1$$ 和 $$y=0$$ 代入了。为了理解这些表达式的行为，让我们仔细看 $$\sigma'(z)$$ 这一项。首先回忆一下 $$\sigma$$ 函数图像：

![](http://upload-images.jianshu.io/upload_images/42741-8ff397e1f73e4090.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以从这幅图看出，当神经元的输出接近 $$1$$ 的时候，曲线变得相当平，所以 $$\sigma'(z)$$ 就很小了。方程 (55) 和 (56) 也告诉我们 $$\partial C/\partial w$$ 和 $$\partial C/\partial b$$ 会非常小。这其实就是学习缓慢的原因所在。而且，我们后面也会提到，这种学习速度下降的原因实际上也是更加一般的神经网络学习缓慢的原因，并不仅仅是在这个特例中特有的。

## 引入交叉熵代价函数
那么我们如何解决这个问题呢？研究表明，我们可以通过使用交叉熵代价函数来替换二次代价函数。为了理解什么是交叉熵，我们稍微改变一下之前的简单例子。假设，我们现在要训练一个包含若干输入变量的的神经元，$$x_1,x_2,...$$ 对应的权重为 $$w_1,w_2,...$$ 和偏差，$$b$$：

![](http://upload-images.jianshu.io/upload_images/42741-9f1c9f7efd4815ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

神经元的输出就是 $$a=\sigma(z)$$，其中 $$z=\sum_jw_jx_j+b$$ 是输入的带权和。我们如下定义交叉熵代价函数：

![](http://upload-images.jianshu.io/upload_images/42741-f3363ceff9e87fb4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中 $$n$$ 是训练数据的总数，对所有的训练数据 $$x$$ 和 对应的目标输出 $$y$$ 进行求和。

公式(57)是否解决学习缓慢的问题并不明显。实际上，甚至将这个定义看做是代价函数也不是显而易见的！在解决学习缓慢前，我们来看看交叉熵为何能够解释成一个代价函数。

将交叉熵看做是代价函数有两点原因。第一，它使非负的，$$C>0$$。可以看出：(a) 公式(57)的和式中所有独立的项都是非负的，因为对数函数的定义域是 $$(0,1)$$；(b) 前面有一个负号。

第二，如果神经元实际的输出接近目标值。假设在这个例子中，$$y=0$$ 而 $$a\approx 0$$。这是我们想到得到的结果。我们看到公式(57)中第一个项就消去了，因为 $$y=0$$，而第二项实际上就是 $$-ln(1-a)\approx 0$$。反之，$$y=1$$ 而 $$a\approx 1$$。所以在实际输出和目标输出之间的差距越小，最终的交叉熵的值就越低了。

综上所述，交叉熵是非负的，在神经元达到很好的正确率的时候会接近 $$0$$。这些其实就是我们想要的代价函数的特性。其实这些特性也是二次代价函数具备的。所以，交叉熵就是很好的选择了。但是交叉熵代价函数有一个比二次代价函数更好的特性就是它避免了学习速度下降的问题。为了弄清楚这个情况，我们来算算交叉熵函数关于权重的偏导数。我们将 $$a=\sigma(z)$$ 代入到公式 (57) 中应用两次链式法则，得到：

![](http://upload-images.jianshu.io/upload_images/42741-f5e7c6fea8b38d47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

将结果合并一下，简化成：

![](http://upload-images.jianshu.io/upload_images/42741-38fcdc9659b6b608.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

根据 $$\sigma(z) = 1/(1+e^{-z})$$ 的定义，和一些运算，我们可以得到 $$\sigma'(z) = \sigma(z)(1-\sigma(z))$$。后面在练习中会要求你计算这个，现在可以直接使用这个结果。我们看到 $$\sigma'$$ 和 $$\sigma(z)(1-\sigma(z))$$ 这两项在方程中直接约去了，所以最终形式就是：

![](http://upload-images.jianshu.io/upload_images/42741-5575d9699b92480b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这是一个优美的公式。它告诉我们权重学习的速度受到 $$\sigma(z)-y$$，也就是输出中的误差的控制。更大的误差，更快的学习速度。这是我们直觉上期待的结果。特别地，这个代价函数还避免了像在二次代价函数中类似公式中 $$\sigma'(z)$$ 导致的学习缓慢，见公式(55)。当我们使用交叉熵的时候，$$\sigma'(z)$$ 被约掉了，所以我们不再需要关心它是不是变得很小。这种约除就是交叉熵带来的特效。实际上，这也并不是非常奇迹的事情。我们在后面可以看到，交叉熵其实只是满足这种特性的一种选择罢了。

根据类似的方法，我们可以计算出关于偏差的偏导数。我这里不再给出详细的过程，你可以轻易验证得到

![](http://upload-images.jianshu.io/upload_images/42741-05567745404c5dcd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 练习
* 验证 \sigma'(z) = \sigma(z)(1-\sigma(z))。

让我们重回最原初的例子，来看看换成了交叉熵之后的学习过程。现在仍然按照前面的参数配置来初始化网络，开始权重为 $$0.6$$，而偏差为 $$0.9$$。点击“Run”按钮看看在换成交叉熵之后网络的学习情况：

![1](http://upload-images.jianshu.io/upload_images/42741-236a80d9625b7533.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2](http://upload-images.jianshu.io/upload_images/42741-d94d7466e960b38c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![3](http://upload-images.jianshu.io/upload_images/42741-d8971200e55bc88b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![4](http://upload-images.jianshu.io/upload_images/42741-cfa4b9cf7de3b2c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![5](http://upload-images.jianshu.io/upload_images/42741-9aadf7001094c81d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

毫不奇怪，在这个例子中，神经元学习得相当出色，跟之前差不多。现在我们再看看之前出问题的那个例子，权重和偏差都初始化为 $$2.0$$：

![1](http://upload-images.jianshu.io/upload_images/42741-461c7ebcebdc9551.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2](http://upload-images.jianshu.io/upload_images/42741-bd5ac4ae285ff597.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![3](http://upload-images.jianshu.io/upload_images/42741-24b6e22490e6469e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![4](http://upload-images.jianshu.io/upload_images/42741-4fedc8ddbdb3a5cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![5](http://upload-images.jianshu.io/upload_images/42741-0860b6265dfa1305.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

成功了！这次神经元的学习速度相当快，跟我们预期的那样。如果你观测的足够仔细，你可以发现代价函数曲线要比二次代价函数训练前面部分要陡很多。正是交叉熵带来的快速下降的坡度让神经元在处于误差很大的情况下能够逃脱出学习缓慢的困境，这才是我们直觉上所期待的效果。

我们还没有提及关于学习率的讨论。刚开始使用二次代价函数的时候，我们使用了 $$\eta=0.15$$。在新例子中，我们还应该使用同样的学习率么？实际上，根据不同的代价函数，我们不能够直接去套用同样的学习率。这好比苹果和橙子的比较。对于这两种代价函数，我只是通过简单的实验来找到一个能够让我们看清楚变化过程的学习率的值。尽管我不愿意提及，但如果你仍然好奇，这个例子中我使用了 $$\eta=0.005$$。

你可能会反对说，上面学习率的改变使得上面的图失去了意义。谁会在意当学习率的选择是任意挑选的时候神经元学习的速度？！这样的反对其实没有抓住重点。上面的图例不是想讨论学习的绝对速度。而是想研究学习速度的变化情况。特别地，当我们使用二次代价函数时，学习在神经元犯了明显的错误的时候却比学习快接近真实值的时候缓慢；而使用交叉熵学习正是在神经元犯了明显错误的时候速度更快。这些现象并不是因为学习率的改变造成的。

我们已经研究了一个神经元的交叉熵。不过，将其推广到多个神经元的多层神经网络上也是很简单的。特别地，假设 $$y=y_1,y_2,...$$ 是输出神经元上的目标值，而 $$a_1^L,a_2^L,...$$是实际输出值。那么我们定义交叉熵如下

![](http://upload-images.jianshu.io/upload_images/42741-80db42bf50ee6cfa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

除了这里需要对所有输出神经元进行求和外，这个其实和我们早前的公式(57)一样的。这里不会给出一个推算的过程，但需要说明的时使用公式(63)确实会在很多的神经网络中避免学习的缓慢。如果你感兴趣，你可以尝试一下下面问题的推导。

那么我们应该在什么时候用交叉熵来替换二次代价函数？实际上，如果在输出神经元使用 sigmoid 激活函数时，交叉熵一般都是更好的选择。为什么？考虑一下我们初始化网络的时候通常使用某种随机方法。可能会发生这样的情况，这些初始选择会对某些训练输入误差相当明显——比如说，目标输出是 $$1$$，而实际值是 $$0$$，或者完全反过来。如果我们使用二次代价函数，那么这就会导致学习速度的下降。它并不会完全终止学习的过程，因为这些权重会持续从其他的样本中进行学习，但是显然这不是我们想要的效果。

## 练习
* 一个小问题就是刚接触交叉熵时，很难一下子记住那些表达式对应的角色。又比如说，表达式的正确形式是 $$−[ylna+(1−y)ln(1−a)]$$
 或者 $$−[alny+(1−a)ln(1−y)]$$。在 $$y=0$$ 或者 $$1$$ 的时候第二个表达式的结果怎样？这个问题会影响第一个表达式么？为什么？
* 在对单个神经元讨论中，我们指出如果对所有的训练数据有 $$\sigma(z)\approx y$$，交叉熵会很小。这个论断其实是和 $$y$$ 只是等于 $$1$$ 或者 $$0$$ 有关。这在分类问题一般是可行的，但是对其他的问题（如回归问题）$$y$$ 可以取 $$0$$  和 $$1$$ 之间的中间值的。证明，交叉熵在 $$\sigma(z)=y$$ 时仍然是最小化的。此时交叉熵的表示是：

![](http://upload-images.jianshu.io/upload_images/42741-5c38ff3dd38581ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

而其中 $$−[y\ln y+(1−y)\ln(1−y)]$$ 有时候被称为 [二元熵](http://en.wikipedia.org/wiki/Binary_entropy_function)

## 问题
* **多层多神经元网络** 用上一章的定义符号，证明对二次代价函数，关于输出层的权重的偏导数为

![](http://upload-images.jianshu.io/upload_images/42741-312d7eff0fb92419.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

项 $$\sigma'(z_j^L)$$ 会在一个输出神经元困在错误值时导致学习速度的下降。证明对于交叉熵代价函数，针对一个训练样本 $$x$$ 的输出误差 $$\delta^L$$为

![](http://upload-images.jianshu.io/upload_images/42741-6cc31b8740f5b9c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用这个表达式来证明关于输出层的权重的偏导数为

![](http://upload-images.jianshu.io/upload_images/42741-adf20b59d623ed79.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里 $$\sigma'(z_j^L)$$ 就消失了，所以交叉熵避免了学习缓慢的问题，不仅仅是在一个神经元上，而且在多层多元网络上都起作用。这个分析过程稍作变化对偏差也是一样的。如果你对这个还不确定，那么请仔细研读一下前面的分析。

* **在输出层使用线性神经元时使用二次代价函数** 假设我们有一个多层多神经元网络，最终输出层的神经元都是线性的，输出不再是 sigmoid 函数作用的结果，而是 $$a_j^L = z_j^L$$。证明如果我们使用二次代价函数，那么对单个训练样本 $$x$$ 的输出误差就是

![](http://upload-images.jianshu.io/upload_images/42741-ab87835060e9894f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

类似于前一个问题，使用这个表达式来证明关于输出层的权重和偏差的偏导数为

![](http://upload-images.jianshu.io/upload_images/42741-29173e61d12177c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这表明如果输出神经元是线性的那么二次代价函数不再会导致学习速度下降的问题。在此情形下，二次代价函数就是一种合适的选择。

## 使用交叉熵来对 MNIST 数字进行分类
交叉熵很容易作为使用梯度下降和反向传播方式进行模型学习的一部分来实现。我们将会在下一章进行对前面的程序 `network.py` 的改进。新的程序写在 `network2.py` 中，不仅使用了交叉熵，还有本章中介绍的其他的技术。现在我们看看新的程序在进行 MNIST 数字分类问题上的表现。如在第一章中那样，我们会使用一个包含 $$30$$ 个隐藏元的网络，而 minibatch 的大小也设置为 $$10$$。我们将学习率设置为 $$\eta=0.5$$ 然后训练 30 回合。`network2.py` 的接口和 `network.py` 稍微不同，但是过程还是很清楚的。你可以使用如 `help(network2.Network.SGD)` 这样的命令来检查对应的文档。
```python
>>> import mnist_loader
>>> training_data, validation_data, test_data = \
... mnist_loader.load_data_wrapper()
>>> import network2
>>> net = network2.Network([784, 30, 10], cost=network2.CrossEntropyCost)
>>> net.large_weight_initializer()
>>> net.SGD(training_data, 30, 10, 0.5, evaluation_data=test_data,
... monitor_evaluation_accuracy=True)
```

注意看下，`net.large_weight_initializer()` 命令使用和第一章介绍的同样的方式来进行权重和偏差的初始化。运行上面的代码我们得到了一个 95.49% 准确度的网络。这跟我们在第一章中使用二次代价函数得到的结果相当接近了，95.42%。

同样也来试试使用 $$100$$ 个隐藏元的交叉熵及其他参数保持不变的情况。在这个情形下，我们获得了 96.82% 的准确度。相比第一章使用二次代价函数的结果 96.59%，这有一定的提升。这看起来是一个小小的变化，但是考虑到误差率已经从 3.41% 下降到 3.18%了。我们已经消除了原误差的 $$1/14$$。这其实是可观的改进。

令人振奋的是交叉熵代价函数给了我们类似的或者更好的结果。然而，这些结果并没有能够确定性地证明交叉熵是更好的选择。原因在于我已经在选择诸如学习率，minibatch 大小等等这样的超参数上做出了一些努力。为了让这些提升更具说服力，我们需要进行对超参数进行深度的优化。当然，这些结果都挺好的，强化了我们早先关于交叉熵优于二次代价的理论论断。

这只是本章后面的内容和整本书剩余内容中的更为一般的模式的一部分。我们将给出一个新的技术，然后进行尝试，随后获得“提升的”结果。当然，看到这些进步是很好的。但是这些提升的解释一般来说都困难重重。在进行了大量工作来优化所有其他的超参数，使得提升真的具有说服力。工作量很大，需要大量的计算能力，我们也不会进行这样耗费的调查。相反，我采用上面进行的那些不正式的测试来达成目标。所以你要记住这样的测试缺少确定性的证明，需要注意那些使得论断失效的信号。

至此，我们已经花了大量篇幅介绍交叉熵。为何对一个只能给出一点点性能提升的技术上花费这么多的精力？后面，我们会看到其他的技术——规范化，会带来更大的提升效果。所以为何要这么细致地讨论交叉熵？部分原因在于交叉熵是一种广泛使用的代价函数，值得深入理解。但是更加重要的原因是神经元的饱和是神经网络中一个关键的问题，整本书都会不断回归到这个问题上。因此我现在深入讨论交叉熵就是因为这是一种开始理解神经元饱和和如何解决这个问题的很好的实验。

## 交叉熵的含义？源自哪里？
我们对于交叉熵的讨论聚焦在代数分析和代码实现。这虽然很有用，但是也留下了一个未能回答的更加宽泛的概念上的问题，如：交叉熵究竟表示什么？存在一些直觉上的思考交叉熵的方法么？我们如何想到这个概念？

让我们从最后一个问题开始回答：什么能够激发我们想到交叉熵？假设我们发现学习速度下降了，并理解其原因是因为在公式(55)(56)中的 $$\sigma'(z)$$ 那一项。在研究了这些公式后，我们可能就会想到选择一个不包含 $$\sigma'(z)$$ 的代价函数。所以，这时候对一个训练样本 $$x$$，代价函数是 $$C=C_x$$ 就满足：

![](http://upload-images.jianshu.io/upload_images/42741-373b9b6f27c9c74f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果我们选择的代价函数满足这些条件，那么同样他们会拥有遇到明显误差时的学习速度越快这样的特性。这也能够解决学习速度下降的问题。实际上，从这些公式开始，现在就给你们看看若是跟随自身的数学嗅觉推导出交叉熵的形式是可能的。我们来推一下，由链式法则，我们有

![](http://upload-images.jianshu.io/upload_images/42741-5f58cf963a71bce6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用 $$\sigma'(z)=\sigma(z)(1-\sigma(z))=a(1-a)$$，上个等式就变成

![](http://upload-images.jianshu.io/upload_images/42741-2564f711fb4365d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对比等式(72)，我们有

![](http://upload-images.jianshu.io/upload_images/42741-1417ab58f98e01bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对此方程关于 $$a$$ 进行积分，得到 

![](http://upload-images.jianshu.io/upload_images/42741-9e1525eab19cfa01.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中 $$constant$$ 是积分常量。这是一个单独的训练样本 $$x$$ 对代价函数的贡献。为了得到整个的代价函数，我们需要对所有的训练样本进行平均，得到了

![](http://upload-images.jianshu.io/upload_images/42741-5f594db7abf56be1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

而这里的常量就是所有单独的常量的平均。所以我们看到方程(71)和(72)唯一确定了交叉熵的形式，并加上了一个常量的项。这个交叉熵并不是凭空产生的。而是一种我们以自然和简单的方法获得的结果。

那么交叉熵直觉含义又是什么？我们如何看待它？深入解释这一点会将我们带到一个不大愿意讨论的领域。然而，还是值得提一下，有一种源自信息论的解释交叉熵的标准方式。粗略地说，交叉熵是惊奇的度量（measure of surprise）。特别地，我们的神经元想要要计算函数 $$x\rightarrow y=y(x)$$。但是，它用函数$$x\rightarrow a=a(x)$$ 进行了替换。假设我们将 $$a$$ 想象成我们神经元估计 $$y = 1$$ 概率，而 $$1-a$$ 则是 $$y=0$$ 的概率。如果输出我们期望的结果，惊奇就会小一点；反之，惊奇就大一些。当然，我这里没有严格地给出“惊奇”到底意味着什么，所以看起来像在夸夸奇谈。但是实际上，在信息论中有一种准确的方式来定义惊奇究竟是什么。不过，我也不清楚在网络上，哪里有好的，短小的自包含对这个内容的讨论。但如果你要深入了解的话，Wikipedia 包含一个[简短的总结](http://en.wikipedia.org/wiki/Cross_entropy#Motivation)，这会指引你正确地探索这个领域。而更加细节的内容，你们可以阅读[Cover and Thomas](http://books.google.ca/books?id=VWq5GG6ycxMC)的第五章涉及 Kraft 不等式的有关信息论的内容。

## 问题
* 我们已经深入讨论了使用二次代价函数的网络中在输出神经元饱和时候学习缓慢的问题，另一个可能会影响学习的因素就是在方程(61)中的 $$x_j$$ 项。由于此项，当输入 $$x_j$$ 接近 $$0$$ 时，对应的权重 $$x_j$$ 会学习得相当缓慢。解释为何不可以通过改变代价函数来消除 $$x_j$$ 项的影响。

## Softmax
本章，我们大多数情况会使用交叉熵来解决学习缓慢的问题。但是，我希望简要介绍一下基于 softmax 神经元层的解决这个问题的另一种观点。我们不会实际在剩下的章节中使用 softmax 层，所以你如果赶时间，就可以跳到下一个小节了。不过，softmax 仍然有其重要价值，一方面它本身很有趣，另一方面，因为我们会在第六章在对深度神经网络的讨论中使用 softmax 层。

softmax 的想法其实就是为神经网络定义一种新式的输出层。开始时和 sigmoid 层一样的，首先计算带权输入 $$z_j^L = \sum_k w_{jk}^La_k^{L-1}+b_j^L$$。不过，这里我们不会使用 sigmoid 函数来获得输出。而是，会应用一种叫做 softmax 函数在 $$z_j^L$$ 上。根据这个函数，激活值 $$a_j^L$$ 就是

![](http://upload-images.jianshu.io/upload_images/42741-b84180005a015ffc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中，分母是对所有的输出神经元进行求和。

如果你不习惯这个函数，方程(78)可能看起来会比较难理解。因为对于使用这个函数的原因你不清楚。为了更好地理解这个公式，假设我们有一个包含四个输出神经元的神经网络，对应四个带权输入，表示为 $$z_1^L, z_2^L, z_3^L, z_4^L$$。下面的例子可以进行对应权值的调整，并给出对应的激活值的图像。可以通过固定其他值，来看看改变 $$z_4^L$$ 的值会产生什么样的影响：

![1](http://upload-images.jianshu.io/upload_images/42741-d21cb21704e16879.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![2](http://upload-images.jianshu.io/upload_images/42741-b2b1987aa0501ef6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![3](http://upload-images.jianshu.io/upload_images/42741-9ba8de22bc879ab7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 这里给出了三个选择的图像，建议去原网站体验

在我们增加 $$z_4^L$$ 的时候，你可以看到对应激活值 $$a_4^L$$ 的增加，而其他的激活值就在下降。类似地，如果你降低 $$z_4^L$$ 那么 $$a_4^L$$ 就随之下降。实际上，如果你仔细看，你会发现在两种情形下，其他激活值的整个改变恰好填补了 $$a_4^L$$ 的变化的空白。原因很简单，根据定义，输出的激活值加起来正好为 $$1$$，使用公式(78)我们可以证明：

![](http://upload-images.jianshu.io/upload_images/42741-56656f179bd9308a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以，如果 $$a_4^L$$ 增加，那么其他输出激活值肯定会总共下降相同的量，来保证和式为 $$1$$。当然，类似的结论对其他的激活函数也需要满足。

方程(78)同样保证输出激活值都是正数，因为指数函数是正的。将这两点结合起来，我们看到 softmax 层的输出是一些相加为 $$1$$ 正数的集合。换言之，softmax 层的输出可以被看做是一个概率分布。

这样的效果很令人满意。在很多问题中，将这些激活值作为网络对于某个输出正确的概率的估计非常方便。所以，比如在 MNIST 分类问题中，我们可以将 $$a_j^L$$ 解释成网络估计正确数字分类为 $$j$$ 的概率。

对比一下，如果输出层是 sigmoid 层，那么我们肯定不能假设激活值形成了一个概率分布。我不会证明这一点，但是源自 sigmoid 层的激活值是不能够形成一种概率分布的一般形式的。所以使用 sigmoid 输出层，我们没有关于输出的激活值的简单的解释。

## 练习
* 构造例子表明在使用 sigmoid 输出层的网络中输出激活值 $$a_j^L$$ 的和并不会确保为 $$1$$。

我们现在开始体会到 softmax 函数的形式和行为特征了。来回顾一下：在公式(78)中的指数函数确保了所有的输出激活值是正数。然后分母的求和又保证了 softmax 的输出和为 $$1$$。所以这个特定的形式不再像之前那样难以理解了：反而是一种确保输出激活值形成一个概率分布的自然的方式。你可以将其想象成一种重新调节 $$z_j^L$$   的方法，然后将这个结果整合起来构成一个概率分布。

## 练习
* **softmax 的单调性** 证明如果 $$j=k$$ 则 $$\partial a_j^L/\partial z_k^L$$ 为正，否则为负。结果是，增加 $$z_^L$$ 会提高对应的输出激活值 $$a_k^L$$ 并降低其他所有输出激活值。我们已经在滑条示例中实验性地看到了这一点，这里需要你给出一个严格证明。
* **softmax 的非局部性** sigmoid 层的一个好处是输出 $$a_j^L$$ 是对应带权输入 $$a_j^L = \sigma(z_j^L)$$ 的函数。解释为何对于 softmax 来说，并不是这样的情况：仍和特定的输出激活值 $$a_j^L$$ 依赖所有的带权输入。

## 问题
* **逆转 softmax 层** 假设我们有一个使用 softmax 输出层的神经网络，然后激活值 $$a_j^L$$ 已知。证明对应带权输入的形式为 $$z_j^L = \ln a_j^L + C$$，其中 $$C$$ 是独立于 $$j$$ 的。

**学习缓慢问题**：我们现在已经对 softmax 神经元有了一定的认识。但是我们还没有看到 softmax 会怎么样解决学习缓慢问题。为了理解这点，先定义一个 log-likelihood 代价函数。我们使用 $$x$$ 表示训练输入，$$y$$ 表示对应的目标输出。然后关联这个训练输入样本的 log-likelihood 代价函数就是

![](http://upload-images.jianshu.io/upload_images/42741-c6a82eff60626d3c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以，如果我们训练的是 MNIST 图像，输入为 $$7$$ 的图像，那么对应的 log-likelihood 代价就是 $$-\ln a_7^L$$。看看这个直觉上的含义，想想当网络表现很好的时候，也就是确认输入为 $$7$$ 的时候。这时，他会估计一个对应的概率 $$a_7^L$$ 跟 $$1$$ 非常接近，所以代价 $$-\ln a_7^L$$ 就会很小。反之，如果网络的表现糟糕时，概率 $$a_7^L$$ 就变得很小，代价 $$-\ln a_7^L$$ 随之增大。所以 log-likelihood 代价函数也是满足我们期待的代价函数的条件的。

那关于学习缓慢问题呢？为了分析它，回想一下学习缓慢的关键就是量 $$\partial C/\partial w_{jk}^L$$ 和 $$\partial C/\partial b_j^L$$ 的变化情况。我不会显式地给出详细的推导——请你们自己去完成这个过程——但是会给出一些关键的步骤：


![](http://upload-images.jianshu.io/upload_images/42741-2ca3c9d2f0c4d8a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 请注意这里的表示上的差异，这里的 $$y$$ 和之前的目标输出值不同，是离散的向量表示，对应位的值为 $$1$$，而其他为 $$0$$。

这些方程其实和我们前面对交叉熵得到的类似。就拿方程(82) 和 (67) 比较。尽管后者我对整个训练样本进行了平均，不过形式还是一致的。而且，正如前面的分析，这些表达式确保我们不会遇到学习缓慢的问题。实际上，将 softmax 输出层和 log-likelihood 组合对照 sigmoid 输出层和交叉熵的组合类比着看是非常有用的。

有了这样的相似性，你会使用哪一种呢？实际上，在很多应用场景中，这两种方式的效果都不错。本章剩下的内容，我们会使用 sigmoid 输出层和交叉熵的组合。后面，在第六章中，我们有时候会使用 softmax 输出层和 log-likelihood 的则和。切换的原因就是为了让我们的网络和某些在具有影响力的学术论文中的形式更为相似。作为一种更加通用的视角，softmax 加上 log-likelihood 的组合更加适用于那些需要将输出激活值解释为概率的场景。当然这不总是合理的，但是在诸如 MNIST 这种有着不重叠的分类问题上确实很有用。

## 问题
* 推导方程(81) 和 (82)
* **softmax 这个名称从何处来？** 假设我们改变一下 softmax 函数，使得输出激活值定义如下

![](http://upload-images.jianshu.io/upload_images/42741-5b3e8cf9fceece21.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中 $$c$$ 是正的常量。注意 $$c=1$$ 对应标准的 softmax 函数。但是如果我们使用不同的 $$c$$ 得到不同的函数，其实最终的量的结果却和原来的 softmax 差不多。特别地，证明输出激活值也会形成一个概率分布。假设我们允许 $$c$$ 足够大，比如说 $$c\rightarrow \infty$$。那么输出激活值 $$a_j^L$$ 的极限值是什么？在解决了这个问题后，你应该能够理解 $$c=1$$ 对应的函数是一个最大化函数的 softened 版本。这就是 softmax 的来源。
> 这让我联想到 EM 算法，对 k-Means 算法的一种推广。

* **softmax 和 log-likelihood 的反向传播** 上一章，我们推到了使用 sigmoid 层的反向传播算法。为了应用在 softmax 层的网络上，我们需要搞清楚最后一层上误差的表示 $$\delta_j^L \equiv \partial C/\partial z_j^L$$。证明形式如下：

![](http://upload-images.jianshu.io/upload_images/42741-73905c0ac973a00d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用这个表达式，我们可以应用反向传播在采用了 softmax 输出层和 log-likelihood 的网络上。
