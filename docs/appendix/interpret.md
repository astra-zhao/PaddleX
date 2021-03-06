# PaddleX可解释性

目前深度学习模型普遍存在一个问题，因为使用模型预测还是一个黑盒，几乎无法去感知它的内部工作状态，预测结果的可信度一直遭到质疑。为此，PadlleX提供了2种对图像分类预测结果进行可解释性研究的算法：LIME和NormLIME。

## LIME
LIME全称Local interpretable model-agnostic explanations，表示一种与模型无关的局部可解释性。其实现步骤主要如下：
1. 获取图像的超像素。  
2. 以输入样本为中心，在其附近的空间中进行随机采样，每个采样即对对象中的超像素进行随机遮掩（每个采样的权重和该采样与原样本的距离成反比）。  
3. 每个采样通过预测模型得到新的输出，这样得到一系列的输入`X`和对应的输出`Y`。  
4. 将`X`转换为超像素特征`F`，用一个简单的、可解释的模型`Model`（这里使用岭回归）来拟合`F`和`Y`的映射关系。  
5. `Model`将得到`F`每个输入维度的权重（每个维度代表一个超像素），以此来解释模型。  

LIME的使用方式可参见[代码示例](https://github.com/PaddlePaddle/PaddleX/blob/develop/tutorials/interpret/lime.py)和[api介绍](../apis/visualize.html#lime)。在使用时，参数中的`num_samples`设置尤为重要，其表示上述步骤2中的随机采样的个数，若设置过小会影响可解释性结果的稳定性，若设置过大则将在上述步骤3耗费较长时间；参数`batch_size`则表示在计算上述步骤3时，预测的batch size，若设置过小将在上述步骤3耗费较长时间，而上限则根据机器配置决定。  

最终LIME可解释性算法的可视化结果如下所示：  
![](../images/lime.png)  
图中绿色区域代表起正向作用的超像素，红色区域代表起反向作用的超像素，"First n superpixels"代表前n个权重比较大的超像素（由上述步骤5计算所得结果）。


## NormLIME
NormLIME是在LIME上的改进，LIME的解释是局部性的，是针对当前样本给的特定解释，而NormLIME是利用一定数量的样本对当前样本的一个全局性的解释，有一定的降噪效果。其实现步骤如下所示：  
1. 下载Kmeans模型参数和ResNet50_vc网络前三层参数。（ResNet50_vc的参数是在ImageNet上训练所得网络的参数；使用ImageNet图像作为数据集，每张图像从ResNet50_vc的第三层输出提取对应超象素位置上的平均特征和质心上的特征，训练将得到此处的Kmeans模型）  
2. 计算测试集中每张图像的LIME结果。（如无测试集，可用验证集代替）  
3. 使用Kmeans模型对所有图像中的所有像素进行聚类。  
4. 对在同一个簇的超像素（相同的特征）进行权重的归一化，得到每个超像素的权重，以此来解释模型。  

NormLIME的使用方式可参见[代码示例](https://github.com/PaddlePaddle/PaddleX/blob/develop/tutorials/interpret/normlime.py)和[api介绍](../apis/visualize.html#normlime)。在使用时，参数中的`num_samples`设置尤为重要，其表示上述步骤2中的随机采样的个数，若设置过小会影响可解释性结果的稳定性，若设置过大则将在上述步骤3耗费较长时间；参数`batch_size`则表示在计算上述步骤3时，预测的batch size，若设置过小将在上述步骤3耗费较长时间，而上限则根据机器配置决定；而`dataset`则是由测试集或验证集构造的数据。  

最终NormLIME可解释性算法的可视化结果如下所示：  
![](../images/normlime.png)  
图中绿色区域代表起正向作用的超像素，红色区域代表起反向作用的超像素，"First n superpixels"代表前n个权重比较大的超像素（由上述步骤5计算所得结果）。图中最后一行代表把LIME和NormLIME对应超像素权重相乘的结果。