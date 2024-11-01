# 刷写系统
1. 准备ubuntu18系统
2. 安装 sdk manage
3. 使用杜邦线短接 3(FC REC) 和 4(GND)针脚
4. 使用USB链接电脑，并上电
5. 将USB映射(物理机不用)给ubuntu
6. 无脑下一步即可

# 可以链接WIFI
注意校园网可以ping到，可以同时连校园网后，使用ssh链接jetson nano

# 进入系统后的准备工作
1. sudo apt update
2. sudo apt install python3-pip
3. sudo pip3 install -U pip
4. sudo pip3 install -U jetson-stats
5. sudo apt install nvidia-jetpack

# 检查环境安装是否正常
` sudo jtop `
主要检查cuda等环境是否安装正确。

# 关于jetson nano的python相关环境
```shell
安装一些前置库
sudo apt install libopenblas-dev zlib1g-dev libjpeg-dev libpng-dev
sudo apt install python3.8-venv
这里的虚拟环境的创建不能使用sudo
python3 -m venv facevenv
激活环境
source ./facevenv/bin/activate

pip install 下载好的torch
对应torch下载地址如下
https://forums.developer.nvidia.com/t/pytorch-for-jetson/72048
以及对应版本的vision
https://github.com/pytorch/vision

安装对应版本vision
git clone -b release/0.15 https://github.com/pytorch/vision torchvision
cd torchvision
python setup.py install

检查缺失依赖并补全
pip check

下面代码输出版本号则没问题
python3 -c "import torch; print(torch.__version__)"
python3 -c "import torchvision; print(torchvision.__version__)"

 
```

# 对于jetson nano 的一些库版本
建议使用2.1.0版本的torch,torchvision为 1.15.2
对应faceFacenet版本为2.5.2
