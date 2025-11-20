## 测试电脑环境

### 第一步：安装必要的库

打开终端（Terminal 或 CMD），运行以下命令来安装 YOLOv8 的官方库 `ultralytics`：

``` Bash
pip install ultralytics
```

### 第二步：编写测试脚本

在电脑上新建一个 Python 文件，命名为 `check_env.py`：

``` python
import torch
import ultralytics
from ultralytics import YOLO
import sys

def check_environment():
    print("="*30)
    print("正在检查环境配置...")
    print("="*30)

    # 1. 检查 Python 版本
    print(f"[1] Python 版本: {sys.version.split()[0]}")

    # 2. 检查 Ultralytics (YOLOv8) 版本
    print(f"[2] Ultralytics 版本: {ultralytics.__version__}")

    # 3. 检查 PyTorch 与 硬件加速 (GPU)
    print(f"[3] PyTorch 版本: {torch.__version__}")
    
    if torch.cuda.is_available():
        gpu_count = torch.cuda.device_count()
        gpu_name = torch.cuda.get_device_name(0)
        print(f"    ✅ 发现 NVIDIA 显卡 (GPU) 加速可用！")
        print(f"    显卡数量: {gpu_count}")
        print(f"    显卡型号: {gpu_name}")
        device = 'cuda'
    else:
        print(f"    ⚠️ 未检测到 NVIDIA 显卡，将使用 CPU 进行计算。")
        print(f"    (提示：对于毕设演示，CPU 推理完全够用；但训练模型会比较慢。)")
        device = 'cpu'

    print("-" * 30)
    print("正在尝试加载模型并进行一次测试推理...")
    
    try:
        # 4. 加载预训练模型 (第一次运行会自动下载 yolov8n.pt，约 6MB)
        model = YOLO('yolov8n.pt') 
        
        # 5. 随便找个东西识别一下 (这里使用官方自带的一张图)
        # source可以是一个在线图片的URL，也可以是本地图片路径
        results = model('https://ultralytics.com/images/bus.jpg', verbose=False)
        
        print("    ✅ 模型加载成功！")
        print("    ✅ 推理测试成功！YOLOv8 运行正常。")
        print(f"    检测到的物体数量: {len(results[0].boxes)}")
        
    except Exception as e:
        print(f"    ❌ 发生错误: {e}")
        print("    建议：请检查网络连接（模型下载需要网络）或截图报错信息给我。")

    print("="*30)
    print("环境检查结束")

if __name__ == "__main__":
    check_environment()
```

### 第三步：运行并查看结果

在终端运行这个文件：

``` bash
python check_env.py
```

**输出结果：**

1. **关于 GPU：**
    
    - 如果显示 **`✅ 发现 NVIDIA 显卡`**：电脑配置很棒，训练模型会很快。
        
    - 如果显示 **`⚠️ 未检测到 NVIDIA 显卡`**：不要慌。依然可以完成。
        
        - _方案：_ 在本地写代码和做界面（CPU 足够），训练模型时可以使用免费的云端算力（比如 Kaggle 或 Google Colab）。
            
2. **关于测试推理：**
    
    - 只要看到 **`✅ 推理测试成功！`**，就说明开发环境已经打通了，可以正式开始做项目了。
        

# 全流程

### 第一阶段：基础环境搭建 (地基)

为了不把你的电脑搞乱，强烈建议使用 **Anaconda** 来管理环境。如果你还没安装 Anaconda，请先去官网下载安装。

**步骤 1：创建项目文件夹**

1. 在你的电脑硬盘里（最好不要在C盘，除非你C盘很大）新建一个文件夹。
    
2. 命名为：`Garbage_System`。
    
3. 在这个文件夹里，右键 -> “通过 VS Code 打开” (或者你用 PyCharm 也可以)。
    

**步骤 2：创建虚拟环境** 打开 VS Code 的终端 (Terminal)，输入以下命令。这会创建一个专门用于这个项目的“纯净”房间：

``` Bash
# 1. 创建名为 garbage_env 的环境，指定 python 版本为 3.9
conda create -n garbage_env python=3.9 -y

# 2. 激活进入这个环境 (激活后，你终端前面的括号会变成 garbage_env)
conda activate garbage_env
```

**步骤 3：安装核心库** 在激活的环境下，安装 YOLOv8 和 界面库 Streamlit：

``` Bash
# 安装 YOLOv8 核心库
pip install ultralytics

# 安装 界面开发库
pip install streamlit

# 安装 图片处理工具
pip install opencv-python-headless
```

---

### 第二阶段：数据准备 (最关键的一步)

题目要求“构建自定义垃圾图像数据集” 。为了快速跑通，我们先用网上的开源数据集，等流程通了，你再拍几张自己的照片加进去微调。

**步骤 1：理解 YOLO 的数据格式** YOLO 对文件夹的结构非常挑剔，**必须**长这样，请严格照做：

``` Plaintext
Garbage_System/  (你的项目根目录)
│
├── datasets/          <-- 新建这个文件夹
│   └── garbage_data/  <-- 新建这个文件夹
│       ├── train/     <-- 用于训练
│       │   ├── images/ (放 jpg 图片)
│       │   └── labels/ (放 txt 标签文件)
│       └── val/       <-- 用于验证/考试
│           ├── images/
│           └── labels/
```

**步骤 2：获取数据**

- **推荐方案：** 去 **Kaggle** 搜索 "Garbage Classification YOLO"。
    
- **快速方案（为了现在就能跑通）：** 我们可以先用极其少量的数据手动模拟一下，确保代码能跑。
    
    1. 在 `train/images` 里随便放 2 张 **瓶子** 的照片，2 张 **电池** 的照片。
        
    2. 你需要用工具给它们“画框”。下载一个叫 `LabelImg` 的软件（或者用在线网站 Roboflow）。
        
    3. **标注规则：**
        
        - 0号类别: `bottle` (瓶子/可回收)
            
        - 1号类别: `battery` (电池/有害)
            
    4. 标注完，你会得到 `.txt` 文件，把它们放到 `train/labels` 对应位置。
        

**步骤 3：创建配置文件 (data.yaml)** 在 `Garbage_System` 根目录下，新建一个文件叫 `data.yaml`，把下面的内容复制进去：

``` YAML
# data.yaml  
  
# 注意：这里的路径可以是绝对路径，也可以是相对路径  
# 这里的 path 使用绝对路径，避免 YOLO 找不到位置  
# 注意：Windows下为了防止出错，我们把反斜杠 "\" 改成正斜杠 "/"#path: C:/app/cod_tool/program_tool/jet/pycharm_fil/25/1119/yolo02/Garbage_System/datasets/garbage_data  
## 数据集根目录  
#train: train/images            # 训练集图片位置  
#val: valid/images                # 验证集图片位置  
  
## 类别数量 (根据你实际的分类来，这里假设我们要分4类)  
#nc: 5  
#  
## 类别名称 (顺序必须和标注时的 ID 对应)  
#  # 从 4 改成 5 (或者更大)  
#names: ['Recyclable', 'Hazardous', 'Kitchen', 'Other', 'Unknown'] # 加个凑数的  
  
names:  
  - Aerosols  
  - Aluminum can  
  - Aluminum caps  
  - Cardboard  
  - Cellulose  
  - Ceramic  
  - Combined plastic  
  - Container for household chemicals  
  - Disposable tableware  
  - Electronics  
  - Foil  
  - Furniture  
  - Glass bottle  
  - Iron utensils  
  - Liquid  
  - Metal shavings  
  - Milk bottle  
  - Organic  
  - Paper bag  
  - Paper cups  
  - Paper shavings  
  - Paper  
  - Papier mache  
  - Plastic bag  
  - Plastic bottle  
  - Plastic can  
  - Plastic canister  
  - Plastic caps  
  - Plastic cup  
  - Plastic shaker  
  - Plastic shavings  
  - Plastic toys  
  - Postal packaging  
  - Printing industry  
  - Scrap metal  
  - Stretch film  
  - Tetra pack  
  - Textile  
  - Tin  
  - Unknown plastic  
  - Wood  
  - Zip plastic bag  
  - Ramen Cup  
  - Food Packet  
nc: 44  
  
#roboflow:  
#  license: CC BY 4.0  
#  project: paper-iybve  
#  url: https://universe.roboflow.com/modern-academy-for-engineering-enkza/paper-iybve/dataset/1  
#  version: 1  
#  workspace: modern-academy-for-engineering-enkza  
  
path: C:/app/cod_tool/program_tool/jet/pycharm_fil/25/1119/yolo02/Garbage_System/datasets/garbage_data  
  
  
train: train/images             # 相对于 path 字段的路径  
val: valid/images               # 相对于 path 字段的路径  
test: test/images               # 相对于 path 字段的路径  
  
#train: ../datasets/garbage_data/train/images  
#val: ../datasets/garbage_data/valid/images # 注意：Roboflow通常叫valid而不是val  
#test: ../datasets/garbage_data/test/images
```

_(注：上面的 names 对应：可回收、有害、厨余、其他)_

---

### 第三阶段：模型训练 (核心算法)

题目要求“完成YOLOv8模型的训练” 。

**步骤 1：编写训练脚本** 在根目录下新建 `train.py`，复制以下代码：

``` Python
from ultralytics import YOLO  
import torch  
print(torch.cuda.is_available())  
  
  
def main():  
    # 1. 加载预训练模型 (yolov8n 是最快、最小的模型，适合笔记本电脑)  
    model = YOLO('yolov8n.pt')  
  
    # 2. 开始训练  
    # data: 指向刚才写的 data.yaml    # epochs: 训练多少轮 (为了测试，先设 10 轮；正式跑建议 50-100)    # imgsz: 图片大小，默认 640    print("开始训练...")  
    # model.train(data='data.yaml', epochs=10, imgsz=640, device='cpu')  
    # 注意：如果你电脑有显卡，把 device='cpu' 删掉，它会自动用显卡  
    # ---------------------------------------------  
    # 关键修改：将 device='cpu' 更改为 device='0'    
    # ---------------------------------------------    
    model.train(data='data.yaml', epochs=10, imgsz=640, device='0')  
  
    # 3. 验证模型  
    metrics = model.val()  
    print("训练结束！模型已保存。")  
  
if __name__ == '__main__':  
    main()
```

**步骤 2：运行训练** 在终端输入：

``` bash
python train.py
```

如果看到进度条在走，恭喜你，你正在训练自己的人工智能了！ 训练完成后，结果会保存在 `runs/detect/train` 文件夹里，里面的 `weights/best.pt` 就是你的**最终成果**。

---

### 第四阶段：系统界面开发 (所见即所得)

题目要求“开发一个简单的Web或桌面应用接口” 。我们使用 Streamlit，它能用 Python 直接画网页。

**步骤 1：编写界面脚本** 在根目录下新建 `app.py`，复制以下代码：

``` Python
import streamlit as st  
from ultralytics import YOLO  
from PIL import Image  
import os  
import cv2  # <<<--- 1. 导入 cv2  
# ----------------------------------------------------  
# 44 个细分类别到 4 个大类别的映射（你需要根据实际情况调整）  
# ----------------------------------------------------  
CATEGORY_MAP = {  
    # 可回收物 (Recyclable) - 投入【蓝色】桶  
    'Aluminum can': '可回收物',  
    'Aluminum caps': '可回收物',  
    'Cardboard': '可回收物',  
    'Combined plastic': '可回收物',  
    'Foil': '可回收物',  
    'Glass bottle': '可回收物',  
    'Iron utensils': '可回收物',  
    'Milk bottle': '可回收物',  
    'Paper bag': '可回收物',  
    'Paper': '可回收物',  
    'Plastic bottle': '可回收物',  
    'Plastic can': '可回收物',  
    'Plastic canister': '可回收物',  
    'Plastic caps': '可回收物',  
    'Plastic cup': '可回收物',  
    'Scrap metal': '可回收物',  
    'Stretch film': '可回收物',  
    'Tin': '可回收物',  
    'Zip plastic bag': '可回收物',  
    'Printing industry': '可回收物',  
  
    # 厨余垃圾 (Kitchen/Wet Waste) - 投入【绿色】/【棕色】桶  
    'Organic': '厨余垃圾',  
  
    # 有害垃圾 (Hazardous Waste) - 投入【红色】桶  
    'Aerosols': '有害垃圾',  
    'Container for household chemicals': '有害垃圾',  
    'Electronics': '有害垃圾',  
  
    # 其他垃圾 (Other/Dry Waste) - 投入【黑色】/【灰色】桶  
    'Cellulose': '其他垃圾',  
    'Ceramic': '其他垃圾',  
    'Disposable tableware': '其他垃圾',  
    'Furniture': '其他垃圾',  
    'Liquid': '其他垃圾', # 液体通常需要处理后扔其他垃圾  
    'Metal shavings': '其他垃圾',  
    'Paper cups': '其他垃圾',  
    'Paper shavings': '其他垃圾',  
    'Papier mache': '其他垃圾',  
    'Plastic bag': '其他垃圾',  
    'Plastic shaker': '其他垃圾',  
    'Plastic shavings': '其他垃圾',  
    'Plastic toys': '其他垃圾',  
    'Postal packaging': '其他垃圾',  
    'Tetra pack': '其他垃圾', # 利乐包通常是复合材料，算其他  
    'Textile': '其他垃圾',  
    'Unknown plastic': '其他垃圾',  
    'Wood': '其他垃圾',  
    'Ramen Cup': '其他垃圾',  
    'Food Packet': '其他垃圾',  
  
    # 默认/未知类别  
    'default': '其他垃圾',  
}  
  
# 设置页面标题  
st.set_page_config(page_title="垃圾智能分类系统", layout="centered")  
  
st.title("🗑️ 基于 YOLOv8 的垃圾智能分类系统")  
st.markdown("---")  
  
# 侧边栏  
st.sidebar.header("设置")  
# 更改默认置信度为 0.50，以过滤掉你现在模型的大量错误猜测  
confidence = st.sidebar.slider("置信度阈值", 0.0, 1.0, 0.50, help="只有只有确定性高于这个分数的才会被显示")  
# 1. 加载模型  
# 训练好后，把路径改成 'runs/detect/train/weights/best.pt'# 现在为了测试，我们先用官方模型占位，等你训练好了再换  
# model_path = 'yolov8n.pt'  
model_path = 'runs/detect/train2/weights/best.pt' # <--- 解开这行注释使用你自己的模型  
  
try:  
    model = YOLO(model_path)  
except Exception as e:  
    st.error(f"模型加载失败，请检查路径: {e}")  
  
# 2. 图片上传功能  
uploaded_file = st.file_uploader("请上传一张垃圾图片...", type=['jpg', 'jpeg', 'png'])  
  
if uploaded_file is not None:  
    # 显示原图  
    image = Image.open(uploaded_file)  
    # st.image(image, caption='原始图片', use_column_width=True)  
    st.image(image, caption='原始图片', use_container_width=True) # <<<--- 修复 use_column_width 警告  
  
    # 按钮触发识别  
    if st.button('🔍 开始智能识别'):  
        with st.spinner('正在分析中...'):  
            # 调用模型进行预测  
            results = model.predict(image, conf=confidence)  
  
            # 绘制结果  (输出是 BGR 格式的 NumPy 数组)  
            res_plotted = results[0].plot()  
  
            # 【关键修复】将 BGR 转换为 Streamlit/RGB 格式  
            res_rgb = cv2.cvtColor(res_plotted, cv2.COLOR_BGR2RGB)  
  
            # 显示结果图  
            st.image(res_rgb, caption='识别结果', use_container_width=True) # <<<--- 使用 RGB 图像  
  
            # 显示具体的检测信息  
            # 只有当检测到物体时才显示 success 和 for 循环内容  
            if len(results[0].boxes) > 0:  
                st.success("检测完成！")  
            else:  
                st.warning("检测完成！但未发现置信度高于阈值的垃圾。请尝试降低阈值。")  
  
  
            for box in results[0].boxes:  
                class_id = int(box.cls[0])  
                class_name = model.names[class_id]  
                prob = float(box.conf[0])  
  
                st.write(f"👉 检测到：**{class_name}** (置信度: {prob:.2%})")  
  
                # 基于 44 个细分类别，给出 4 个大类别的分类建议  
                general_category = CATEGORY_MAP.get(class_name, CATEGORY_MAP['default'])  
  
                if '可回收物' in general_category:  
                    st.info(f"💡 分类建议：请投入【{general_category}桶】")  
                elif '厨余垃圾' in general_category:  
                    st.success(f"🍎 分类建议：请投入【{general_category}桶】")  
                elif '有害垃圾' in general_category:  
                    st.warning(f"⚠️ 分类建议：请投入【{general_category}桶】")  
                else: # 其他垃圾  
                    st.error(f"🗑️ 分类建议：请投入【{general_category}桶】") # 使用 error 或其他样式强调
```

**步骤 2：启动系统** 在终端输入：

Bash

```
streamlit run app.py
```

系统会自动弹出一个网页。你可以上传一张图片（比如你在百度搜一张“瓶子”的图），看看它能不能框出来！