# [CUDA]｜Windows系统安装教程全攻略

## 安装检测

在安装之前先打开CMD窗口，粘贴如下命令，查看您的系统中是否已安装CUDA。

```
nvcc -V
```

如果您的电脑中已安装过CUDA，将会看到如下提示：

```
nvcc: NVIDIA (R) Cuda compiler driver

Copyright (c) 2005-2023 NVIDIA Corporation

Built on Wed_Feb__8_05:53:42_Coordinated_Universal_Time_2023

Cuda compilation tools, release 12.1, V12.1.66

Build cuda_12.1.r12.1/compiler.32415258_0
```

如果您的电脑中未安装过CUDA，将会看到如下提示：

```
'nvcc' 不是内部或外部命令，也不是可运行的程序或批处理文件。
```

## 适配下载

在安装CUDA之前，我们应该先知道自己的电脑应该安装哪个版本的CUDA，因为新旧版本兼容性不同。

想查看自己应该安装的CUDA版本，需要先在CMD命令行中执行如下命令：

```
nvidia-smi
```

在执行完上述命令之后，将会自动弹出如下内容，我们只需要查看第一行中的`CUDA Version: 12.1`这一部分，这代表[本站]需要下载CUDA版本为12.1的安装程序。

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

## 下载地址

CUDA官网：[CUDA Toolkit Archive | NVIDIA Developer](https://developer.nvidia.com/cuda-toolkit-archive)

打开CUDA官网，在其中找到适合自己的版本，因为本站已经知道自己所需要的CUDA版本是12.1.0，所以直接点击`CUDA Toolkit 12.1.0`。

在点击适合自己的CUDA版本之后，将会看到如下界面，我们按照自身的系统情况选择下载即可：

![[技术笔记/_resources/CUDA｜Windows安装教程/cab3ccb7c59c9b49a46498c6afe19ca1_MD5.webp]]

## 安装教程

双击运行下载的CUDA EXE安装包文件，然后将会看到如下界面，然后根据图片上方的文字提示操作即可。

1.建议使用默认路径，不要自行修改，直接点击**OK**按钮即可。

![[技术笔记/_resources/CUDA｜Windows安装教程/2e1eb9d01e5b9bd6937bb28476b88979_MD5.webp]]

2.点击同意并继续

![[技术笔记/_resources/CUDA｜Windows安装教程/5b22a26f511cdc9cfd2bf753993eed38_MD5.webp]]

3.直接默认精简安装即可，点击**下一步**。

![[技术笔记/_resources/CUDA｜Windows安装教程/cff0215c3318cbb38660daea64eb1803_MD5.webp]]

4.勾选红框的内容，然后点击**Next**按钮。

![[技术笔记/_resources/CUDA｜Windows安装教程/46dcfcd30cefb8afabf3c67b555e290c_MD5.webp]]

5.点击**下一步**按钮。

![[技术笔记/_resources/CUDA｜Windows安装教程/82705f0131b39dfeba08c6949a851d97_MD5.webp]]

6.点击**关闭**按钮。

![[技术笔记/_resources/CUDA｜Windows安装教程/5aaa7f9068318b9f501b53d7901c72c8_MD5.webp]]

安装完成，可见关于CUDA的安装还是非常简单的，唯一需要注意的是，一定要确定所下载的CUDA安装包与自己的系统所匹配即可。