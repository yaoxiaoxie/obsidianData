### torch的配置

使用此命令检测是否安装支持Gpu版本的torch
```
python -c "import torch; print(torch.__version__)"
```

结果：
```
(.venv) PS C:\25\1231\yolo05> python -c "import torch; print('CUDA可用:', torch.cuda.is_available(), 'PyTorch版本:', torch.__version__)" CUDA可用: False PyTorch版本: 2.9.1+cpuPS

C:\25\1231\yolo0101> python -c "import torch; print('CUDA可用:', torch.cuda.is_available(), 'PyTorch版本:', torch.__version__)" CUDA可用: True PyTorch版本: 2.8.0.dev20250414+cu128
```

解决方法：在项目中安装CUDA版PyTorch

```
# 在yolo项目中激活虚拟环境后
pip uninstall torch torchvision torchaudio
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
# 或者根据你的CUDA版本选择cu121/cu124等
```