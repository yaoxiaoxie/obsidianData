
# cuDNN｜Windows安装教程


cuDNN（CUDA Deep Neural Network library）是由NVIDIA开发的一个加速深度神经网络的库，为深度学习应用提供GPU加速的功能，cuDNN主要为深度神经网络提升速度。

# cuDNN｜Windows系统安装教程

cuDNN（CUDA Deep Neural Network library）是由 NVIDIA 开发的一个加速深度神经网络的库，它为深度学习应用提供了 GPU 加速的功能。cuDNN 主要提供了深度神经网络的基本操作，例如卷积、池化、归一化等等，这些操作都可以在 GPU 上进行加速。

cuDNN 使用 CUDA 技术来实现 GPU 加速，这使得它能够利用 NVIDIA 的 GPU 硬件来加速深度神经网络的训练和推理。cuDNN 能够充分利用 GPU 的并行计算能力，大大加速了深度神经网络的训练和推理速度，从而提高了深度学习应用的效率。

cuDNN 的优势在于它可以高效地利用 NVIDIA 的 GPU 硬件资源，实现深度神经网络的快速训练和推理。同时，cuDNN 还提供了丰富的 API 和工具，方便开发者和研究人员使用和调试深度学习应用。

## 前置条件

在安装cuDNN之前，我们需要先安装Windows版本的CUDA。

## Nvidia｜登陆

如果您已有Nvidia的开发者账号，那么直接【[点击此处](https://developer.nvidia.com/login)】后，在弹出的网页内输入账号登陆即可。

## Nvidia｜注册

关于Nvidia的开发者注册还是挺麻烦的，毕竟需要填写一些必要的资料。

我们【[点击此处](https://developer.nvidia.com/login)】前往Nvidia开发者账号的注册页面，打开之后的界面如下，填写自己的邮箱后，点击`Next`按钮。

此时我们需要输入`密码`，以及再次输入`确认密码`，然后进行验证码码确认，最后点击`创建账户`按钮即可。

在你点击`创建账户`之后，将会在该邮箱内收到一条邮件，该邮件为注册确认邮件，我们在邮箱内点击`验证电子邮箱地址`按钮即可。

此时将会自动弹出一个网页，用以提示你邮箱已被成功验证。

我们回到刚刚的注册界面，不勾选任何内容，直接点击`提交`按钮。

此时将会让你填写一些相关信息，其实这些内容都是可以随意填写的，因为这并不会进行任何官方验证。

值得注意的是`加入NVIDIA开发人员计划，访问下载内容（如cuDNN）、操作视频等。`选项，必须要勾选，否则注册后也无法下载cuDNN。

在我们点击提交按钮之后，将会自动重定向至官方，这代表我们已经成功注册并登陆啦。

## cuDNN｜下载

在进行此步骤之前，请确保你已经完成了Nvidia账号的注册或登陆，否则无法下载下载。

我们【[点击此处](https://developer.nvidia.com/rdp/cudnn-archive)】前往cuDNN的下载页面，打开之后将会看到如下所示的内容，我们需要选择适合自己的版本。

> 如果你不知道自己的CUDA版本是多少，或者不知道CUDA是什么，请【[点击此处](https://openai.wiki/cuda-windows-install.html)】前往查看关于CUDA的内容。

我们从上图中可以看到，每一个版本后面都会有一个版本号，我们单独拿出一个来讲解。

第一个下载地址的完整名称为【Download `cuDNN v8.9.2` `(June 1st, 2023`), for `CUDA 12.x`】，对我们有用的其实就两个部分，第一个是cuDNN的版本，其次是CUDA的版本。[本站](https://openai.wiki/ "本站")的CUDA版本为12.1，那么就可以下载`CUDA 12.x`的版本，因为这其中的`x`是通用的意思。

所以，如果你的CUDA版本是`11.x`，那么就下载第`2`个即可，其它同理。

综上所述，适合站长的版本为`Download cuDNN v8.9.2 (June 1st, 2023), for CUDA 12.x`，所以站长点击该按钮。

点击该按钮之后将会自动展开该选项，这将会包含所有相关系统的cuDNN下载地址，我们选择`Local Installer for Windows (Zip)`按钮即可自动开始下载，大小一般在700MB左右。
## cuDNN配置

### 安装文件

我们首先需要知道自己的CUDA安装位置，如果你不知道自己的CUDA安装在什么位置，可以在CMD中执行如下内容。

```
where nvcc
```

执行完上面的命令之后，将会自动获取CUDA的安装目录。

```
C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1\bin\nvcc.exe
```

我们只要到版本号这一目录即可，也就是`C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1`，后面的部分省略。

注意：每个人所获取到的版本可能不同，本站的是CUDA是12.1版本，你的可能是v11.8等，所以不要照抄，一定要自行获取。

此时将我们刚刚下载好的压缩包文件解压，然后将文件内的所有文件移动是我们获取到的目录内。（如果提示已存在，直接选择覆盖所有文件即可。）

### 环境变量

我们完成上面的所有步骤之后，还需要将相关目录添加至系统环境变量内。

其实我们在安装CUDA时，应该在默认情况下已经添加了两个关于CUDA的环境变量，分为如下：

```
C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\CUDA版本\bin

C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\CUDA版本\libnvvp
```

我们还需要添加额外两个，我们刚刚已经获取到了CUDA的安装目录，另外两个文件夹也位于该目录内，分别是`include`文件夹和`lib`文件夹。

我们可以通过CMD执行如下命令来快速将其添加至系统的环境变量内，但需要注意以下情况。

**注意：必须是**`**以管理员身份方式运行的CMD**`**窗口，不然必出错，而且会导致你本身的系统环境错乱。**

**注意：必须将下面的代码中**`**CUDA版本号**`**替换为你自己的版本，否则成功被添加后也会无效。**

```
setx PATH "%PATH%;%ProgramFiles%\NVIDIA GPU Computing Toolkit\CUDA\CUDA版本号\include"
```

```
setx PATH "%PATH%;%ProgramFiles%\NVIDIA GPU Computing Toolkit\CUDA\CUDA版本号\lib"
```

### 验证安装

我们需要先将CMD切换至CUDA目录，注意替换`CUDA版本号`为你自己的版本。

```
cd /d %ProgramFiles%\NVIDIA GPU Computing Toolkit\CUDA\CUDA版本号\extras\demo_suite
```

然后我们执行如下命令：

```
bandwidthTest.exe
```

返回以下内容则代表配置成功：

```
C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.8\extras\demo_suite>bandwidthTest.exe

[CUDA Bandwidth Test] - Starting...

Running on...

Device 0: NVIDIA GeForce RTX 2080 Ti

Quick Mode

Host to Device Bandwidth, 1 Device(s)

PINNED Memory Transfers

Transfer Size (Bytes) Bandwidth(MB/s)

33554432 11692.6

Device to Host Bandwidth, 1 Device(s)

PINNED Memory Transfers

Transfer Size (Bytes) Bandwidth(MB/s)

33554432 10730.4

Device to Device Bandwidth, 1 Device(s)

PINNED Memory Transfers

Transfer Size (Bytes) Bandwidth(MB/s)

33554432 465465.1

Result = PASS

NOTE: The CUDA Samples are not meant for performance measurements. Results may vary when GPU Boost is enabled.
```

然后我们执行如下命令：

```
deviceQuery.exe
```

返回以下内容则代表配置成功：

```
C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.8\extras\demo_suite>deviceQuery.exe

deviceQuery.exe Starting...

CUDA Device Query (Runtime API) version (CUDART static linking)

Detected 1 CUDA Capable device(s)

Device 0: "NVIDIA GeForce RTX 2080 Ti"

CUDA Driver Version / Runtime Version 12.1 / 11.8

CUDA Capability Major/Minor version number: 7.5

Total amount of global memory: 11264 MBytes (11810832384 bytes)

(68) Multiprocessors, ( 64) CUDA Cores/MP: 4352 CUDA Cores

GPU Max Clock rate: 1650 MHz (1.65 GHz)

Memory Clock rate: 7000 Mhz

Memory Bus Width: 352-bit

L2 Cache Size: 5767168 bytes

Maximum Texture Dimension Size (x,y,z) 1D=(131072), 2D=(131072, 65536), 3D=(16384, 16384, 16384)

Maximum Layered 1D Texture Size, (num) layers 1D=(32768), 2048 layers

Maximum Layered 2D Texture Size, (num) layers 2D=(32768, 32768), 2048 layers

Total amount of constant memory: zu bytes

Total amount of shared memory per block: zu bytes

Total number of registers available per block: 65536

Warp size: 32

Maximum number of threads per multiprocessor: 1024

Maximum number of threads per block: 1024

Max dimension size of a thread block (x,y,z): (1024, 1024, 64)

Max dimension size of a grid size (x,y,z): (2147483647, 65535, 65535)

Maximum memory pitch: zu bytes

Texture alignment: zu bytes

Concurrent copy and kernel execution: Yes with 2 copy engine(s)

Run time limit on kernels: Yes

Integrated GPU sharing Host Memory: No

Support host page-locked memory mapping: Yes

Alignment requirement for Surfaces: Yes

Device has ECC support: Disabled

CUDA Device Driver Mode (TCC or WDDM): WDDM (Windows Display Driver Model)

Device supports Unified Addressing (UVA): Yes

Device supports Compute Preemption: Yes

Supports Cooperative Kernel Launch: Yes

Supports MultiDevice Co-op Kernel Launch: No

Device PCI Domain ID / Bus ID / location ID: 0 / 1 / 0

Compute Mode:

< Default (multiple host threads can use ::cudaSetDevice() with device simultaneously) >

deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 12.1, CUDA Runtime Version = 11.8, NumDevs = 1, Device0 = NVIDIA GeForce RTX 2080 Ti

Result = PASS
```
