# Day 1总结：计算机视觉与OpenMMLab开源算法体系
## CV 常见任务目标

- 目标检测
- 目标分类
- 目标定位
- 目标识别
- 语义分割
- 实例分割
- 关键点检测

<image src="img/objectives.png"/>
<br/>

随着研究不断深入，深度学习模型的精度越来越高，2015年ResNet模型已经超越了人类的识别能力。
神经网络结构越来越复杂， 层数越来越深。

<image src="img/accuracy.png" align="center" width="900"/>
<br/>
<image src="img/depth_grow.png" align="center" width="900"/>
<br/>

## openmm 框架
由openMMLab主导, 社区共建。
openMM是基于Pytorch搭建的Code Base，在Pytorch的基础上增加了许多新功能。
被认为是当代最完整的计算机视觉开源算法体系。同时兼具先进的底层架构，最前沿的算法支持与开箱即用的便捷性。

<image src="img/structure_2.0.png" align="center" width="900"/>
<br/>

## 机器学习基本流程
- 训练
- 验证
- 应用

<image src="img/routines.png" align="center" width="900"/>
<br/>

- 通过反向传播与梯度下降不断更新网络参数
<image src="img/training.png" align="center" width="900"/>
<br/>

训练数据不当，次数过多可能产生过拟合，应当避免。
<image src="img/overfitting.png" align="center" width="900"/>
<br/>

注：全部图片来自子豪师兄直播课截图，侵删。
