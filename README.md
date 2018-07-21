# Emphasis-Detection-Prosody-Transfer
首先需要下载 https://cloud.tsinghua.edu.cn/d/7947958556a0417cbdc6/ 中的两个wav文件，放在目录audio/original下，作为数据集。  
同时，需要到 http://www.sppas.org 下载SPPAS，并安装sppas的依赖，将文件夹SPPAS放在项目根目录下。
## Dependencies
* python3
* python2.7+
* ffmpeg
* tensorflow
* inaSpeechSegmenter
* pydub
* pandas
## Steps
除特殊说明外，建议用python3运行脚本。
### Data Retrieval
1. （*optional*）分别运行presegmentation.py中两段语句，在text/目录下会生成两个csv文件。该步骤耗时较长，因此csv文件将上传至repo中。
2. 运行segmentation.py，将cn.wav和en.wav两个原始音轨中的男声片段与女声片段切分出来，分别存储在audio/segment/cn(en)/male(female)目录下。
3. （*optional*）运行transcription.py，调用百度的语音识别api，将切分后的语音转录为文字，以csv格式存放在text文件夹下。四个csv文件同样上传至repo中。
4. 挑选可用数据，利用selection.py脚本生成描述可用数据集的csv文件text/dataset_meta.csv。
### Data Processing
1. 运行parallelization.py，将dataset_meta.csv中描述的所有语音与文字进行时域上的对齐，生成text/dataset_time_align.csv。
**注意：该步骤中会通过python的os模块执行shell命令调用SPPAS的python脚本，SPPAS的运行环境为python2.7+。
在os的函数调用中，默认使用`python`命令来执行python2.7。在不同的运行环境下，可能需要修改此处命令来完成python2的调用。**
2. 运行extraction.py，计算dataset_time_align.csv所约定的语音-文字对齐关系下的韵律特征，在text目录下生成 dataset_feature.csv
和dataset_feature_avg.csv两个文件，前者存储每个句子单位（词/字）的特征，后者存储整个句子的平均特征。其中：  
    * duration表示每个音节所占用的平均时长
    * Raw 表示原始数据
    * Z0 表示z-标准化处理。
    * Norm 表示MinMax归一化处理
    * NormZ0 表示先进行归一化后又进行标准化处理
    * Diff 表示经标准化处理后，与整个句子的平均值的差
    * MFCC 的初始特征为40*n维矩阵。目前的处理是对每一维特征，先z-标准化，再归一化，
    最后取n帧的平均值，从而得到40维的向量。Avg MFCC 与 NormZ0 MFCC 中存储的即为该向量。
    Diff MFCC中存储的是后者相对于前者的相对差
