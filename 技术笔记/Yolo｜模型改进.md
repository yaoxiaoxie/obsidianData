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
        elif m is CBAM:
            args = [ch[f], *args]
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
def bbox_wiou(box1, box2, scale=1.0):  
    # box1: pred [N, 4] (xyxy), box2: target [N, 4] (xyxy)  
  
    # 1. 基础 IoU 计算  
    b1_x1, b1_y1, b1_x2, b1_y2 = box1.chunk(4, -1)  
    b2_x1, b2_y1, b2_x2, b2_y2 = box2.chunk(4, -1)  
  
    inter_x1 = torch.max(b1_x1, b2_x1)  
    inter_y1 = torch.max(b1_y1, b2_y1)  
    inter_x2 = torch.min(b1_x2, b2_x2)  
    inter_y2 = torch.min(b1_y2, b2_y2)  
  
    inter_area = (inter_x2 - inter_x1).clamp(0) * (inter_y2 - inter_y1).clamp(0)  
    b1_area = (b1_x2 - b1_x1) * (b1_y2 - b1_y1)  
    b2_area = (b2_x2 - b2_x1) * (b2_y2 - b2_y1)  
    union = b1_area + b2_area - inter_area + 1e-9  
    iou = inter_area / union  
  
    # 2. WIoU 计算  
    # 最小包围框的宽和高  
    cw = torch.max(b1_x2, b2_x2) - torch.min(b1_x1, b2_x1)  
    ch = torch.max(b1_y2, b2_y2) - torch.min(b1_y1, b2_y1)  
  
    # 中心点距离的平方  
    b1_cx, b1_cy = (b1_x1 + b1_x2) / 2, (b1_y1 + b1_y2) / 2  
    b2_cx, b2_cy = (b2_x1 + b2_x2) / 2, (b2_y1 + b2_y2) / 2  
    center_dist_sq = (b1_cx - b2_cx) ** 2 + (b1_cy - b2_cy) ** 2  
  
    # WIoU v1: 距离注意力  
    R_WIoU = torch.exp(center_dist_sq / (cw ** 2 + ch ** 2 + 1e-9))  
  
    # WIoU v3: 动态非单调聚焦机制  
    # outlier degree  
    loss_iou = 1 - iou  
    dist = R_WIoU * loss_iou  
    loss_iou_detach = loss_iou.detach()  
  
    # 动态参数 (alpha, delta) - 可根据需要调整，推荐 alpha=1.9, delta=3    alpha = 1.9  
    delta = 3.0  
  
    # 梯度增益 r    beta = loss_iou_detach / (loss_iou_detach.mean() + 1e-9)  
    r = beta / (delta * torch.pow(alpha, beta - delta) + 1e-9)  
  
    return r * dist  
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
        # 原代码: iou = bbox_iou(pred_bboxes[fg_mask], target_bboxes[fg_mask], xywh=False, CIoU=True)
        # 原代码: loss_iou = ((1.0 - iou) * weight).sum() / target_scores_sum
        
        # 新代码: 直接计算 loss
        loss_wiou = bbox_wiou(pred_bboxes[fg_mask], target_bboxes[fg_mask])
        loss_iou = (loss_wiou * weight).sum() / target_scores_sum
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

### 6、跑“终极方案”

消融实验
```python
from ultralytics import YOLO

if __name__ == '__main__':
    # ================= 配置区域 =================
    # 请根据你当前想跑的实验，修改下面的 experiment_tag 变量
    # 可选值: 
    #   'baseline'   -> 实验1: 原版模型 (需还原 loss.py)
    #   'cbam'       -> 实验2: 只有CBAM (需还原 loss.py)
    #   'wiou'       -> 实验3: 只有WIoU (保持当前 loss.py)
    #   'final'      -> 实验4: CBAM + WIoU (保持当前 loss.py)
    
    experiment_tag = 'final'  # <---【每次训练前改这里！】
    
    # 训练参数设置
    EPOCHS = 100           # 正式训练建议 100-300
    BATCH_SIZE = 16        # 显存够就 16，不够改 8 或 4
    DATA_PATH = 'traffic_sign.yaml'
    # ===========================================

    print(f"🚀 正在准备进行实验: {experiment_tag}")

    # 1. 根据实验标签自动选择模型文件
    if experiment_tag == 'baseline' or experiment_tag == 'wiou':
        # 这两组实验用的是原版结构，所以直接加载官方权重，不仅结构对，还有预训练参数
        print("Loading: Standard YOLOv8n...")
        model = YOLO('yolov8n.pt') 
        
    elif experiment_tag == 'cbam' or experiment_tag == 'final':
        # 这两组实验用的是改过结构的 yaml，必须加载 yaml 文件
        print("Loading: YOLOv8-CBAM structure...")
        model = YOLO('yolov8-cbam.yaml') 
        
        # 【可选】加载预训练权重加速收敛（虽然结构变了，但这招通常好用）
        # model.load('yolov8n.pt') 

    # 2. 开始训练
    results = model.train(
        data=DATA_PATH,
        epochs=EPOCHS,
        imgsz=640,
        batch=BATCH_SIZE,
        workers=2,
        project='runs/train',      # 所有实验都会保存在 runs/train 目录下
        name=experiment_tag,       # 文件夹名字会自动变成 'baseline', 'final' 等，方便你看
        patience=50,               # 50轮不提升就早停
        save=True,                 # 保存模型
        device=0                   # 使用 0 号显卡，如果没有显卡改成 'cpu'
    )

    print(f"✅ 实验 {experiment_tag} 结束！请记得截图 results.png")
```

