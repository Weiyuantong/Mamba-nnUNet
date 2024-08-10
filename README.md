# 项目操作流程

   本项目在 **Ubuntu 18.04.3 LTS** 系统下部署运行，还适用**Ubuntu 20.04**版本。

   软件环境为 **Anaconda 4.1.0 和 PyCharm Community Edition 2023.1.2**

# 一.配置环境

## 1.conda创建环境

本项目在**python==3.9.19**环境下运行

`conda create -n nnunet python=3.9`

## 2.安装nnUNet，Mamba框架

`pip install nnunet`

`pip install mamba-ssm`

`pip install causal-conv1d>=1.2.0`

## 3.配置torch

由于本项目的linux服务器的cuda版本为11.3，我们要配置适用于本项目的torch。

`conda install pytorch==1.10.0 torchvision==0.11.1 torchaudio==0.10.0 cudatoolkit=11.3 -c pytorch`

也可以根据自己硬件的cuda版本来选择对应版本的pytorch

## 4.离线安装nnUNet包（可选）
**第一步**：点击百度网盘链接：https://pan.baidu.com/s/1hzKEVjweXPK1MeIsTSQcSQ?pwd=demu 
提取码：demu 

**第二步**：下载python包 `nnunet_package.zip `到本地 `project/`

**第三步**：`cd project/nnunet_package`

**第四步**：安装`project/`目录下的`nnunet_package`的python包   `pip install ./*`


# 二.数据前预处理
调整输入CT图像的hu值，调整肺窗的窗宽 `lung_window_width` 和
窗位 `lung_window_center`

具体步骤：配置`project/python/调整窗宽窗位.py` 文件

设置CT图像 `lung_window_center=45`, `lung_window_width=150`

![窗位窗宽设计](picture/picture1.png)

在命令行运行 `python project/python/调整窗宽窗位.py`
# 三. 数据集准备

nnUNet对于你要训练的数据是有严格要求的，体现在我们保存数据的路径上

**第一步**：建立文件夹拓扑结构，使用github上的集成框架

**在项目路径下运行命令**
`cd project/python`
`git clone https://github.com/MIC-DKFZ/nnUNet.git`
`cd nnUNet`

nnUNet的文件夹拓扑结构搭建完毕

**第二步**：将数据集放入 `nnUNet_raw_data` 文件夹。
对应 `project/python/nnunet/nnUNetFrame/DATASET/nnUNet_raw` 路径下的 `nnUNet_raw_data` 文件夹
数据集的文件夹名字应是 `Task0X_name` ,本项目为 `Task02_liver`

其中 `nnUNet_raw_data` 存放原始数据，`nnUNet_cropped_data ` 存放nnUNet模型预处理数据

**第三步**：指定image和label路径。
在`Task02_Liver`里建立`imagesTr`，`labelsTr`,`imagesTs`文件夹

`imagesTr`:用于存放原始image，即不带标签的CT图像，格式为.nii.gz

`labelsTr`:用于存放原始标签label，即不带标签的CT图像，格式为.nii.gz

**第四步**：编写JSON文件。
以下是目标json文件格式：
![Json文件](picture/picture2.png)

配置  **`project/python/生成json.py`** 文件

![输入图片说明](picture/picture3.png)

`name`: 指定json文件里的任务名字            `description`: 项目描述

`labels`: `0` 代表 `图像背景`， `1` 代表 `分割对象`

`num_training`: 训练集文件个数        `num_test`: 测试集文件个数

`modality`:  文件模态，本项目模态为`CT`

`tensor_image_size`: 文件维度，本项目数据为三维图像

![输入图片说明](picture/picture4.png)

`training_images`: 指定 `image` 的相对地址
`training_labels`: 指定 `label` 的相对地址
`output_folder`: 指定json文件输出保存的路径

**自动生成json**：  `python project/python/生成json.py`

最后的数据集格式应为
nnUNet_raw_data_base/nnUNet_raw_data/Task02_Liver/
├── dataset.json
├── imagesTr
│   ├── liver_001.nii.gz
│   ├── liver_001.nii.gz
│   ├── ...
├── labelsTs
│   ├── liver_001.nii.gz
│   ├── liver_002.nii.gz
│   ├── ...
————————————————


# 四.配置Linux环境

这一步是指定nnUNet读取文件的路径

**第一步**：进入conda环境 `conda activate nnunet`

**第二步**：使用vim编辑.bashrc文件  `vim ~/.bashrc`

**第三步**：指定 `nnUNet_preprocessed`, `nnUNet_raw`, `nnUNet_trained_models` 文件夹路径

将光标移至 `.bashrc` 文件最下端， 输入：

`export nnUNet_raw_data_base="/home/work/nnUNet/nnUNetFrame/DATASET/nnUNet_raw"`

`export nnUNet_preprocessed="/home/work/nnUNet/nnUNetFrame/DATASET/nnUNet_preprocessed"`

`export RESULTS_FOLDER="/home/work/nnUNet/nnUNetFrame/DATASET/nnUNet_trained_models"`

其中 **`/home/work`** 为相对路径，从**用户名**到**项目根路径**的路径。

**第四步**：保存设置

`source ~/.bashrc` 


# 五.数据集转化

![输入图片说明](picture/picture5.png)
比如Task02_Liver的任务名称为“Liver”，任务ID为2。

数据集转换命令： `nnUNet_convert_decathlon_task -i /home/work/nnUNet/nnUNetFrame/DATASET/nnUNet_raw/nnUNet_raw_data/Task02_Liver
`

**nnUNet要求将原始数据转换成特定的格式，以便了解如何读取和解释数据。**
每个分割数据集存储为单独的“任务”。**命名包括任务与任务ID，即三位整数和相关联的任务名称。**

数据集转换操作完成以后，在你的`Task02_Liver`文件夹旁边，出现了一个`Task002_Liver`文件夹

Task001_BrainTumour/ 
├── dataset.json 
├── imagesTr 
│   ├── liver_001_0000.nii.gz
│   ├── liver_002_0000.nii.gz
│   ├── ...
├── labelsTs
│   ├── liver_001_0000.nii.gz
│   ├── liver_002_0000.nii.gz
│   ├── ...   
————————————————

# 六.数据后预处理

数据集后预处理命令： `nnUNet_plan_and_preprocess -t 2 --verify_dataset_integrity`

我们的 `Task_id`是 `2`，所以这里的数字就是 `2`

输出结果存放在 `nnUNet_preprocessed/Task002_Liver` 文件夹里


# 七.模型训练

**对于我们的Task02_Liver来说，应该运行的命令如下：**
`nnUNet_train 3d_fullres nnUNetTrainerV2 2 3`

`3d_fullres` 是指 `3D全分辨率` 网络架构，还可以选择 `3d_lowres` 和 `3d_cascade_fullres`       经过比较，**`3d_fullres`**  性能最佳

`2`: 任务ID        
`nnUNetTrainerV2`: 选用的训练器
`3`: 指 `K折交叉检验` 的 `折数`,   可以指定从 `0` 到 `4`折，本项目共训练了5折

训练结束后的模型文件会保存在 `nnunet/nnUNetFrame/DATASET/nnUNet_trained_models` 文件夹里

**注意：项目源代码中只保留了性能最好的第四折模型**

# 八.模型推理
**运行命令：**
`nnUNet_predict -i 要预测数据的文件夹路径 -o 输出文件夹路径 -t 2 -m 3d_fullres -f 3`

`要预测数据的文件夹路径`:  经过数据集转化后的image文件路径，是待推理文件的路径

**注意：`要预测数据的文件夹路径` 里的文件命名方式必须以 `_XXXX` 结尾，在本项目中全部以 `_0000` 结尾**

![输入文件命名示例](picture/picture8.png)

`输出文件夹路径`: 预测推理输出的标签存放路径

`2`: 任务ID，本项目是 `2`

`3`: `K折交叉验证`的折数

`3d_fullres` 是指 `3D全分辨率` 网络架构


# 九.检验模型推理效果

配置 `project/python/cal.py` 文件

![输入图片说明](picture/picture6.png)

`root_path`: `真实标签的绝对路径`
`infer_path`: `预测标签的绝对路径`              

**注意：标签的路径里不要包含中文字符!!!，建议采用全英文绝对路径，可以放置在系统磁盘根目录下**

输出结果包括：`Processing Data`  , `Single case Dice` , `Single case Hausdorff Distance`

`Processing Data` : `当前推理预测的 .nii.gz 标签文件`

`Single case Dice` : `推理结果的 Dice 系数，表示模型分割结果与专家手工勾画结果的重合度`

`Single case Hausdorff Distance` : `推理结果的 HD 系数，模型分割结果与专家手工勾画结果三维表面的距离差异`

<img src="picture/picture7.png" style="zoom:150%;" />

`Single case Dice` 系数越大，代表推理出的结果与真实勾画结果重合度越高，推理效果越好

`HD` 系数越小，代表推理出的结果与真实勾画结果三维表面距离越小，推理效果越好
