# 目录

<!-- TOC -->

- [目录](#目录)
- [SE-ResNeXt描述](#SE-ResNeXt描述)
- [模型架构](#模型架构)
- [数据集](#数据集)
- [特性](#特性)
    - [混合精度](#混合精度)
- [环境要求](#环境要求)
- [快速入门](#快速入门)
- [脚本说明](#脚本说明)
    - [脚本及样例代码](#脚本及样例代码)
    - [脚本参数](#脚本参数)
    - [训练过程](#训练过程)
        - [训练](#训练)
        - [分布式训练](#分布式训练)
    - [评估过程](#评估过程)
        - [评估](#评估)
    - [导出过程](#导出过程)
        - [导出](#导出)
    - [推理过程](#推理过程)
        - [推理](#推理)
- [模型描述](#模型描述)
    - [性能](#性能)
        - [评估性能](#评估性能)
            - [ImageNet-1k上的SE-ResNeXt](#ImageNet-1k上的SE-ResNeXt)
        - [推理性能](#推理性能)
            - [ImageNet-1k上的SE-ResNeXt](#ImageNet-1k上的SE-ResNeXt)
- [ModelZoo主页](#modelzoo主页)

<!-- /TOC -->

# SE-ResNeXt描述

SE-ResNeXt是一个图像分类网络架构。作者注重于通道关系提出了Squeezeand-Excitation（SE）块，并在ResNeXt模型的基础上加入了该模块，提高了分类准确率。

[论文](https://arxiv.org/abs/1709.01507) ：Jie Hu, Li Shen, Samuel Albanie, Gang Sun, Enhua Wu."Squeeze-and-Excitation Networks"

# 模型架构

SE-ResNeXt的总体网络架构如下： [链接](https://arxiv.org/abs/1709.01507)

# 数据集

使用的数据集：ImageNet-1k

- 数据集大小：125G，共1000个类、125万张彩色图像
    - 训练集：120G，共120万张图像
    - 测试集：5G，共5万张图像
- 数据格式：RGB
    - 注：数据将在src/dataset.py中处理。

# 特性

## 混合精度

采用[混合精度](https://www.mindspore.cn/tutorial/training/zh-CN/master/advanced_use/enable_mixed_precision.html) 的训练方法，使用支持单精度和半精度数据来提高深度学习神经网络的训练速度，同时保持单精度训练所能达到的网络精度。混合精度训练提高计算速度、减少内存使用的同时，支持在特定硬件上训练更大的模型或实现更大批次的训练。

# 环境要求

- 硬件（Ascend）
    - 使用Ascend来搭建硬件环境。
- 框架
    - [MindSpore](https://www.mindspore.cn/install/en)
- 如需查看详情，请参见如下资源：
    - [MindSpore教程](https://www.mindspore.cn/tutorial/training/zh-CN/master/index.html)
    - [MindSpore Python API](https://www.mindspore.cn/doc/api_python/en/master/index.html)

# 快速入门

通过官方网站安装MindSpore后，您可以按照如下步骤进行训练和评估：

- Ascend处理器环境运行

  ```python
  # 运行训练示例
  python train.py --device_id=0 > train.log 2>&1 &

  # 运行分布式训练示例
  bash ./scripts/run_train.sh [RANK_TABLE_FILE] imagenet

  # 运行评估示例
  python eval.py --checkpoint_path ./ckpt_0 > ./eval.log 2>&1 &

  # 运行推理示例
  bash run_infer_310.sh [MINDIR_PATH] [DATA_PATH] [DEVICE_ID]
  ```

  对于分布式训练，需要提前创建JSON格式的hccl配置文件。

  请遵循以下链接中的说明：

 <https://gitee.com/mindspore/mindspore/tree/master/model_zoo/utils/hccl_tools.>

# 脚本说明

## 脚本及样例代码

```bash
├── model_zoo
    ├── README.md                    // 所有模型相关说明
    ├── SE-ResNeXt
        ├── README_CN.md             // SE-ResNeXt相关说明
        ├── ascend310_infer          // 实现310推理源代码
        ├── scripts
        │   ├──run_eval.sh           // Ascend评估的shell脚本
        │   ├──run_infer_310.sh      // Ascend推理的shell脚本
        │   ├──run_train.sh          // 分布式到Ascend的shell脚本
        ├── src
        │   ├──config.py             // 参数配置
        │   ├──dataset.py            // 创建数据集
        │   ├──senet_ms.py           // SE-ResNeXt架构
        ├── eval.py                  // 评估脚本
        ├── export.py                // 将checkpoint文件导出到air/mindir
        ├── postprocess.py           // 310推理后处理脚本
        ├── train.py                 // 训练脚本
```

## 脚本参数

在config.py中可以同时配置训练参数和评估参数。

- 配置SE-ResNeXt和ImageNet-1k数据集。

  ```python
  'name':'imagenet'        # 数据集
  'pre_trained':'False'    # 是否基于预训练模型训练
  'num_classes':1000       # 数据集类数
  'lr_init':0.4            # 初始学习率，八卡并行训练时设置为0.4，单卡训练时可以设置为0.05
  'batch_size':128         # 训练批次大小
  'epoch_size':120         # 总计训练epoch数
  'momentum':0.9           # 动量
  'weight_decay':1e-4      # 权重衰减值
  'image_height':224       # 输入到模型的图像高度
  'image_width':224        # 输入到模型的图像宽度
  'data_path':'/data/ILSVRC2012_train/'  # 训练数据集的绝对全路径
  'val_data_path':'/data/ILSVRC2012_val/'  # 评估数据集的绝对全路径
  'device_target':'Ascend' # 运行设备
  'device_id':0            # 用于训练或评估数据集的设备ID，使用run_train.sh进行分布式训练时可以忽略
  'keep_checkpoint_max':30 # 最多保存30个ckpt模型文件
  'checkpoint_path':'./train_parallel6/ckpt_6/train_senet_imagenet-110_1251.ckpt'  # checkpoint文件保存的绝对全路径
  ```

更多配置细节请参考脚本`config.py`。

## 训练过程

### 训练

- Ascend处理器环境运行

  ```bash
  python train.py --device_id=0 > train.log 2>&1 &
  ```

  上述python命令将在后台运行，可以通过生成的train.log或log文件查看结果。

  训练结束后，可以在默认脚本文件夹下找到检查点文件，采用以下方式得到损失值：

  ```bash
  # grep "loss is " train.log
  ...
  epoch: 22 step: 1251, loss is 2.7866466
  epoch: 23 step: 1251, loss is 2.7370944
  ...
  ```

### 分布式训练

- Ascend处理器环境运行

  ```bash
  bash ./scripts/run_train.sh [RANK_TABLE_FILE] imagenet
  ```

  上述shell脚本将在后台运行分布训练。

## 评估过程

### 评估

- 在Ascend环境运行时评估ImageNet-1k数据集

  “./ckpt_0”是保存了训练好的.ckpt模型文件的目录。

  ```bash
  python eval.py --checkpoint_path ./ckpt_0 > ./eval.log 2>&1 &
  OR
  bash ./scripts/run_eval.sh
  ```

## 导出过程

### 导出

将checkpoint文件导出成mindir格式模型。

  ```shell
  python export.py --ckpt_file [CKPT_FILE]
  ```

## 推理过程

### 推理

在进行推理之前我们需要先导出模型。mindir可以在任意环境上导出，air模型只能在昇腾910环境上导出。以下展示了使用mindir模型执行推理的示例。

- 在昇腾310上使用ImageNet-1k数据集进行推理

  推理的结果保存在scripts目录下，在acc.log日志文件中可以找到类似以下的结果。

  ```shell
  # Ascend310 inference
  bash run_infer_310.sh [MINDIR_PATH] [DATA_PATH] [DEVICE_ID]
  Total data: 50000, top1 accuracy: 0.79388, top5 accuracy: 0.94522.
  ```

# 模型描述

## 性能

### 评估性能

#### ImageNet-1k上的SE-ResNeXt

| 参数                 | Ascend                                                      |
| -------------------------- | ----------------------------------------------------------- |
| 模型版本              | SE-ResNeXt                                                |
| 资源                   | Ascend 910               |
| 上传日期              | 2021-07-31                                 |
| MindSpore版本          | 1.2.0                                                 |
| 数据集                    | ImageNet-1k，5万张图像                                                |
| 训练参数        | epoch=120, batch_size=128, lr_init=0.4（八卡为0.4，单卡可以设为0.05）             |
| 优化器                  | Momentum                                                    |
| 损失函数              | Softmax交叉熵                                       |
| 输出                    | 概率                                                 |
| 分类准确率             | 八卡：top1:79.38%, top5:94.51%               |
| 速度                      | 八卡：435毫秒/步                        |
| 总时长                 | 八卡：18.34小时/120轮                                             |

### 推理性能

#### ImageNet-1k上的FishNet99

| 参数                 | Ascend                                                       |
| -------------------------- | ----------------------------------------------------------- |
| 模型版本              | SE-ResNeXt                                                |
| 资源                   | Ascend 310               |
| 上传日期              | 2021-07-31                                 |
| MindSpore版本          | 1.2.0                                                 |
| 数据集                    | ImageNet-1k，5万张图像                                                |
| 分类准确率             | top1:79.39%,top5:94.52%                       |
| 速度                      | Average time 15.1009 ms of infer_count 50000                        |

# ModelZoo主页  

 请浏览官网[主页](https://gitee.com/mindspore/mindspore/tree/master/model_zoo)。