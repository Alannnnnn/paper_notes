# Finding Tiny Faces in the Wild with Generative Adversarial Network
## 当前技术背景
  人脸识别的技术已经十分成熟了，但是在自然环境中的微小人脸的识别却不尽如人意。其中主要是由于微小的人脸，大多缺乏细节信息，并且十分模糊。现代的人脸识别技术在中大型的脸上取得了不错的成果，但是在微型的人脸上检测上遇到的困难主要是，小脸上缺乏足够的信息让脸与背景分开，例如部分面部和手部的区域。另一个问题是现在的基于cnn的人脸识别器采用下采样卷积特征图来表示面部，这样丢失了太多空间信息，太过粗糙不能描述微小面部。

## 他人研究现状
  其中文献28（X.Xu,D.Sun,J.Pan,Y.Zhang,H.Pfister,andM.-H.Yang. Learning to super-resolve blurry face and text images. In The IEEE International Conference on Computer Vision (ICCV), Oct 2017. 2, 3）使用双线性操作直接对图像进行上采样，并在上采样图像上彻底搜索面部。但是，这种方法会增加计算成本，消耗时间也会显著增加。
  
其余使用中间轮廓特征图来表示特定尺度的面，这保持了计算负担和性能之间的平衡。但是却会使得特征图缺乏辨识力，导致很多错误结果。

## 生成对抗网络（GAN）简介
  GAN的主要思想是拥有两个竞争的神经网络模型。 一个将噪声数据作为输入，并产生样本（所谓的生成器）。 另一个模型（称为鉴别器）从生成器和训练数据接收样本，并且必须能够区分两个来源。 这两个网络进行连续的博弈，生成器学习产生越来越多的现实样本，鉴别器正在学习越来越好地区分生成的数据和实际数据。 这两个网络同时进行训练，最后的希望是竞争能够使生成器生成的样本与实际数据不可区分。
## 文章技术简介
  本文采用生成对抗网络(GAN）从模糊的图像中生成高清晰度的人脸。并且设计了一个新的网络来解决超分辨和精细化的问题，  同时设计了一个新的loss 来指导生成器更好的还原细节，同时也促进鉴别器识别 真人脸和假人脸，无人脸和有人脸的图像。为了解决人脸检测中的麻烦，该文章提出了一种统一的端到端卷积神经网络，用于基于经典的生成对抗网络（GAN）框架进行更好的人脸检测。该文的探测器中有两个子网络，一个生成器网络和一个鉴别器网络。在生成器子网络中，使用超分辨率网络（SRN）对小面进行上采样，以便找到那些微小的面。与通过双线性操作重新调整大小相比，SRN可以减少伪像并提高上采样图像的质量，具有较大的放大因子（在我们当前实现中为4倍），然而，即使使用如此复杂的SRN，由于分辨率非常低（10×10像素）的面部，上采样图像也不令人满意（通常模糊并且缺少精细细节）。因此，提出了一种细化网络（RN）来恢复上采样图像中的一些缺失细节，并生成清晰的高分辨率图像以进行分类。在鉴别器子网中，我们引入了一种新的损失函数，该函数强制鉴别器网络同时区分真/假面和面/非面。生成的图像和真实图像通过鉴别器网络以共同区分它们是真实图像还是生成的高分辨率图像以及它们是面部还是非面部。

## 网络结构
### 生成网络
  包括：1.上采样子网  2.细化子网   
  与寻常GAN不同，并不是接受随机噪声为输入 。 第一个子网将低分辨率图像作为输入，输出是超分辨率图像。由于模糊的小脸缺乏细节，所生成的超分辨率面部通常模糊。因此，我们设计了第二个子网络来优化来自第一个子网络的超分辨率图像。此外，我们将分类分支添加到鉴别器网络以进行检测，这意味着我们的鉴别器可以对面部和非面部进行分类以及区分伪造和真实图像。
### 鉴别器网络
  采用VGG19 作为骨干网络，同时避免过多的下采样。
  ![生成器鉴别器网络架构](https://github.com/Alannnnnn/paper_notes/blob/master/82E66D88-D175-4BEB-8E62-D54667BC208E.png)
## 损失函数
  采用了以下三个损失来优化生成网络：
### 像素损失
  生成网络的输入是小模糊图像而不是随机噪声。强制生成器输出接近超分辨率的一种自然方法就是采用像素mse损失。
### 对抗性损失

  对抗性损失，帮助生成网络产生更加清晰细节，来使得鉴别器无法分辨。
### 分类损失

  起着两个作用，第一个是区分高分辨率图像，包括生成的和自然的真实高分辨率图像，是鉴别器网络中的面部还是非面部。另一个角色是促进生成器网络重构更清晰的图像。（在鉴别器中，引入了分类损失，来分类高分辨率图像是真实的还是假的，是人脸，还是非人脸。）
  
## 总结
该文主要做出以下三点贡献。 （1）提出了一种新的统一端到端卷积神经网络人脸检测体系结构，其中超分辨率和细化网络用于生成真实，清晰的高分辨率图像，并引入鉴别器网络对人脸进行分类（例如人脸/非人脸,真实图像还是生成图像）。 （2）引入新的损失以促进鉴别器网络同时区分真/假图像和面/非面。更重要的是，分类损失用于指导生成网络生成更清晰的面部以便于分类。 （3）最后，该文证明了他们的方法在模糊的小脸上恢复清晰的高分辨率面部的有效性，并表明检测性能优于WIDER FACE数据集上的其他最先进的方法，尤其是最具挑战性的Hard子集。


 
