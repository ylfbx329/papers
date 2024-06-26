# Video Visual Relation Detection 视频视觉关系检测

## 摘要

视觉关系链接视觉和语言，由（主语，谓语，宾语）三元组描述，能够超越物体更全面的理解视觉内容。本文提出了一种新的视觉任务即视频视觉关系检测（VidVRD）以在视频中进行视觉关系检测而不是图像中（ImgVRD）。相比于图像，视频提供了一种更自然的视觉关系检测的特征，如动态关系和时间关系。然而，由于准确目标追踪的困难和在视频中存在多样化的关系，VidVRD比ImgVRD更困难。为此提出了一种包含目标追踪、短期关系预测和贪心关系关联的VidVRD方法。而且制作了第一个VidVRD数据集以进行验证，其包含1000个手动标注视觉关系的视频，以验证本文方法。

## 介绍

弥合视觉和语言之间的差距在多媒体分析中至关重要，这吸引了大量的研究工作，包括视觉概念注释[3,24]、带字幕的语义描述[7]和视觉问答[1]等。视觉关系检测（VRD）是最近对对象之外的视觉内容提供更全面理解的努力，旨在捕捉对象之间的各种交互[22]。它可以有效地支持许多视觉语言任务，例如字幕[15,34]、视觉搜索[2,8]和视觉问答[1,21]。

视觉关系包含一对被边界框定位的物体和一个连接的谓词，其中两个物体可以被各种谓词连接，相同的谓词也可以连接不同表现的物体。本文使用三元关系定义一种视觉关系，表示一种唯一的（主语、谓语、宾语）组合。由于组合的复杂性，三元关系的可能空间远大于对象。因此现有的目标检测方法不适用于VRD。针对VRD提出的方法实际仍然仅适用于静止图像。相比于静止图像，视频提供了一组更自然的特征来检测视觉关系，如物体间的动态相互影响。从视频的时空信息中提取的动态特征能够帮助针对相似预测的消除歧义。同时一些视觉关系比如动态关系只能在视频中检测出来，因为相比于ImgVRD，VidVRD更加通用和可行。

二者另一个不同是视频中的视觉关系随着时间而变化，物体可能短暂的出现或消失在帧图像中从而导致视觉关系的出现与否。甚至当两个物体同时出现在相同的帧中，其相互作用可能短暂的改变。因此VidVRD任务应该能被重新定义为解决视觉关系的易变性。

定义：为了与ImgVRD的定义保持一致，VidVRD定义为给定一组感兴趣的对象类别C和谓语类别P，VidVRD旨在视频中检测C*P*C的视觉关系实例，其中视觉关系实例由关系三元组（主语、谓语、宾语）表示，主语和宾语的轨迹为Ts和To。Ts和To是两个边界边界框的序列，在视觉关系的最大持续时间内分别框选主语和宾语。

相比于ImgVRD，VidVRD面临更多技术挑战。首先，VidVRD需要用边界框轨迹定位物体。因为边界框轨迹的精度受每帧目标定位性能和目标跟踪性能的影响。我们提出的VidVRD方法（图3）通过在视频的每个重叠短片段中生成对象轨迹来解决这个困难，然后根据预测的视觉关系将它们关联到对象轨迹中。第二，VidVRD需要在最大的时间段内定位视觉关系。我们提出一个贪心的关联算法以合并视觉实例检测在相邻的片段中，如果他们有相同的关系三元组并且轨迹高度重合。第三，VidVRD需要预测更多类型的视觉关系，因为一些视觉关系仅能在视频中被检测。为了有效的关系预测，我们提出了关系预测模型，其从主客体对中提取多样的特征，特征包括外观、运动和相对特征。我们将这些特征编码为一个关系特征，并使用单独的主语、谓语和宾语预测器来预测视觉关系。

构建VidVRD数据集，构建谓语描述机制，从ILSVRC2016-VID中构建数据集。包含1000张手动标注视觉关系和物体框轨迹的视频。

本文主要贡献有：
1. 提出一项新的VidVRD任务，旨在探索视频中的物体视觉关系，相比于ImgVRD，其是一种更加灵活的VRD任务。
2. 我们提出了一种VidVRD方法，通过对象轨迹提议、关系预测和贪婪关系关联来检测视频中的视觉关系
3. 构建了包含1000个手动标注视觉关系的视频的VidVRD验证数据集

## 相关工作

视频目标检测，旨在给定的视频中检测属于预先定义类别的物体，并且使用边界框轨迹定位他们。SOTA方法通过集成最新的图像目标检测和多目标追踪技术解决问题。由于视频存在模糊、相机运动和物体遮挡，目标检测的准确度仍然较低。另一方面，tracking-by-detection策略的多目标跟踪由于目标检测器的高漏检率往往会产生较短的轨迹，因此开发了额外的合并算法来获得更时间一致的目标轨迹。提出利用视频目标检测器生成短期的目标轨迹提案，其避免了其共同的弱点。请注意，我们的方法可以应用于任何图像目标识别和多目标跟踪方法之上。

视觉关系检测。最近的研究工作在于图像的VRD。显而易见的是在VRD中的一个基本的挑战是如何通过学习仅有的训练样本建模和预测巨大数量的关系。现存方法分别预测主语谓语和宾语。将复杂度从O(N^2K)降低到O(N+K)，N和K分别是目标和谓语的数量。一些方法利用语言先验知识和正则化关系嵌入空间显著提高了性能，提取关系相关特征是VRD的另一个症结。[5,42]特别使用基于坐标或二进制掩码的特征来增强检测空间关系的性能。[5,16]还研究了关系三元组组件之间的视觉特征级别连接，以利用额外的统计依赖关系，但是需要O(NK)的参数建模。因此，为了解决这些问题，我们将提出一个视频特定的关系特征和一个新的训练准则来学习单独的预测模型。

动作识别。因为动作在视觉关系中是一个主要的谓语类别，VidVRD能够利用动作识别的进步。在动作识别中特征表示扮演着关键角色在类内差异、背景模糊和相机运动中。手工制作的特征和深度神经网络都被开发用于解决这个问题。我们利用提升的密集轨迹方法（iDT）作为一部分的特征，因为iDT在大多数动作识别中仍然有杰出的性能，尤其是当训练数据不充分时，请注意，我们提出的方法旨在检测对象之间的一般关系而不是动作，例如空间和比较关系。

## 数据集

我们基于ILSVRC2016-VID[31]的训练集和验证集构建了第一个VidVRD评估数据集，其中包含30类对象的手动标注边界框的视频。在仔细查看和分析视频内容后，我们选择了1000个包含清晰和丰富视觉关系的视频，而单个对象和模糊视觉关系的视频被忽略。我们将视频集随机分成训练集和测验集，分别包含800个视频和200个视频。

基于这1000个视频，我们用视觉关系中经常出现的额外五个对象类别来补充30个对象类别，即人、球、沙发、滑板和飞盘。所有由此产生的35个对象类别描述了独立的对象，即我们不包括对象之间的部分关系，例如“带轮子的自行车”，在构建的数据集中。

接下来，我们构建谓语类别集如下：我们直接使用及物动词作为谓语，例如“骑”；我们以比较的格式将形容词转移到谓语中，例如“更快”；我们从相机视点手动定义常见的空间谓语以确保一致性，例如“上面”。虽然不及物动词通常只描述对象的属性，但它们在关系表示方面具有表现力。例如，在视觉关系中，“走在后面”比“后面”提供了更多的信息。因此，我们还包括不及物动词和空间谓语的组合，以及不及物动词和“with”的组合，后者表示以相同方式作用的两个对象。我们在谓语定义中排除介词，因为空间类的介词可以被定义的空间谓语覆盖，而剩下的介词类型主要与部分关系相关，根据宾语定义已经排除了。根据上述谓语定义机制和视频内容，我们选择了14个及物动词、3个比较级、11个空间描述符和11个不及物动词，能够导出160类谓语。在构建的数据集中，视频中出现了132个谓语类别。数量超过了以前的工作。

8名志愿者参与了视频标注，另外2名志愿者负责标注检查。在对象标注阶段，所有视频中属于附加5个类别的对象被手动标注其类别和边界框轨迹。在谓词标注阶段，为了考虑视觉关系是时间可变的，所有视频被预先分解为30帧的片段，其中15帧重叠。然后，要求对每个片段中出现的所有谓词进行标注，以获得片段级视觉关系实例。为了节省标注劳动，我们只标注训练集中的典型片段和测验集中的所有片段。对于测验集，具有相同对象对和谓词的相邻片段中的视觉关系实例被自动链接，以生成视频级视觉关系实例。

表1显示了构建的VidVRD数据集3的统计数据。总体而言，我们的数据集总共包含3,219个关系三元组（即视觉关系类型的数量），测验集有258个从未出现在训练集中的关系三元组。在实例级别，测验集包含4,835个视觉关系实例，其中432个实例在训练集中是看不见的。请注意，虽然测验集中的视频是完全标注的，但仍有一小部分内容没有任何视觉关系，因为这些视频的某些部分包含少于两个对象。从表1下部可用的分段级别统计数据来看，我们数据集中每个分段的视觉关系实例数量为9.5个，高于视觉关系数据集[22]中每个图像的7.6个实例，这表明我们的数据集更完整地标注了。

## 视频视觉关系检测

VidVRD的主要挑战是解决视觉关系随时间的易变性。为此，我们提出了一种短期检测视觉关系实例的VidVRD方法，然后通过关联算法形成视频中的整体视觉关系实例。该方法背后的假设是基本视觉关系总是可以在短时间内被识别，而更复杂的关系可以从基本视觉关系的序列中推断出来。检测短期视觉关系也有助于检测视频中视觉关系的出现和消失，并减轻直接分析长期持续时间的计算负担。

## 对象轨迹提案

给定的视频分解为长度为L，有L/2为重叠帧，并且对每个视频段生成对象轨迹提案。相比于生成整个视频的对象轨迹提案，短期提案能够减少目标追踪算法中常见的漂移问题，漂移是由光照变化和遮挡等引起的。每段单独的对象轨迹提案能够生成更多的不同的候选者。多样性对于后续的关系建模很重要，因为它为鲁棒建模提供了对象的各种外观和运动特点。

我们的对象跟踪建议是基于类似于[11]的视频目标检测方法在每个片段上实现的。首先，我们为数据集中使用的35个类别使用对象检测器来检测片段帧中的对象。对象检测器使用带有ResNet101[9]的Faster-RCNN[30]在由MS-COCO[19]和ILSVRC2016-DET[31]数据集中的35个类别的训练/验证图像组成的图像集上进行训练。其次，我们使用Dlib中[6]的有效实现来跟踪整个片段的帧级检测结果。为了减少重叠建议的数量，我们在生成的跟踪上执行vIoU>0.5的非最大抑制（NMS），其中vIoU表示两个跟踪的体积并交比。因此，我们平均每个片段生成19.7个对象跟踪建议。

## 关系预测

假设（Ts，To）是一个段中的一对对象轨迹提议，每个提议都是边界框序列的形式。预测关系三元组<主题，谓词，对象>涉及识别Ts和To的对象类别，以及它们之间的相互作用。在实践中，由于组合数量巨大且训练数据不足，不可能为每个单个关系三元组学习单独的预测器/决策函数。我们的模型（图4）学习了单独的主语，谓词和宾语预测器，以降低建模复杂性并利用各种关系中的公共组件。该模型还利用了丰富的关系特征，该特征结合了主语和宾语的外观和运动特征，以及它们之间的相对特征。

关系特征抽取。我们提取了Ts和To的对象特征来描述它们的外观和运动特征。特别是，我们首先提取了片段内的HoG、HoF、MBH和改进密集轨迹（iDT）特征[37]，这些特征捕获了运动和低级视觉特征。为了对特征进行编码，我们使用100,000个随机采样的特征为iDT中的四种描述符类型中的每一种训练密码本。每个密码本的大小设置为1,000。然后将T的对象特征计算为包含在T中的一袋iDT特征，其中iDT的一半位于T的区域内才被认为是包含的。此外，我们将对象特征附加到类特征[36]中，该类特征是由深度神经网络预测的分类概率（即N类）的N-d向量，以编码视觉外观中的语义属性。

为了提取Ts和To之间的相对特征，我们提出了一个描述两个物体之间相对位置、大小和运动的相关性特征。分别表示${C_s}^t=(x_s^t, y_s^t)$和${S_s}^t=(w_s^t, h_s^t)$作为时间t（resp.To）的中心点和Ts的大小，我们计算相对位置ΔC、相对大小ΔS和相对运动ΔM描述符作为：

为了表征丰富的空间关系，如“后面”、“更大”和“过去”，以及它们的各种组合，如“过去的后面”，我们使用字典学习为每种类型的描述符训练一个密码本。



