### torch的配置

使用此命令检测是否安装支持Gpu版本的torch
```bash
python -c "import torch; print(torch.__version__)"
```

结果：
```bash
(.venv) PS C:\25\1231\yolo05> python -c "import torch; print('CUDA可用:', torch.cuda.is_available(), 'PyTorch版本:', torch.__version__)" CUDA可用: False PyTorch版本: 2.9.1+cpuPS

C:\25\1231\yolo0101> python -c "import torch; print('CUDA可用:', torch.cuda.is_available(), 'PyTorch版本:', torch.__version__)" CUDA可用: True PyTorch版本: 2.8.0.dev20250414+cu128
```

解决方法：在项目中安装CUDA版PyTorch

```bash
# 在yolo项目中激活虚拟环境后
pip uninstall torch torchvision torchaudio
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
# 或者根据你的CUDA版本选择cu121/cu124等
```

### ultralytics的配置

本地下载ultralytics方便进行模型修改
```bash
git clone https://github.com/ultralytics/ultralytics.git
```

打开Windows 文件资源管理器，把刚才下载下来的那个==ultralytics==文件夹，手动重命名为 ==yolo_source==。

回到 PyCharm 的终端：
```bash
# 先卸载可能存在的残留
pip uninstall ultralytics -y

# 进入改名后的文件夹 
cd yolo_source  
# 重新安装 
pip install -e .
```

验证
```bash
运行测试脚本：
cd ..
python testEnv/testEnv/test_ultralytics.py
```


