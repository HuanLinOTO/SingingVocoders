# SingingVocoders
A collection of neural vocoders suitable for singing voice synthesis tasks.

# English version [README_en.md](README_en.md)
## If you have any questions, please open an issue.


# 预处理 
python [process.py](process.py) --config 配置文件 --num_cpu 并行数量 --strx 1 代表 强制绝对路径 0 代表相对路径

和预处理有关的配置文件项
```
DataIndexPath: dataX11  这个是训练数据 index 的位置预处理会自动生成

valid_set_name: validX 这个是val index 的名字预处理会自动生成

train_set_name: trainX 这个是训练的 index 的名字预处理会自动生成

data_input_path: []  这个是你的 wav 的输入目录

data_out_path: [] 这个是你的 npz 的输出目录, 预处理之后的格式是 npz

val_num: 1 这个是你要的 val 数量 
```

例子
```
data_input_path: ['wav/in1','wav/in2'] 这个是你的wav的输入目录

data_out_path: ['wav/out1','wav/out2']这个是你的npz的输出目录
val_num: 5 这个是你要的 val 数量，预处理的时候会自动抽取文件
两个列表里面的路径是一一对应的所以说数量要一样

然后预处理的时候会扫描全部的 .wav 文件，包括子文件夹的

正常情况下只有这三个要改
```
# 离线数据增强
将预处理脚本替换为[process_aug.py](process_aug.py) 并增添配置项
```
key_aug: false 表示训练时不进行增强
aug_min: 0.9 最小变调倍数
aug_max: 1.4 最大变调倍数
aug_num: 1 数据增强倍数
```
即可，注意数据增强可能会损伤音质！
# 在线数据增强（推荐）
增加配置项，注意使用在线数据增强请使用[process.py](process.py) 脚本，否则会造成离线增强与在线增强叠加
```angular2html
key_aug: true 表示在训练时进行增强
key_aug_prob: 0.5 增强概率
aug_min: 0.9 最小变调倍数
aug_max: 1.4 最大变调倍数
```
注意数据增强可能会损伤音质！
# 训练
python [train.py](train.py) --config 配置文件 --exp_name ckpt名字 --work_dir 工作目录（可选）

# 导出
python [export_ckpt.py](export_ckpt.py) --ckpt_path ckpt路径  --save_path 导出的ckpt路径 --work_dir 工作目录（可选） 

# 注意

因为 pytorch-lightning 的问题所以说在 GAN 训练过程中实际的步数是它显示步数的一半

如果你需要微调社区声码器权重建议使用[ft_hifigan.yaml](configs%2Fft_hifigan.yaml) 配置文件

如何使用微调功能建议参考 openvpi/diffsinger [项目文档](https://github.com/openvpi/DiffSinger/blob/main/docs/BestPractices.md#fine-tuning-and-parameter-freezing)

少量步数的微调可以冻结 mpd 模块

建议不要用 bf16 可能会产生音质问题

少量数据差不多 2000 步就可以微调完成

# 快速开始

## 预处理
以下是你需要根据自己的数据集修改的配置项
```angular2html

data_input_path: []  这个列表 是你原始wav文件的路径

data_out_path: [] 此列表 预处理输出的npz文件的路径

val_num: 1 这个是在验证的时候 抽取的音频文件数量
```
然后执行预处理
```angular2html
python process.py --config (your config path) --num_cpu (Number of cpu threads used in preprocessing)  --strx (1 for a forced absolute path 0 for a relative path)
```
## 训练
```angular2html
python train.py --config (your config path) --exp_name (your ckpt name) --work_dir Working catalogue (optional)
```
测试中的配置项
```angular2html
use_stftloss: false  是否启用stft loss
lab_aux_melloss: 45
lab_aux_stftloss: 2.5 两种loss的混合控制
```
如果有其他需要可以修改 stftloss 的其他相关参数
## 导出
```angular2html
python export_ckpt.py --ckpt_path (your ckpt path)  --save_path (output ckpt path) --work_dir Working catalogue (optional)
```
# 注意事项
实际步数是显示的一半

微调 nsf-hifigan 声码器请将 [releases](https://github.com/openvpi/SingingVocoders/releases) 中的权重解压后放至主目录下，并使用 [ft_hifigan.yaml](configs%2Fft_hifigan.yaml)

微调请使用 44100 Hz 采样率音频，并不要修改其他 mel 参数，除非你明确知道你在做什么

微调功能使用请参考 openvpi/DiffSinger [项目文档](https://github.com/openvpi/DiffSinger/blob/main/docs/BestPractices.md#fine-tuning-and-parameter-freezing)

导出的权重可以在 [DDSP-SVC](https://github.com/yxlllc/DDSP-SVC), [Diffusion-SVC](https://github.com/CNChTu/Diffusion-SVC), [so-vits-svc](https://github.com/svc-develop-team/so-vits-svc) 和 [DiffSinger (openvpi)](https://github.com/openvpi/DiffSinger) 等项目中使用

如果要进一步导出成在 [OpenUtau](https://github.com/stakira/OpenUtau) 中使用的 onnx 格式权重，请使用 [这个](https://github.com/openvpi/DiffSinger/blob/main/scripts/export.py) 脚本

配置文件中配置项的继承关系为: [base.yaml](configs%2Fbase.yaml) -> [base_hifi.yaml](configs%2Fbase_hifi.yaml) -> [ft_hifigan.yaml](configs%2Fft_hifigan.yaml)

不要使用bf16训练模型, 它可能导致音质问题

2000 步左右即可微调完成 (显示的是4000步)

冻结 mpd 模块可能可以有更好的结果

# 其它模型
[HiFivae.yaml](configs%2FHiFivae.yaml)hifivae.yaml 训练vae模型

[base_hifi_chroma.yaml](configs%2Fbase_hifi_chroma.yaml) 训练忽略8度nsf hifigan

[base_hifi.yaml](configs%2Fbase_hifi.yaml) 训练nsf hifigan

[base_ddspgan.yaml](configs%2Fbase_ddspgan.yaml) 训练带鉴别器的ddsp模型

[ddsp_univnet.yaml](configs%2Fddsp_univnet.yaml) 训练ddsp 混合univnet模型

[nsf_univnet.yaml](configs%2Fnsf_univnet.yaml) 训练带nsf的univnet（推荐）

[univnet.yaml](configs%2Funivnet.yaml) 训练原版univnet

[lvc_base_ddspgan.yaml](configs%2Flvc_base_ddspgan.yaml) 训练使用lvc滤波器的 ddsp模型
