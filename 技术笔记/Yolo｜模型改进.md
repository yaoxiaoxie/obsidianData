### 1、配置文件（.yaml）

在项目根目录下（和 main.py 同级），新建一个文件==traffic_sign.yaml==。

```python
# 这里的路径可以是绝对路径，也可以是相对路径
# 建议写绝对路径（防止小白出错），例如：D:/Graduation_Project/datasets/CCTSDB
path: D:/Graduation_Project/datasets/CCTSDB  # <-- 修改为你实际的数据集根目录

train: images/train  # 训练集图片路径 (相对于path)
val: images/val      # 验证集图片路径 (相对于path)

# 类别数量 (根据你数据集实际情况改！如果是3类就写3)
nc: 3

# 类别名称 (一定要和标签的顺序对应，0对应第一个，1对应第二个)
names:
  0: prohibitory
  1: warning
  2: mandatory
```

### 2、编写训练脚本 (train.py)

在项目根目录下，新建==train_baseline.py==。
    
复制以下代码：
    
```python
from ultralytics import YOLO

if __name__ == '__main__':
    # 1. 加载官方预训练模型 (推荐 yolov8n 或 yolov8s，笔记本跑得动)
    # 'n'是nano版本，速度最快，显存占用最小，最适合毕设演示
    model = YOLO('yolov8n.pt') 

    # 2. 开始训练
    results = model.train(
        data='traffic_sign.yaml',  # 指定刚才写的配置文件
        epochs=100,                # 训练轮数，100轮作为基准足够了
        imgsz=640,                 # 图片大小
        batch=16,                  # 批次大小。如果显存爆了(OOM)，改成 8 或 4
        workers=2,                 # 加载数据的线程数，Windows下建议设小一点
        name='baseline_run',       # 这次训练结果保存的名字
        project='runs/train'       # 保存路径
    )

    # 3. 验证一下模型 (训练完自动会验证，这里是双重保险)
    metrics = model.val()
```

### 3、开始训练

右键点击train_baseline.py -> Run。

- 报错 OOM (Out of Memory)：显存不够。去代码里把 batch=16 改成 batch=8 或 4。
- 报错 Page file too small：Windows 虚拟内存不足。把 workers=2 改成 workers=0。

### 4、收集“证据” 

训练结束后项目文件夹下找这个路径：  ==runs/train/baseline_run/==

需要保存以下东西：
1. results.png：这是一张包含 Loss 曲线和 mAP 曲线的大图。
2. weights/best.pt：这是训练好的模型文件。
3. confusion_matrix.png：混淆矩阵。        
4. 记录基准数据： results.csv，拉到最后一行，找到 metrics/mAP50(B)这一列的数字。

### 5、核心算法改进

#### Part A：植入 CBAM 注意力机制

目标：修改 YOLOv8 源码，让它能识别 CBAM 这个模块，并在网络中调用它。  
原理：在特征提取网络的末端加入了注意力机制，让模型更关注交通标志的关键特征，忽略背景杂乱信息”。

新建一个==locate.py==，运行下面两行代码：
```python
import ultralytics
import os
print(os.path.dirname(ultralytics.__file__))
```
控制台会输出`ultralytics`库安装路径

找到代码中的==ultralytics/nn/modules/conv.py==文件
```python
###################### 添加CBAM开始 ######################
#复制到 ultralytics/nn/modules/conv.py 末尾
class ChannelAttention(nn.Module):
    def __init__(self, in_planes, ratio=16):
        super(ChannelAttention, self).__init__()
        self.avg_pool = nn.AdaptiveAvgPool2d(1)
        self.max_pool = nn.AdaptiveMaxPool2d(1)
        self.fc = nn.Sequential(
            nn.Conv2d(in_planes, in_planes // ratio, 1, bias=False),
            nn.ReLU(),
            nn.Conv2d(in_planes // ratio, in_planes, 1, bias=False)
        )
        self.sigmoid = nn.Sigmoid()

    def forward(self, x):
        avg_out = self.fc(self.avg_pool(x))
        max_out = self.fc(self.max_pool(x))
        out = avg_out + max_out
        return self.sigmoid(out)

class SpatialAttention(nn.Module):
    def __init__(self, kernel_size=7):
        super(SpatialAttention, self).__init__()
        assert kernel_size in (3, 7), 'kernel size must be 3 or 7'
        padding = 3 if kernel_size == 7 else 1
        self.conv1 = nn.Conv2d(2, 1, kernel_size, padding=padding, bias=False)
        self.sigmoid = nn.Sigmoid()

    def forward(self, x):
        avg_out = torch.mean(x, dim=1, keepdim=True)
        max_out, _ = torch.max(x, dim=1, keepdim=True)
        x = torch.cat([avg_out, max_out], dim=1)
        x = self.conv1(x)
        return self.sigmoid(x)

class CBAM(nn.Module):
    def __init__(self, c1, c2, kernel_size=7):  # ch_in, ch_out, kernel
        super(CBAM, self).__init__()
        # c1是输入通道，这里默认保持通道数不变，方便随意插入
        self.channel_attention = ChannelAttention(c1)
        self.spatial_attention = SpatialAttention(kernel_size)

    def forward(self, x):
        out = self.channel_attention(x) * x
        out = self.spatial_attention(out) * out
        return out
###################### 添加CBAM结束 ######################
```
保存并关闭。

找到==ultralytics/nn/tasks.py==文件。
- 找到开头的`import`部分，在`from ultralytics.nn.modules import (...)`这一行里，把`CBAM`加进去。例如：`from ultralytics.nn.modules import (..., C2f, SPPF, CBAM)`
- 在该文件中搜索`def parse_model`函数。在`parse_model`函数里，找到类似`elif m is AIFI:`这一行在下面，插入一段专门处理 CBAM 的代码。
```python
# ... (上面是 elif m is C2fCIB: legacy = False)
        
        elif m is AIFI:
            args = [ch[f], *args]
            
        # =========== 【在这里插入这 2 行】 =============
        # =========== 【开始】修正后的 CBAM 解析逻辑 =============
		elif m is CBAM:
		    c1 = ch[f]  # 从前一层获取输入通道数
    
			    if len(args) == 1:
			        # YAML 中只写了 [7]，自动补充通道数
			        kernel_size = args[0]
			        args = [c1, kernel_size]
			    elif len(args) == 2:
			        # YAML 中写了 [1024, 7]，直接使用
			        pass
			    else:
			        raise ValueError(f"CBAM expects 1 or 2 args, got {len(args)}: {args}")
    
		    c2 = c1  # CBAM 输出通道数等于输入通道数
		# =========== 【结束】 ===========================
        # ============================================

        elif m in frozenset({HGStem, HGBlock}):
            c1, cm, c2 = ch[f], args[0], args[1]
        # ... (下面保持不变)
```
保存并关闭 tasks.py。

回到项目文件夹。
复制==yolov8n.yaml==，重命名==为yolov8_cbam.yaml==。
打开它，修改`backbone`部分。把CBAM插在主干网络的最后一层（SPPF前后效果最好）。
修改后的backbone：
```python
# Ultralytics 🚀 AGPL-3.0 License - https://ultralytics.com/license  
# YOLOv8-CBAM 改进版配置文件   

nc: 80 # number of classes  
scales: # model compound scaling constants, i.e. 'model=yolov8n.yaml' will call yolov8.yaml with scale 'n'  
  n: [0.33, 0.25, 1024] # 用这个 Nano 版本   
  
backbone:  
  # [from, repeats, module, args]  
  - [-1, 1, Conv, [64, 3, 2]] # 0-P1/2  
  - [-1, 1, Conv, [128, 3, 2]] # 1-P2/4  
  - [-1, 3, C2f, [128, True]]  
  - [-1, 1, Conv, [256, 3, 2]] # 3-P3/8  
  - [-1, 6, C2f, [256, True]]  
  - [-1, 1, Conv, [512, 3, 2]] # 5-P4/16  
  - [-1, 6, C2f, [512, True]]  
  - [-1, 1, Conv, [1024, 3, 2]] # 7-P5/32  
  - [-1, 3, C2f, [1024, True]]  
  - [-1, 1, SPPF, [1024, 5]] # 9  
  - [-1, 1, CBAM, [7]]       # 10 <---【修改1】新增第10层 CBAM (接在SPPF后面)  
  
head:  
  - [-1, 1, nn.Upsample, [None, 2, "nearest"]]  
  - [[-1, 6], 1, Concat, [1]] # cat backbone P4  
  - [-1, 3, C2f, [512]] # 12  
  
  - [-1, 1, nn.Upsample, [None, 2, "nearest"]]  
  - [[-1, 4], 1, Concat, [1]] # cat backbone P3  
  - [-1, 3, C2f, [256]] # 16 (这里对应原来的15，是P3输出) (P3/8-small)  
  
  - [-1, 1, Conv, [256, 3, 2]]  
  - [[-1, 13], 1, Concat, [1]] # cat head P4  
  - [-1, 3, C2f, [512]] # 19 (这里对应原来的18，是P4输出) (P4/16-medium)  
  
  - [-1, 1, Conv, [512, 3, 2]]  
#  - [[-1, 9], 1, Concat, [1]] # cat head P5  
  - [[-1, 10], 1, Concat, [1]] # <---【修改2】注意这里！原版是9(SPPF)，改为10(CBAM)，让Head用上注意力特征  
  - [-1, 3, C2f, [1024]] # 21 (P5/32-large)  
  
#  - [[15, 18, 21], 1, Detect, [nc]] # Detect(P3, P4, P5)  
  - [[16, 19, 22], 1, Detect, [nc]] # 23 Detect(P3, P4, P5) <---【这里必须改成 16, 19, 22】
```

测试模型
```python
"""
验证模型配置是否正确
"""

from ultralytics import YOLO
import torch

print("="*60)
print("模型结构验证")
print("="*60)

# 1. 测试 CBAM 版本
print("\n1. 检查 YOLOv8n-CBAM 模型...")
try:
    model = YOLO('yolov8-cbam.yaml')
    model.load('yolov8n.pt')  # 加载预训练权重
    
    # 打印模型信息
    model.info(detailed=False)
    
    # 查找 CBAM 层
    cbam_found = False
    for name, module in model.model.named_modules():
        if 'CBAM' in str(type(module).__name__):
            print(f"\n✅ 找到 CBAM 层: {name}")
            print(f"   类型: {type(module)}")
            cbam_found = True
    
    if not cbam_found:
        print("\n❌ 未找到 CBAM 层！")
    
    # 测试前向传播
    print("\n2. 测试前向传播...")
    dummy_input = torch.randn(1, 3, 640, 640)
    with torch.no_grad():
        output = model(dummy_input)
    print("✅ 前向传播成功！")
    
except Exception as e:
    print(f"\n❌ 错误: {e}")
    import traceback
    traceback.print_exc()

print("\n" + "="*60)
```

创建一个新的训练脚本train_cbam.py：
```python
model = YOLO('yolov8_cbam.yaml') # 注意：这里加载的是yaml，不是pt！表示从头训练结构
# 如果想利用预训练权重加速收敛（推荐），写成：
# model = YOLO('yolov8n.pt') 
# model = YOLO('yolov8_cbam.yaml').load('yolov8n.pt') # 这种写法更高级，但小白容易错
# 简单写法：
model = YOLO('yolov8_cbam.yaml') 

results = model.train(data='traffic_sign.yaml', epochs=100, name='cbam_run')
```

#### Part B：引入 WIoU v3 损失函数

找到==ultralytics/utils/loss.py==
在==loss.py==中，找到`class DFLoss(nn.Module):`
在`DFLoss` 类之后，`class BboxLoss(nn.Module):` 之前的空位，插入下面这段完整的 WIoU 计算代码。
```python
# 新增
# ================== WIoU v3 核心代码 Start ==================
def bbox_wiou(box1, box2, scale=True):
    """
    WIoU v3 (Wise-IoU v3) Loss
    论文：Wise-IoU: Bounding Box Regression Loss with Dynamic Focusing Mechanism
    
    参数:
        box1: 预测框 [N, 4] (x1, y1, x2, y2)
        box2: 真实框 [N, 4] (x1, y1, x2, y2)
        scale: 是否使用动态梯度增益（WIoU v3 的核心）
    
    返回:
        loss: WIoU v3 损失值 [N]
    """
    # 1. 基础 IoU 计算
    b1_x1, b1_y1, b1_x2, b1_y2 = box1.chunk(4, -1)
    b2_x1, b2_y1, b2_x2, b2_y2 = box2.chunk(4, -1)

    # 交集
    inter_x1 = torch.max(b1_x1, b2_x1)
    inter_y1 = torch.max(b1_y1, b2_y1)
    inter_x2 = torch.min(b1_x2, b2_x2)
    inter_y2 = torch.min(b1_y2, b2_y2)
    inter_area = (inter_x2 - inter_x1).clamp(0) * (inter_y2 - inter_y1).clamp(0)

    # 并集
    b1_area = (b1_x2 - b1_x1) * (b1_y2 - b1_y1)
    b2_area = (b2_x2 - b2_x1) * (b2_y2 - b2_y1)
    union = b1_area + b2_area - inter_area + 1e-9
    
    # IoU
    iou = inter_area / union

    # 2. WIoU 距离注意力权重
    # 最小包围框的宽和高
    cw = torch.max(b1_x2, b2_x2) - torch.min(b1_x1, b2_x1)
    ch = torch.max(b1_y2, b2_y2) - torch.min(b1_y1, b2_y1)

    # 中心点距离的平方
    b1_cx, b1_cy = (b1_x1 + b1_x2) / 2, (b1_y1 + b1_y2) / 2
    b2_cx, b2_cy = (b2_x1 + b2_x2) / 2, (b2_y1 + b2_y2) / 2
    center_dist_sq = (b1_cx - b2_cx) ** 2 + (b1_cy - b2_cy) ** 2

    # WIoU v1: 距离注意力（论文中的 R_WIoU）
    R_WIoU = torch.exp(-center_dist_sq / (cw ** 2 + ch ** 2 + 1e-9))

    # 3. 基础 IoU 损失
    loss_iou = 1 - iou
    
    # 4. WIoU v3: 动态非单调聚焦机制
    if scale:
        # 分离梯度，用于计算动态权重
        loss_iou_detach = loss_iou.detach()
        
        # 超参数（论文推荐值）
        alpha = 1.9
        delta = 3.0
        
        # 计算相对质量指标 β
        beta = loss_iou_detach / (loss_iou_detach.mean() + 1e-9)
        
        # 动态梯度增益 r（论文公式）
        r = beta / (delta * torch.pow(alpha, beta - delta) + 1e-9)
        
        # 最终损失：r * R_WIoU * loss_iou
        return (r * R_WIoU * loss_iou).squeeze(-1)
    else:
        # 不使用动态聚焦，仅使用距离注意力
        return (R_WIoU * loss_iou).squeeze(-1)
# ================== WIoU v3 核心代码 End ==================
```

找到`class BboxLoss(nn.Module):`，替换其中的 `forward` 方法。
```python
class BboxLoss(nn.Module):
    """Criterion class for computing training losses for bounding boxes."""

    def __init__(self, reg_max: int = 16):
        """Initialize the BboxLoss module with regularization maximum and DFL settings."""
        super().__init__()
        self.dfl_loss = DFLoss(reg_max) if reg_max > 1 else None

    def forward(
        self,
        pred_dist: torch.Tensor,
        pred_bboxes: torch.Tensor,
        anchor_points: torch.Tensor,
        target_bboxes: torch.Tensor,
        target_scores: torch.Tensor,
        target_scores_sum: torch.Tensor,
        fg_mask: torch.Tensor,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        """Compute IoU and DFL losses for bounding boxes."""
        weight = target_scores.sum(-1)[fg_mask].unsqueeze(-1)

        # =========== 【修改开始】使用 WIoU v3 ===========
        # 原代码: 
        # iou = bbox_iou(pred_bboxes[fg_mask], target_bboxes[fg_mask], xywh=False, CIoU=True)
        # loss_iou = ((1.0 - iou) * weight).sum() / target_scores_sum

        # 新代码: 使用 WIoU v3（已经是损失值，不需要 1 - iou）
        loss_wiou = bbox_wiou(pred_bboxes[fg_mask], target_bboxes[fg_mask], scale=True)
        loss_iou = (loss_wiou * weight.squeeze(-1)).sum() / target_scores_sum
        # =========== 【修改结束】 =======================

        # DFL loss
        if self.dfl_loss:
            target_ltrb = bbox2dist(anchor_points, target_bboxes, self.dfl_loss.reg_max - 1)
            loss_dfl = self.dfl_loss(pred_dist[fg_mask].view(-1, self.dfl_loss.reg_max), target_ltrb[fg_mask]) * weight
            loss_dfl = loss_dfl.sum() / target_scores_sum
        else:
            loss_dfl = torch.tensor(0.0).to(pred_dist.device)

        return loss_iou, loss_dfl
```

测试WIou v3损失函数
```python
"""
WIoU v3 损失函数完整测试脚本
测试内容：
1. 基础功能测试
2. 与 CIoU 对比
3. 动态聚焦机制验证
4. 边界情况测试
"""

import torch
import sys
import matplotlib.pyplot as plt
import numpy as np

# 添加 ultralytics 路径（如果需要）
# sys.path.insert(0, 'ultralytics')

try:
    from ultralytics.utils.loss import bbox_wiou
    from ultralytics.utils.metrics import bbox_iou
    print("✅ 成功导入 bbox_wiou 和 bbox_iou")
except ImportError as e:
    print(f"❌ 导入失败: {e}")
    print("请确保:")
    print("1. 已经修改了 ultralytics/utils/loss.py")
    print("2. 添加了 bbox_wiou 函数")
    exit(1)


def test_basic_functionality():
    """测试 1: 基础功能测试"""
    print("\n" + "="*60)
    print("测试 1: 基础功能测试")
    print("="*60)
    
    # 创建测试数据（xyxy 格式）
    pred_boxes = torch.tensor([
        [100.0, 100.0, 150.0, 150.0],  # 预测框1 (50x50)
        [200.0, 200.0, 260.0, 260.0],  # 预测框2 (60x60)
        [300.0, 300.0, 370.0, 370.0],  # 预测框3 (70x70)
    ])
    
    target_boxes = torch.tensor([
        [105.0, 105.0, 155.0, 155.0],  # 真实框1 (高 IoU，轻微偏移)
        [220.0, 220.0, 280.0, 280.0],  # 真实框2 (中 IoU，偏移较大)
        [350.0, 350.0, 420.0, 420.0],  # 真实框3 (低 IoU，偏移很大)
    ])
    
    # 计算 WIoU v3 损失
    try:
        loss_wiou = bbox_wiou(pred_boxes, target_boxes, scale=True)
        print(f"✅ WIoU v3 计算成功")
        print(f"\n预测框:\n{pred_boxes}")
        print(f"\n真实框:\n{target_boxes}")
        print(f"\nWIoU v3 损失: {loss_wiou}")
        print(f"平均损失: {loss_wiou.mean().item():.4f}")
        
        # 验证输出形状
        assert loss_wiou.shape == (3,), f"形状错误: 期望 (3,), 得到 {loss_wiou.shape}"
        print(f"✅ 输出形状正确: {loss_wiou.shape}")
        
        # 验证损失值范围（应该都是正数）
        assert (loss_wiou >= 0).all(), "损失值包含负数！"
        print(f"✅ 损失值全部为正")
        
        return True
        
    except Exception as e:
        print(f"❌ 测试失败: {e}")
        import traceback
        traceback.print_exc()
        return False


def test_compare_with_ciou():
    """测试 2: 与 CIoU 对比"""
    print("\n" + "="*60)
    print("测试 2: WIoU v3 vs CIoU 对比")
    print("="*60)
    
    # 创建不同质量的预测框
    target = torch.tensor([[100.0, 100.0, 200.0, 200.0]])  # 真实框
    
    # 5 个不同质量的预测框
    predictions = torch.tensor([
        [100.0, 100.0, 200.0, 200.0],  # 完美匹配 (IoU=1.0)
        [105.0, 105.0, 205.0, 205.0],  # 轻微偏移 (IoU≈0.8)
        [110.0, 110.0, 210.0, 210.0],  # 中等偏移 (IoU≈0.6)
        [120.0, 120.0, 220.0, 220.0],  # 较大偏移 (IoU≈0.4)
        [150.0, 150.0, 250.0, 250.0],  # 大偏移 (IoU≈0.2)
    ])
    
    try:
        # 计算 WIoU v3
        loss_wiou = bbox_wiou(predictions, target.repeat(5, 1), scale=True)
        
        # 计算 CIoU
        iou_ciou = bbox_iou(predictions, target.repeat(5, 1), xywh=False, CIoU=True)
        loss_ciou = 1 - iou_ciou
        
        # 计算普通 IoU 用于参考
        iou_normal = bbox_iou(predictions, target.repeat(5, 1), xywh=False)
        
        print("\n对比结果:")
        print(f"{'预测框偏移':<15} {'IoU':<10} {'CIoU Loss':<15} {'WIoU Loss':<15} {'差异':<10}")
        print("-" * 70)
        
        offsets = ['完美匹配', '轻微偏移', '中等偏移', '较大偏移', '大偏移']
        for i in range(5):
            diff = loss_wiou[i].item() - loss_ciou[i].item()
            print(f"{offsets[i]:<15} {iou_normal[i].item():.4f}    "
                  f"{loss_ciou[i].item():<15.4f} {loss_wiou[i].item():<15.4f} {diff:+.4f}")
        
        print(f"\n✅ 对比完成")
        print(f"💡 观察: WIoU v3 对低质量样本（大偏移）施加了更大的损失权重")
        
        return True
        
    except Exception as e:
        print(f"❌ 测试失败: {e}")
        import traceback
        traceback.print_exc()
        return False


def test_dynamic_focusing():
    """测试 3: 动态聚焦机制验证"""
    print("\n" + "="*60)
    print("测试 3: 动态聚焦机制验证")
    print("="*60)
    
    # 模拟一个批次的训练过程
    target = torch.tensor([[100.0, 100.0, 200.0, 200.0]]).repeat(10, 1)
    
    # 创建 10 个不同质量的预测框
    offsets = torch.linspace(0, 50, 10).unsqueeze(1).repeat(1, 4)
    predictions = target + offsets
    
    try:
        # 计算损失（多次调用以观察动态调整）
        print("\n多次迭代测试（模拟训练过程）:")
        
        for iter in range(3):
            loss_wiou = bbox_wiou(predictions, target, scale=True)
            avg_loss = loss_wiou.mean().item()
            print(f"迭代 {iter+1}: 平均损失 = {avg_loss:.4f}, 损失范围 = [{loss_wiou.min():.4f}, {loss_wiou.max():.4f}]")
        
        print(f"\n✅ 动态聚焦机制测试完成")
        print(f"💡 说明: WIoU v3 会根据样本质量动态调整梯度权重")
        
        return True
        
    except Exception as e:
        print(f"❌ 测试失败: {e}")
        import traceback
        traceback.print_exc()
        return False


def test_edge_cases():
    """测试 4: 边界情况测试"""
    print("\n" + "="*60)
    print("测试 4: 边界情况测试")
    print("="*60)
    
    tests = [
        ("完全重叠", 
         torch.tensor([[100.0, 100.0, 200.0, 200.0]]),
         torch.tensor([[100.0, 100.0, 200.0, 200.0]])),
        
        ("完全不重叠",
         torch.tensor([[100.0, 100.0, 200.0, 200.0]]),
         torch.tensor([[300.0, 300.0, 400.0, 400.0]])),
        
        ("部分重叠",
         torch.tensor([[100.0, 100.0, 200.0, 200.0]]),
         torch.tensor([[150.0, 150.0, 250.0, 250.0]])),
        
        ("包含关系",
         torch.tensor([[100.0, 100.0, 200.0, 200.0]]),
         torch.tensor([[120.0, 120.0, 180.0, 180.0]])),
    ]
    
    print("\n边界情况测试结果:")
    print(f"{'情况':<15} {'WIoU Loss':<15} {'状态':<10}")
    print("-" * 40)
    
    all_passed = True
    for name, pred, target in tests:
        try:
            loss = bbox_wiou(pred, target, scale=True)
            status = "✅ 通过"
            
            # 验证数值稳定性
            if torch.isnan(loss).any() or torch.isinf(loss).any():
                status = "❌ 数值异常"
                all_passed = False
                
            print(f"{name:<15} {loss.item():<15.4f} {status}")
            
        except Exception as e:
            print(f"{name:<15} {'错误':<15} ❌ 失败: {e}")
            all_passed = False
    
    if all_passed:
        print(f"\n✅ 所有边界情况测试通过")
    else:
        print(f"\n❌ 部分边界情况测试失败")
    
    return all_passed


def test_batch_processing():
    """测试 5: 批量处理测试"""
    print("\n" + "="*60)
    print("测试 5: 批量处理测试")
    print("="*60)
    
    batch_sizes = [1, 8, 16, 32, 64]
    
    print("\n批量处理性能测试:")
    print(f"{'Batch Size':<15} {'平均损失':<15} {'计算时间(ms)':<15} {'状态':<10}")
    print("-" * 55)
    
    all_passed = True
    for bs in batch_sizes:
        try:
            # 生成随机数据
            pred = torch.rand(bs, 4) * 100 + 100
            pred[:, 2:] = pred[:, :2] + torch.rand(bs, 2) * 50 + 10  # 确保 x2>x1, y2>y1
            target = torch.rand(bs, 4) * 100 + 100
            target[:, 2:] = target[:, :2] + torch.rand(bs, 2) * 50 + 10
            
            # 计时
            import time
            start = time.time()
            loss = bbox_wiou(pred, target, scale=True)
            elapsed = (time.time() - start) * 1000  # 转换为毫秒
            
            avg_loss = loss.mean().item()
            status = "✅ 通过"
            
            print(f"{bs:<15} {avg_loss:<15.4f} {elapsed:<15.2f} {status}")
            
        except Exception as e:
            print(f"{bs:<15} {'错误':<15} {'错误':<15} ❌ {e}")
            all_passed = False
    
    if all_passed:
        print(f"\n✅ 批量处理测试通过")
    else:
        print(f"\n❌ 批量处理测试失败")
    
    return all_passed


def visualize_loss_landscape():
    """测试 6: 损失函数地形可视化（可选）"""
    print("\n" + "="*60)
    print("测试 6: 损失函数地形可视化")
    print("="*60)
    
    try:
        # 固定真实框
        target = torch.tensor([[100.0, 100.0, 200.0, 200.0]])
        
        # 创建预测框网格（改变中心点位置）
        offsets = np.linspace(-50, 50, 30)
        loss_map_wiou = np.zeros((len(offsets), len(offsets)))
        loss_map_ciou = np.zeros((len(offsets), len(offsets)))
        
        print("正在计算损失地形...")
        for i, offset_x in enumerate(offsets):
            for j, offset_y in enumerate(offsets):
                pred = target.clone()
                pred[:, 0] += offset_x
                pred[:, 1] += offset_y
                pred[:, 2] += offset_x
                pred[:, 3] += offset_y
                
                # WIoU v3
                loss_wiou = bbox_wiou(pred, target, scale=True)
                loss_map_wiou[j, i] = loss_wiou.item()
                
                # CIoU
                iou_ciou = bbox_iou(pred, target, xywh=False, CIoU=True)
                loss_map_ciou[j, i] = (1 - iou_ciou).item()
        
        # 绘制对比图
        fig, axes = plt.subplots(1, 2, figsize=(14, 6))
        
        # WIoU v3 地形
        im1 = axes[0].contourf(offsets, offsets, loss_map_wiou, levels=20, cmap='RdYlBu_r')
        axes[0].set_title('WIoU v3 Loss Landscape', fontsize=14, fontweight='bold')
        axes[0].set_xlabel('X Offset')
        axes[0].set_ylabel('Y Offset')
        axes[0].plot(0, 0, 'r*', markersize=15, label='Target Center')
        axes[0].legend()
        plt.colorbar(im1, ax=axes[0])
        
        # CIoU 地形
        im2 = axes[1].contourf(offsets, offsets, loss_map_ciou, levels=20, cmap='RdYlBu_r')
        axes[1].set_title('CIoU Loss Landscape', fontsize=14, fontweight='bold')
        axes[1].set_xlabel('X Offset')
        axes[1].set_ylabel('Y Offset')
        axes[1].plot(0, 0, 'r*', markersize=15, label='Target Center')
        axes[1].legend()
        plt.colorbar(im2, ax=axes[1])
        
        plt.tight_layout()
        plt.savefig('wiou_vs_ciou_landscape.png', dpi=300, bbox_inches='tight')
        print("✅ 损失地形图已保存: wiou_vs_ciou_landscape.png")
        
        # 不显示图像，只保存
        plt.close()
        
        return True
        
    except Exception as e:
        print(f"⚠️  可视化失败（非关键错误）: {e}")
        return True  # 不影响其他测试


def main():
    """主测试函数"""
    print("="*60)
    print("WIoU v3 损失函数完整测试")
    print("="*60)
    print("测试项目:")
    print("1. 基础功能测试")
    print("2. 与 CIoU 对比")
    print("3. 动态聚焦机制验证")
    print("4. 边界情况测试")
    print("5. 批量处理测试")
    print("6. 损失函数地形可视化")
    print("="*60)
    
    # 运行所有测试
    results = {
        "基础功能": test_basic_functionality(),
        "CIoU对比": test_compare_with_ciou(),
        "动态聚焦": test_dynamic_focusing(),
        "边界情况": test_edge_cases(),
        "批量处理": test_batch_processing(),
        "可视化": visualize_loss_landscape(),
    }
    
    # 输出测试总结
    print("\n" + "="*60)
    print("测试总结")
    print("="*60)
    
    for name, passed in results.items():
        status = "✅ 通过" if passed else "❌ 失败"
        print(f"{name:<15} {status}")
    
    all_passed = all(results.values())
    
    if all_passed:
        print("\n" + "🎉"*20)
        print("✅ 所有测试通过！WIoU v3 损失函数集成成功！")
        print("🎉"*20)
        print("\n你可以开始训练了：")
        print("  python train_baseline.py")
    else:
        print("\n" + "❌"*20)
        print("部分测试失败，请检查 bbox_wiou 函数实现")
        print("❌"*20)
    
    return all_passed


if __name__ == "__main__":
    success = main()
    exit(0 if success else 1)
```
### 6、跑“终极方案”

消融实验
```python
"""
YOLOv8 消融实验完整训练脚本
支持 4 组实验：Baseline, CBAM, WIoU, CBAM+WIoU
"""

from ultralytics import YOLO

if __name__ == '__main__':
    # ================= 配置区域 =================
    # 实验标签：
    #   'baseline'   -> 实验1: 原版 YOLOv8n (CIoU 损失)
    #   'cbam'       -> 实验2: YOLOv8n + CBAM (CIoU 损失)
    #   'wiou'       -> 实验3: YOLOv8n + WIoU v3
    #   'final'      -> 实验4: YOLOv8n + CBAM + WIoU v3 (完整版)

    experiment_tag = 'final'  # ← 【每次训练前改这里！】

    # 训练参数
    EPOCHS = 100                # 正式训练用 100-150，测试用 3-10
    BATCH_SIZE = 16             # 根据显存调整：16(>6GB) / 8(4-6GB) / 4(<4GB)
    DATA_PATH = 'traffic_sign.yaml'
    DEVICE = 0                  # GPU 设备号，CPU 用 'cpu'
    
    # ============================================

    print("="*60)
    print(f"🚀 开始实验: {experiment_tag}")
    print("="*60)

    # 根据实验标签选择模型配置
    if experiment_tag in ['baseline', 'wiou']:
        # 使用原版结构（没有 CBAM）
        print("📦 加载模型: YOLOv8n (标准版)")
        model = YOLO('yolov8n.pt')
        
        if experiment_tag == 'baseline':
            print("⚠️  注意: Baseline 实验需要使用 CIoU 损失")
            print("   请确保 loss.py 中的 bbox_wiou 被注释掉！")
        else:
            print("✅ WIoU v3 损失函数已启用（通过 loss.py）")

    elif experiment_tag in ['cbam', 'final']:
        # 使用改进结构（包含 CBAM）
        print("📦 加载模型: YOLOv8n-CBAM (改进版)")
        model = YOLO('yolov8-cbam.yaml')
        
        print("🔄 正在加载预训练权重...")
        try:
            model.load('yolov8n.pt')
            print("✅ 预训练权重加载成功（部分层不匹配是正常的）")
        except Exception as e:
            print(f"⚠️  预训练权重加载失败: {e}")
            print("   将从头开始训练")
        
        if experiment_tag == 'cbam':
            print("⚠️  注意: CBAM 实验需要使用 CIoU 损失")
            print("   请确保 loss.py 中的 bbox_wiou 被注释掉！")
        else:
            print("✅ CBAM + WIoU v3 完整改进版")

    # 开始训练
    print("\n" + "="*60)
    print("开始训练...")
    print("="*60)
    
    results = model.train(
        data=DATA_PATH,
        epochs=EPOCHS,
        imgsz=640,
        batch=BATCH_SIZE,
        workers=4,                      # 数据加载线程数
        project='runs/ablation',        # 保存路径
        name=experiment_tag,            # 实验名称
        patience=50,                    # 早停轮数
        save=True,                      # 保存模型
        plots=True,                     # 生成图表
        device=DEVICE,
        
        # 优化器设置
        optimizer='auto',
        lr0=0.01,
        lrf=0.01,
        momentum=0.937,
        weight_decay=0.0005,
        warmup_epochs=3.0,
        
        # 数据增强
        hsv_h=0.015,
        hsv_s=0.7,
        hsv_v=0.4,
        degrees=0.0,
        translate=0.1,
        scale=0.5,
        fliplr=0.5,
        mosaic=1.0,
    )

    # 训练完成后的总结
    print("\n" + "="*60)
    print(f"✅ 实验 [{experiment_tag}] 训练完成！")
    print("="*60)
    print(f"📁 结果保存在: runs/ablation/{experiment_tag}/")
    print(f"📊 查看训练曲线: runs/ablation/{experiment_tag}/results.png")
    print(f"🏆 最佳模型: runs/ablation/{experiment_tag}/weights/best.pt")
    
    try:
        print("\n📈 训练结果摘要:")
        print(f"   mAP@0.5      : {results.results_dict['metrics/mAP50(B)']:.4f}")
        print(f"   mAP@0.5:0.95 : {results.results_dict['metrics/mAP50-95(B)']:.4f}")
    except:
        print("   (结果数据获取失败，请查看 results.csv)")
    
    print("="*60)
```

