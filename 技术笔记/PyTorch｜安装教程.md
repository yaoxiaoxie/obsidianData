
# PyTorch｜安装教程

PyTorch 是一个基于 Python 的开源机器学习框架，它由 Facebook 开发并维护。PyTorch 的主要特点是其动态计算图机制，以及提供了丰富的神经网络层和工具库。

# PyTorch

PyTorch 是一个基于 Python 的开源机器学习框架，它由 Facebook 开发并维护。PyTorch 的主要特点是其动态计算图机制，以及提供了丰富的神经网络层和工具库。

与静态计算图的框架（例如 TensorFlow）不同，PyTorch 的动态计算图机制可以在运行时动态构建计算图，这使得开发者可以更方便地使用 Python 编程语言进行实验和调试。此外，PyTorch 还支持 GPU 计算，可以快速地处理大规模数据和深度神经网络。

PyTorch 提供了许多用于神经网络开发的工具库和接口，包括：

- `torch.nn`：提供了各种神经网络层和损失函数。
- `torch.optim`：提供了各种优化器，例如梯度下降和自适应优化器。
- `torch.utils.data`：提供了数据处理和加载工具。
- `torchvision`：提供了图像处理和计算机视觉工具。

此外，PyTorch 还支持许多高级特性，例如自动微分、分布式训练、动态网络、量化和混合精度等。这些特性使得 PyTorch 成为一个非常灵活和可扩展的机器学习框架，广泛应用于研究和工业界。

## 前置知识

PyTorch官网：[https://pytorch.org/](https://pytorch.org/)

在安装PyTorch之前，我们需要先安装CUDA。但是在安装CUDA之前呢，我们需要先知道自己的电脑支持哪个版本的CUDA。

想查看自己应该安装的CUDA版本，需要先在CMD命令行中执行如下命令：

```
nvidia-smi
```

在执行完上述命令之后，将会自动弹出如下内容，我们只需要查看第一行中的`CUDA Version: 12.1`这一部分，这代表[本站](https://openai.wiki/ "本站")需要下载CUDA版本为12.1的安装程序。

```
+---------------------------------------------------------------------------------------+

| NVIDIA-SMI 531.41 Driver Version: 531.41 CUDA Version: 12.1 |

|-----------------------------------------+----------------------+----------------------+

| GPU Name TCC/WDDM | Bus-Id Disp.A | Volatile Uncorr. ECC |

| Fan Temp Perf Pwr:Usage/Cap| Memory-Usage | GPU-Util Compute M. |

| | | MIG M. |

|=========================================+======================+======================|

| 0 NVIDIA GeForce RTX 2080 Ti WDDM | 00000000:01:00.0 On | N/A |

| 25% 32C P8 7W / 260W| 903MiB / 11264MiB | 4% Default |

| | | N/A |

+-----------------------------------------+----------------------+----------------------+

+---------------------------------------------------------------------------------------+

| Processes: |

| GPU GI CI PID Type Process name GPU Memory |

| ID ID Usage |

|=======================================================================================|

| 0 N/A N/A 5900 C+G ...nt.CBS_cw5n1h2txyewy\SearchHost.exe N/A |

| 0 N/A N/A 7172 C+G ...crosoft\Edge\Application\msedge.exe N/A |

| 0 N/A N/A 8768 C+G ...__8wekyb3d8bbwe\WindowsTerminal.exe N/A |

| 0 N/A N/A 10764 C+G ...5n1h2txyewy\ShellExperienceHost.exe N/A |

| 0 N/A N/A 11748 C+G C:\Windows\explorer.exe N/A |

| 0 N/A N/A 14776 C+G ...Programs\Microsoft VS Code\Code.exe N/A |

| 0 N/A N/A 15816 C+G ...CBS_cw5n1h2txyewy\TextInputHost.exe N/A |

| 0 N/A N/A 17028 C+G ...on\111.0.1661.54\msedgewebview2.exe N/A |

| 0 N/A N/A 18224 C+G ...inaries\Win64\EpicGamesLauncher.exe N/A |

| 0 N/A N/A 18836 C+G ...ne\Binaries\Win64\EpicWebHelper.exe N/A |

| 0 N/A N/A 19300 C+G ...__8wekyb3d8bbwe\WindowsTerminal.exe N/A |

+---------------------------------------------------------------------------------------+
```

其实上面写着`CUDA Version: 12.1`，并不代表一定要下载12.1版本，这里的12.1代表的是您电脑支持的CUDA最高版本。CUDA是向下兼容的，所以您安装12.0或者11.8等低于12.1的版本都是可以的。

因为要顾及PyTorch的兼容性问题，所以我们不能直接下载12.1版本的CUDA，我们需要先打开PyTorch官网看一下目前的兼容性问题。

本站因为想在Windows的Conda中安装PyTorch，所以依次选择`Stable`->`Windows`->`Conda`->`Python`->`CUDA 11.8`。其中的`Stable`其实就是稳定版的PyTorch，`Preview(Nightly)`是每天更新的预览测试版，可能存在未知Bug，所以最好选择Stable版本。

说到这里可以看到，我们要安装的东西已经很明朗啦。因为PyTorch官网中没有12.1版本CUDA的支持，所以我们应该安装11.8版本的CUDA。

## CUDA

### 下载

_提示：CUDA部分的教程配图是其它版本的，但不影响您的观看学习和安装，所有步骤完全一致。_

CUDA官网：[CUDA Toolkit Archive | NVIDIA Developer](https://developer.nvidia.com/cuda-toolkit-archive)

打开CUDA官网，在其中找到适合自己的版本，因为本站已经知道自己所需要的CUDA版本是11.8.0，所以直接点击`CUDA Toolkit 11.8.0`。

在点击适合自己的CUDA版本之后，将会看到如下界面，我们按照自身的系统情况选择下载即可：

### 安装

双击运行下载的CUDA EXE安装包文件，然后将会看到如下界面，然后根据图片上方的文字提示操作即可。

1.建议使用默认路径，不要自行修改，直接点击**OK**按钮即可。

2.点击同意并继续

3.直接默认精简安装即可，点击**下一步**。

4.勾选红框的内容，然后点击**Next**按钮。

5.点击**下一步**按钮。

6.点击**关闭**按钮。

安装完成，可见关于CUDA的安装还是非常简单的，唯一需要注意的是，一定要确定所下载的CUDA安装包与自己的系统所匹配即可。

## PyTorch

### Python

如果您希望在电脑中的Python内直接安装PyTorch，那么在下面依次选择`Stable`->`Windows`->`Pip`->`Python`->`CUDA 11.8`，然后复制最后一行的内容。

打开CMD窗口，在窗口内粘贴如下的内容，等待片刻即可完成PyTorch的安装。

```
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
```

### Conda

如果您还不知道什么是Conda，或者没有安装过Conda，请先阅读下面这篇文章。

如果您希望在电脑中的Conda安装PyTorch，那么在下面依次选择`Stable`->`Windows`->`Conda`->`Python`->`CUDA 11.8`，然后复制最后一行的内容。

打开CMD窗口，先激活想要使用的Conda环境，然后在粘贴已经复制好的命令行，等待片刻即可。

```
conda install pytorch torchvision torchaudio pytorch-cuda=11.8 -c pytorch -c nvidia
```

安装过程日志如下，仅供参考：

```
openai.wiki>conda install pytorch torchvision torchaudio pytorch-cuda=11.8 -c pytorch -c nvidia

Collecting package metadata (current_repodata.json): done

Solving environment: failed with initial frozen solve. Retrying with flexible solve.

Solving environment: failed with repodata from current_repodata.json, will retry with next repodata source.

Collecting package metadata (repodata.json): done

Solving environment: done

==> WARNING: A newer version of conda exists. <==

current version: 23.1.0

latest version: 23.3.1

Please update conda by running

$ conda update -n base -c defaults conda

Or to minimize the number of packages updated during conda update use

conda install conda=23.3.1
```

## Package Plan ##

```
environment location: D:\openai.wiki\ChatGLM-6B\MyENV

added / updated specs:

- pytorch

- pytorch-cuda=11.8

- torchaudio

- torchvision

The following packages will be downloaded:

package | build

---------------------------|-----------------

cuda-cccl-12.1.55 | 0 1.2 MB nvidia

cuda-cudart-11.8.89 | 0 1.4 MB nvidia

cuda-cudart-dev-11.8.89 | 0 723 KB nvidia

cuda-cupti-11.8.87 | 0 11.5 MB nvidia

cuda-libraries-11.8.0 | 0 1 KB nvidia

cuda-libraries-dev-11.8.0 | 0 1 KB nvidia

cuda-nvrtc-11.8.89 | 0 72.1 MB nvidia

cuda-nvrtc-dev-11.8.89 | 0 16.1 MB nvidia

cuda-nvtx-11.8.86 | 0 43 KB nvidia

cuda-profiler-api-12.1.55 | 0 18 KB nvidia

cuda-runtime-11.8.0 | 0 1 KB nvidia

filelock-3.9.0 | py310haa95532_0 19 KB

jinja2-3.1.2 | py310haa95532_0 215 KB

libcublas-11.11.3.6 | 0 33 KB nvidia

libcublas-dev-11.11.3.6 | 0 375.9 MB nvidia

libcufft-10.9.0.58 | 0 6 KB nvidia

libcufft-dev-10.9.0.58 | 0 144.6 MB nvidia

libcurand-10.3.2.56 | 0 3 KB nvidia

libcurand-dev-10.3.2.56 | 0 50.0 MB nvidia

libcusolver-11.4.1.48 | 0 29 KB nvidia

libcusolver-dev-11.4.1.48 | 0 94.1 MB nvidia

libcusparse-11.7.5.86 | 0 13 KB nvidia

libcusparse-dev-11.7.5.86 | 0 175.7 MB nvidia

libnpp-11.8.0.86 | 0 294 KB nvidia

libnpp-dev-11.8.0.86 | 0 143.2 MB nvidia

libnvjpeg-11.9.0.86 | 0 4 KB nvidia

libnvjpeg-dev-11.9.0.86 | 0 1.9 MB nvidia

markupsafe-2.1.1 | py310h2bbff1b_0 26 KB

mpmath-1.2.1 | py310haa95532_0 779 KB

networkx-2.8.4 | py310haa95532_1 2.6 MB

numpy-1.23.5 | py310h60c9a35_0 11 KB

numpy-base-1.23.5 | py310h04254f7_0 6.0 MB

pip-23.0.1 | py310haa95532_0 2.8 MB

pytorch-2.0.0 |py3.10_cuda11.8_cudnn8_0 1.35 GB pytorch

pytorch-cuda-11.8 | h24eeafa_3 7 KB pytorch

sympy-1.11.1 | py310haa95532_0 11.8 MB

torchaudio-2.0.0 | py310_cu118 5.7 MB pytorch

torchvision-0.15.0 | py310_cu118 7.8 MB pytorch

urllib3-1.26.15 | py310haa95532_0 195 KB

zstd-1.5.4 | hd43e919_0 683 KB

------------------------------------------------------------

Total: 2.45 GB

The following NEW packages will be INSTALLED:

blas pkgs/main/win-64::blas-1.0-mkl

brotlipy pkgs/main/win-64::brotlipy-0.7.0-py310h2bbff1b_1002

bzip2 pkgs/main/win-64::bzip2-1.0.8-he774522_0

ca-certificates pkgs/main/win-64::ca-certificates-2023.01.10-haa95532_0

certifi pkgs/main/win-64::certifi-2022.12.7-py310haa95532_0

cffi pkgs/main/win-64::cffi-1.15.1-py310h2bbff1b_3

charset-normalizer pkgs/main/noarch::charset-normalizer-2.0.4-pyhd3eb1b0_0

cryptography pkgs/main/win-64::cryptography-39.0.1-py310h21b164f_0

cuda-cccl nvidia/win-64::cuda-cccl-12.1.55-0

cuda-cudart nvidia/win-64::cuda-cudart-11.8.89-0

cuda-cudart-dev nvidia/win-64::cuda-cudart-dev-11.8.89-0

cuda-cupti nvidia/win-64::cuda-cupti-11.8.87-0

cuda-libraries nvidia/win-64::cuda-libraries-11.8.0-0

cuda-libraries-dev nvidia/win-64::cuda-libraries-dev-11.8.0-0

cuda-nvrtc nvidia/win-64::cuda-nvrtc-11.8.89-0

cuda-nvrtc-dev nvidia/win-64::cuda-nvrtc-dev-11.8.89-0

cuda-nvtx nvidia/win-64::cuda-nvtx-11.8.86-0

cuda-profiler-api nvidia/win-64::cuda-profiler-api-12.1.55-0

cuda-runtime nvidia/win-64::cuda-runtime-11.8.0-0

filelock pkgs/main/win-64::filelock-3.9.0-py310haa95532_0

flit-core pkgs/main/win-64::flit-core-3.8.0-py310haa95532_0

freetype pkgs/main/win-64::freetype-2.12.1-ha860e81_0

giflib pkgs/main/win-64::giflib-5.2.1-h8cc25b3_3

idna pkgs/main/win-64::idna-3.4-py310haa95532_0

intel-openmp pkgs/main/win-64::intel-openmp-2021.4.0-haa95532_3556

jinja2 pkgs/main/win-64::jinja2-3.1.2-py310haa95532_0

jpeg pkgs/main/win-64::jpeg-9e-h2bbff1b_1

lerc pkgs/main/win-64::lerc-3.0-hd77b12b_0

libcublas nvidia/win-64::libcublas-11.11.3.6-0

libcublas-dev nvidia/win-64::libcublas-dev-11.11.3.6-0

libcufft nvidia/win-64::libcufft-10.9.0.58-0

libcufft-dev nvidia/win-64::libcufft-dev-10.9.0.58-0

libcurand nvidia/win-64::libcurand-10.3.2.56-0

libcurand-dev nvidia/win-64::libcurand-dev-10.3.2.56-0

libcusolver nvidia/win-64::libcusolver-11.4.1.48-0

libcusolver-dev nvidia/win-64::libcusolver-dev-11.4.1.48-0

libcusparse nvidia/win-64::libcusparse-11.7.5.86-0

libcusparse-dev nvidia/win-64::libcusparse-dev-11.7.5.86-0

libdeflate pkgs/main/win-64::libdeflate-1.17-h2bbff1b_0

libffi pkgs/main/win-64::libffi-3.4.2-hd77b12b_6

libnpp nvidia/win-64::libnpp-11.8.0.86-0

libnpp-dev nvidia/win-64::libnpp-dev-11.8.0.86-0

libnvjpeg nvidia/win-64::libnvjpeg-11.9.0.86-0

libnvjpeg-dev nvidia/win-64::libnvjpeg-dev-11.9.0.86-0

libpng pkgs/main/win-64::libpng-1.6.39-h8cc25b3_0

libtiff pkgs/main/win-64::libtiff-4.5.0-h6c2663c_2

libuv pkgs/main/win-64::libuv-1.44.2-h2bbff1b_0

libwebp pkgs/main/win-64::libwebp-1.2.4-hbc33d0d_1

libwebp-base pkgs/main/win-64::libwebp-base-1.2.4-h2bbff1b_1

lz4-c pkgs/main/win-64::lz4-c-1.9.4-h2bbff1b_0

markupsafe pkgs/main/win-64::markupsafe-2.1.1-py310h2bbff1b_0

mkl pkgs/main/win-64::mkl-2021.4.0-haa95532_640

mkl-service pkgs/main/win-64::mkl-service-2.4.0-py310h2bbff1b_0

mkl_fft pkgs/main/win-64::mkl_fft-1.3.1-py310ha0764ea_0

mkl_random pkgs/main/win-64::mkl_random-1.2.2-py310h4ed8f06_0

mpmath pkgs/main/win-64::mpmath-1.2.1-py310haa95532_0

networkx pkgs/main/win-64::networkx-2.8.4-py310haa95532_1

numpy pkgs/main/win-64::numpy-1.23.5-py310h60c9a35_0

numpy-base pkgs/main/win-64::numpy-base-1.23.5-py310h04254f7_0

openssl pkgs/main/win-64::openssl-1.1.1t-h2bbff1b_0

pillow pkgs/main/win-64::pillow-9.4.0-py310hd77b12b_0

pip pkgs/main/win-64::pip-23.0.1-py310haa95532_0

pycparser pkgs/main/noarch::pycparser-2.21-pyhd3eb1b0_0

pyopenssl pkgs/main/win-64::pyopenssl-23.0.0-py310haa95532_0

pysocks pkgs/main/win-64::pysocks-1.7.1-py310haa95532_0

python pkgs/main/win-64::python-3.10.10-h966fe2a_2

pytorch pytorch/win-64::pytorch-2.0.0-py3.10_cuda11.8_cudnn8_0

pytorch-cuda pytorch/win-64::pytorch-cuda-11.8-h24eeafa_3

pytorch-mutex pytorch/noarch::pytorch-mutex-1.0-cuda

requests pkgs/main/win-64::requests-2.28.1-py310haa95532_1

setuptools pkgs/main/win-64::setuptools-65.6.3-py310haa95532_0

six pkgs/main/noarch::six-1.16.0-pyhd3eb1b0_1

sqlite pkgs/main/win-64::sqlite-3.41.1-h2bbff1b_0

sympy pkgs/main/win-64::sympy-1.11.1-py310haa95532_0

tk pkgs/main/win-64::tk-8.6.12-h2bbff1b_0

torchaudio pytorch/win-64::torchaudio-2.0.0-py310_cu118

torchvision pytorch/win-64::torchvision-0.15.0-py310_cu118

typing_extensions pkgs/main/win-64::typing_extensions-4.4.0-py310haa95532_0

tzdata pkgs/main/noarch::tzdata-2022g-h04d1e81_0

urllib3 pkgs/main/win-64::urllib3-1.26.15-py310haa95532_0

vc pkgs/main/win-64::vc-14.2-h21ff451_1

vs2015_runtime pkgs/main/win-64::vs2015_runtime-14.27.29016-h5e58377_2

wheel pkgs/main/win-64::wheel-0.38.4-py310haa95532_0

win_inet_pton pkgs/main/win-64::win_inet_pton-1.1.0-py310haa95532_0

wincertstore pkgs/main/win-64::wincertstore-0.2-py310haa95532_2

xz pkgs/main/win-64::xz-5.2.10-h8cc25b3_1

zlib pkgs/main/win-64::zlib-1.2.13-h8cc25b3_0

zstd pkgs/main/win-64::zstd-1.5.4-hd43e919_0

Proceed ([y]/n)?

Downloading and Extracting Packages

pytorch-cuda-11.8 | 7 KB | ############################################################################ | 100%

jinja2-3.1.2 | 215 KB | ############################################################################ | 100%

pytorch-2.0.0 | 1.35 GB | ############################################################################ | 100%

libnpp-dev-11.8.0.86 | 143.2 MB | ############################################################################ | 100%

libnvjpeg-dev-11.9.0 | 1.9 MB | ############################################################################ | 100%

libcublas-11.11.3.6 | 33 KB | ############################################################################ | 100%

cuda-nvrtc-dev-11.8. | 16.1 MB | ############################################################################ | 100%

pip-23.0.1 | 2.8 MB | ############################################################################ | 100%

markupsafe-2.1.1 | 26 KB | ############################################################################ | 100%

cuda-nvtx-11.8.86 | 43 KB | ############################################################################ | 100%

numpy-1.23.5 | 11 KB | ############################################################################ | 100%

numpy-base-1.23.5 | 6.0 MB | ############################################################################ | 100%

cuda-cudart-11.8.89 | 1.4 MB | ############################################################################ | 100%

libcusolver-11.4.1.4 | 29 KB | ############################################################################ | 100%

cuda-nvrtc-11.8.89 | 72.1 MB | ############################################################################ | 100%

cuda-cudart-dev-11.8 | 723 KB | ############################################################################ | 100%

libnvjpeg-11.9.0.86 | 4 KB | ############################################################################ | 100%

libcublas-dev-11.11. | 375.9 MB | ############################################################################ | 100%

cuda-profiler-api-12 | 18 KB | ############################################################################ | 100%

filelock-3.9.0 | 19 KB | ############################################################################ | 100%

torchvision-0.15.0 | 7.8 MB | ############################################################################ | 100%

urllib3-1.26.15 | 195 KB | ############################################################################ | 100%

libcufft-10.9.0.58 | 6 KB | ############################################################################ | 100%

torchaudio-2.0.0 | 5.7 MB | ############################################################################ | 100%

cuda-runtime-11.8.0 | 1 KB | ############################################################################ | 100%

networkx-2.8.4 | 2.6 MB | ############################################################################ | 100%

zstd-1.5.4 | 683 KB | ############################################################################ | 100%

libnpp-11.8.0.86 | 294 KB | ############################################################################ | 100%

... (more hidden) ...

Downloading and Extracting Packages

Preparing transaction: done

Verifying transaction: done

Executing transaction: done

done
```
