【论文笔记】使用多流密集网络的密度感知单图像去雨
===================================

## 0 摘要

提出一种密度感知多路稠密连接卷积神经网络算法，DID-MDN。

这种算法可以使网络自动地判断雨密度，然后并根据所得雨量密度估计标签去除雨线。

为了更好的描述雨线的尺寸和形状，一种多路密集连接的去雨网络能够利用不同尺寸雨线的特征。

此外，创建了一个新的带有雨量密度的新数据集，用来训练密度感知网络。

在合成的和真实的数据集上的大量实验证明这种方法达到了有效的提升。另外，进行消融研究以证明在所提出的方法中由不同模块获得的改进。


## 1 介绍
现有单图像去雨方法的局限：

现有方法仅能处理特定类型的雨图，不能有效考虑雨滴的形状，范围和密度 => 导致过度去雨和去雨不足

解决方案：1.扩充数据集 2.密度感知

提出DID-MDN网络，自动的判断雨量密度信息（大、中、小），并将其运用到去雨中。

DID-MDN包括两步：雨密度分类和雨线去除。

（1）为了准确估计雨密度级别，一种新的残差网络利用残差模块在有雨的做分类。
（2）雨线去除算法基于多流密集连接网络，它考虑了雨线的尺寸和形状信息。

一旦雨密度级别估计出来，融合已估计的雨密度信息到多流密集连接网络，得到最终的除雨的输出结果。

此外，为了高效的训练这种网络，合成了一个大规模的数据集，包含12000张图片，有不同雨密度级别。

此论文有如下贡献：

1. DID-MDN 自动的判断雨量密度信息，然后通过估计出来的雨量密度标签去除雨线。
2. 基于残差可以作为更好的描述雨密度信息的特征，论文提出残差感知分类器来判断雨量密度。
3. 合成数据集包括 12000 带有雨量密度的训练图片和1200张测试图片。据作者所知这是第一个包含雨量标签的数据集。尽管是在合成数据上训练的，它可以泛化到真实世界的图片。
4. 在三个高挑战性的数据集上（两个合成、一个真实）和比赛上进行了大量的实验。

## 2 背景和相关工作
### 2.1 单图像去雨理论基础
数学上，一张雨图y可以被认为是雨滴r和背景x的结合，即

<img src="https://latex.codecogs.com/gif.latex?y&space;=&space;x&space;&plus;&space;r" title="y = x + r" />

在单图像去雨中，主要任务是根据雨图y恢复背景x

### 2.2 不同尺度特征聚合

用densely-connected block 作为构建模块，然后连接每个块的特征。

## 3 Proposed Method

DID-MDN（Density-aware Image De-raining Multi-stream Dense Network）主要分为两个部分：
1. residual-aware classifier,残差感知雨量密度分类器，主要用来确定雨量密度信息
2. multi-stram dense network，多流密集感知去雨网络，主要基于多流密集连接网络，考虑雨线的尺寸和形状，结合雨量密度信息进行去雨。

![](http://ww1.sinaimg.cn/large/006ocvumgy1g0f2tqajs8j31f80o6grd.jpg)

### 3.1 残差感知雨密度分类器（Residual-aware Rain-density Classifier）
1. 不同雨密度下效果不好

    尽管一些过去的方法性能达到了明显的进步，但是它们往往会过度去雨或者欠去雨。主要是因为用一个网络很难学到在不同雨密度下情况。为了精确估计有雨图片的密度，这里提出残差感知雨密度分类器。

2. 通用分类模型不好使

    训练一个新分类器的的常用策略是微调一个预训练模型，像VGG-16/Res-net/Dense-net。然而我们发现，在我们任务里直接微调一个深度模型不是一个有效的方案。这主要是因为高级特征倾向于关注定位离散物体。然而小的雨线不能集中于高级特征。换句话说，雨线信息会在高级特征中缺失，因此可能 会降低分类性能。提出一个更好的特征表示能够描述雨线特征很重要。

    把r=y−x当作用来描述雨密度的残差模块。为了从观察y估计残差模块<img src="https://latex.codecogs.com/gif.latex?\hat{r}" title="\hat{r}" />
    ，MDN用带雨密度的新数据集训练。估计的残差当作输入来训练最终的分类器。
    用这种方法，残差估计部分当作特征提取步骤。分类部分主要由三个卷积层（Conv kernel size 3x3）、一个平均池化层（AP kernel size 9x9）、两个全连接层(FC)。

    残差感知分类器Loss：
    1. 残差特征提取网络训练估计残差部分；
    2. 分类子网络用估计的残差作为输入来训练
    3. 两步联合起来优化。
    
    整体的损失函数：
    
    <img src="https://latex.codecogs.com/gif.latex?L=L_{E,r}&plus;L_{C}" title="L=L_{E,r}+L_{C}" />
    
    <img src="https://latex.codecogs.com/gif.latex?L_{C}" title="L_{C}" />表示雨密度分类的交叉熵损失
    ，<img src="https://latex.codecogs.com/gif.latex?L_{E,r}" title="L_{E,r}" />表示每个像素的欧氏损失。

### 3.2 多流密集感知去雨网络MDN
联合不同尺度的特征可是一个更有效的方法来获取变化的雨线。

基于观察和使用多尺度特征用于单张图像去雨的成功的刺激，这里提出一个更有效的多流稠密连接网络来估计雨线。每一路用不同核尺寸的dense-block创建。
多路block用Dense1(7x7)、Dense2(5x5)、Dense3(3x3)表示。
另外 ，为了进一步提升信息从不同block流动和利用每个dense-block的特征，介绍一种修改的连接，它所有的每个块的特征连接在一起用于雨线估计。而不是仅用每一路的两个卷积层，这里创建了不同特征的短路径来加强特征聚合和获得更好的收敛。

1. 为了利用雨密度信息来指导云雨操作，上采样label map与三路雨线的特征连接起来。
2. 然后连接特征用来估计残差<img src="https://latex.codecogs.com/gif.latex?\hat{r}" title="\hat{r}" />雨线信息。
3. 另外 ，residual 残差从输入雨图中减去，来估计粗糙的去雨图像 。
4. 最后，为了改善粗糙的去雨图像和确认更好的细节被保留，两个带relu的卷积层被采用作为最后的改善。

去雨网络的loss

<img src="https://latex.codecogs.com/gif.latex?L=L_{E,r}&plus;L_{E,d}&plus;\lambda_FL_F" title="L=L_{E,r}+L_{E,d}+\lambda_FL_F" />

<img src="https://latex.codecogs.com/gif.latex?L_{E,d}" title="L_{E,d}" />表示重建去雨图像每个像素的欧式损失，

<img src="https://latex.codecogs.com/gif.latex?L_{F}" title="L_{F}" />表示去雨图像的特征级损失，定义为

<img src="https://latex.codecogs.com/gif.latex?L_{F}=\frac{1}{CWH}||F(\hat{x})^{c,w,h}-F(x)^{c,w,h}||^2_2" title="L_{F}=\frac{1}{CWH}||F(\hat{x})^{c,w,h}-F(x)^{c,w,h}||^2_2" />


### 3.3 Testing
测试阶段，估计出用残差感知分类器的雨密度标签。然后，上采样label-map喂进多流网络，得到最终去雨图像
