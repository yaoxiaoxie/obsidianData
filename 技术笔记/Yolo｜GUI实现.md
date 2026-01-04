
```python
import sys  
import cv2  
import os  
import time  
import numpy as np  
import multiprocessing  # 记得在文件顶部导入  
from datetime import datetime  
from ultralytics import YOLO  
  
from PyQt5.QtWidgets import (QApplication, QMainWindow, QLabel, QPushButton,  
                             QVBoxLayout, QWidget, QHBoxLayout, QComboBox,  
                             QSpinBox, QGroupBox, QStatusBar, QFileDialog,  
                             QScrollArea, QFrame, QCheckBox)  
from PyQt5.QtGui import QImage, QPixmap, QFont  
from PyQt5.QtCore import QThread, pyqtSignal, Qt  
  
# ----------------------------------------------------------------------  
# 1. 核心检测线程（支持 ONNX 加速与性能测试模式）  
# ----------------------------------------------------------------------  
class DetectionThread(QThread):  
    change_pixmap_signal = pyqtSignal(QImage)      # 更新画面  
    fps_signal = pyqtSignal(float)                 # 更新FPS  
    detection_signal = pyqtSignal(int)             # 更新检测数量  
    class_count_signal = pyqtSignal(dict)          # 更新类别统计  
    error_signal = pyqtSignal(str)                 # 错误/状态提示  
    finished_signal = pyqtSignal()                 # 播放结束  
    progress_signal = pyqtSignal(int, int)         # 进度条  
  
    def __init__(self, model_path="best.pt", source=0,  
                 source_type="camera", confidence=0.25,  
                 use_onnx=False, realtime_mode=True):  
        super().__init__()  
        self.running = True  
        self.model_path = model_path  
        self.source = source  
        self.source_type = source_type  
        self.confidence = confidence  
  
        # --- 新增参数 ---        self.use_onnx = use_onnx          # 是否使用 ONNX        self.realtime_mode = realtime_mode # 是否原速播放（False则全速跑，用于测极限FPS）  
  
        self.model = None  
        self.cap = None  
        self.fps = 0  
  
    def run(self):  
        try:  
            # 1. 模型加载逻辑（含自动导出 ONNX）  
            target_model_path = self.model_path  
  
            if self.use_onnx:  
                # 构造 onnx 文件名 (例如: detector.pt -> detector.onnx)  
                base_name = os.path.splitext(self.model_path)[0]  
                onnx_path = base_name + ".onnx"  
  
                # 检查是否存在，不存在则自动导出  
                if not os.path.exists(onnx_path):  
                    self.error_signal.emit(f"正在将模型导出为 ONNX 格式，请稍候...\n目标路径: {onnx_path}")  
                    try:  
                        # 加载 PyTorch 模型进行导出  
                        temp_model = YOLO(self.model_path)  
                        temp_model.export(format='onnx')  
                        self.error_signal.emit("导出成功！正在加载 ONNX 模型...")  
                    except Exception as e:  
                        self.error_signal.emit(f"ONNX 导出失败，自动降级回 PyTorch: {e}")  
                        onnx_path = self.model_path # 回退  
  
                target_model_path = onnx_path  
  
            # 加载模型 (YOLOv8 会自动根据文件后缀识别加载方式)  
            if not os.path.exists(target_model_path) and not self.use_onnx:  
                self.error_signal.emit(f"模型文件未找到: {target_model_path}")  
                return  
  
            self.model = YOLO(target_model_path)  
  
            # 2. 根据源类型处理  
            if self.source_type == "image":  
                self.process_image()  
            else:  
                self.process_video()  
  
        except Exception as e:  
            self.error_signal.emit(f"检测线程系统错误: {str(e)}")  
        finally:  
            self.cleanup()  
  
    def process_image(self):  
        """处理单张图片"""  
        try:  
            frame = cv2.imread(self.source)  
            if frame is None:  
                self.error_signal.emit("无法读取图片文件")  
                return  
  
            # 推理  
            results = self.model(frame, conf=self.confidence, verbose=False)  
            detections = results[0].boxes  
            num_detections = len(detections)  
            class_counts = self.count_classes(results[0])  
  
            # 绘图  
            annotated_frame = results[0].plot()  
  
            # 显示模式标记  
            mode_str = "ONNX Runtime" if self.use_onnx else "PyTorch"  
            cv2.putText(annotated_frame, f"Mode: {mode_str}", (10, 80),  
                        cv2.FONT_HERSHEY_SIMPLEX, 0.8, (0, 0, 255), 2)  
            cv2.putText(annotated_frame, f"Count: {num_detections}", (10, 40),  
                        cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 255, 0), 2)  
  
            # 格式转换  
            rgb_image = cv2.cvtColor(annotated_frame, cv2.COLOR_BGR2RGB)  
            h, w, ch = rgb_image.shape  
            qt_image = QImage(rgb_image.data, w, h, ch * w, QImage.Format_RGB888)  
  
            # 发送信号  
            self.change_pixmap_signal.emit(qt_image.copy())  
            self.detection_signal.emit(num_detections)  
            self.class_count_signal.emit(class_counts)  
            self.finished_signal.emit()  
  
        except Exception as e:  
            self.error_signal.emit(f"图片处理错误: {str(e)}")  
  
    def process_video(self):  
        """处理视频流（支持性能测试模式）"""  
        try:  
            # 打开视频源  
            if self.source_type == "camera":  
                self.cap = cv2.VideoCapture(self.source)  
                self.cap.set(cv2.CAP_PROP_FRAME_WIDTH, 1280)  
                self.cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 720)  
                self.cap.set(cv2.CAP_PROP_FPS, 30)  
                video_fps = 30  
            else:  
                self.cap = cv2.VideoCapture(self.source)  
                video_fps = self.cap.get(cv2.CAP_PROP_FPS)  
                if video_fps <= 0: video_fps = 25  
  
            if not self.cap.isOpened():  
                self.error_signal.emit(f"无法打开视频源: {self.source}")  
                return  
  
            total_frames = int(self.cap.get(cv2.CAP_PROP_FRAME_COUNT)) if self.source_type == "video" else 0  
            frame_delay = 1.0 / video_fps # 标准帧间隔  
  
            frame_count = 0  
            start_time = datetime.now()  
  
            while self.running:  
                frame_start = time.time() # 记录开始时间  
  
                ret, frame = self.cap.read()  
                if not ret:  
                    if self.source_type == "video":  
                        self.finished_signal.emit()  
                    else:  
                        self.error_signal.emit("摄像头读取失败")  
                    break  
  
                # --- YOLO 推理 ---                results = self.model(frame, conf=self.confidence, verbose=False)  
                detections = results[0].boxes  
                class_counts = self.count_classes(results[0])  
                annotated_frame = results[0].plot()  
  
                # --- 信息叠加 ---                mode_str = "ONNX" if self.use_onnx else "PyTorch"  
                info_text = f"FPS: {self.fps:.1f} | Mode: {mode_str} | Obj: {len(detections)}"  
                cv2.putText(annotated_frame, info_text, (10, 30),  
                            cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 0), 2)  
  
                # --- 图片转换 ---                rgb_image = cv2.cvtColor(annotated_frame, cv2.COLOR_BGR2RGB)  
                h, w, ch = rgb_image.shape  
                qt_image = QImage(rgb_image.data, w, h, ch * w, QImage.Format_RGB888)  
  
                # --- 发送信号 ---                self.change_pixmap_signal.emit(qt_image.copy())  
                self.detection_signal.emit(len(detections))  
                self.class_count_signal.emit(class_counts)  
  
                if self.source_type == "video":  
                    curr_pos = int(self.cap.get(cv2.CAP_PROP_POS_FRAMES))  
                    self.progress_signal.emit(curr_pos, total_frames)  
  
                # --- FPS 计算 ---                frame_count += 1  
                if frame_count >= 15: # 每15帧刷新一次FPS，反应更灵敏  
                    elapsed = (datetime.now() - start_time).total_seconds()  
                    self.fps = frame_count / elapsed if elapsed > 0 else 0  
                    self.fps_signal.emit(self.fps)  
                    frame_count = 0  
                    start_time = datetime.now()  
  
                # --- 速度控制 (核心修改) ---  
                # 如果是视频文件 且 开启了原速播放模式，则进行延时  
                # 如果关闭原速播放，则全力运行（测试最大FPS）  
                if self.source_type == "video" and self.realtime_mode:  
                    processing_time = time.time() - frame_start  
                    wait_time = frame_delay - processing_time  
                    if wait_time > 0:  
                        time.sleep(wait_time)  
  
        except Exception as e:  
            self.error_signal.emit(f"视频处理错误: {str(e)}")  
  
    def count_classes(self, result):  
        class_counts = {}  
        if result.boxes is not None and len(result.boxes) > 0:  
            class_ids = result.boxes.cls.cpu().numpy()  
            for class_id in class_ids:  
                class_id = int(class_id)  
                class_name = result.names[class_id] if class_id in result.names else f"Class_{class_id}"  
                class_counts[class_name] = class_counts.get(class_name, 0) + 1  
        return class_counts  
  
    def cleanup(self):  
        if self.cap is not None:  
            self.cap.release()  
  
    def stop(self):  
        self.running = False  
        self.wait()  
  
    def update_confidence(self, conf):  
        self.confidence = conf  
  
# ----------------------------------------------------------------------  
# 2. 主界面 (UI)# ----------------------------------------------------------------------  
class MainWindow(QMainWindow):  
    def __init__(self):  
        super().__init__()  
        self.setWindowTitle("基于YOLO的交通标志检测系统 (毕设专用版)")  
        self.setGeometry(100, 100, 1250, 800)  
  
        # 样式表  
        self.setStyleSheet("""  
            QMainWindow { background-color: #2b2b2b; }            QLabel { color: #ffffff; }            QPushButton {                background-color: #0d7377; color: white; border: none;                border-radius: 5px; padding: 10px; font-weight: bold;            }            QPushButton:hover { background-color: #14a085; }            QPushButton:disabled { background-color: #555555; }            QGroupBox {                color: #ffffff; border: 2px solid #0d7377;                border-radius: 5px; margin-top: 10px; font-weight: bold;            }            QGroupBox::title { subcontrol-origin: margin; left: 10px; padding: 0 5px; }            QComboBox, QSpinBox, QCheckBox {                background-color: #3b3b3b; color: white;                border: 1px solid #0d7377; border-radius: 3px; padding: 5px;            }            QCheckBox { padding: 8px; }        """)  
  
        # --- 布局初始化 ---        central_widget = QWidget()  
        self.setCentralWidget(central_widget)  
        main_layout = QHBoxLayout()  
        central_widget.setLayout(main_layout)  
  
        # === 左侧：视频区域 ===        video_layout = QVBoxLayout()  
        self.image_label = QLabel(self)  
        self.image_label.setAlignment(Qt.AlignCenter)  
        self.image_label.setStyleSheet("border: 3px solid #0d7377; background-color: #000; border-radius: 10px;")  
        self.image_label.setMinimumSize(800, 600)  
        self.image_label.setScaledContents(True)  
        video_layout.addWidget(self.image_label)  
        main_layout.addLayout(video_layout, 3)  
  
        # === 右侧：控制面板 ===        control_panel = QVBoxLayout()  
        control_panel.setSpacing(15)  
  
        # 1. 功能按钮  
        btn_group = QGroupBox("系统控制")  
        btn_layout = QVBoxLayout()  
  
        self.start_btn = QPushButton("🎥 启动摄像头检测")  
        self.start_btn.clicked.connect(self.start_detection)  
        btn_layout.addWidget(self.start_btn)  
  
        self.stop_btn = QPushButton("⏹ 停止检测")  
        self.stop_btn.setEnabled(False)  
        self.stop_btn.clicked.connect(self.stop_detection)  
        btn_layout.addWidget(self.stop_btn)  
  
        hbox_file = QHBoxLayout()  
        self.upload_image_btn = QPushButton("🖼 图片")  
        self.upload_image_btn.clicked.connect(self.upload_image)  
        hbox_file.addWidget(self.upload_image_btn)  
  
        self.upload_video_btn = QPushButton("🎬 视频")  
        self.upload_video_btn.clicked.connect(self.upload_video)  
        hbox_file.addWidget(self.upload_video_btn)  
        btn_layout.addLayout(hbox_file)  
  
        self.snapshot_btn = QPushButton("📷 截图存证")  
        self.snapshot_btn.setEnabled(False)  
        self.snapshot_btn.clicked.connect(self.save_snapshot)  
        btn_layout.addWidget(self.snapshot_btn)  
  
        btn_group.setLayout(btn_layout)  
        control_panel.addWidget(btn_group)  
  
        # 2. 参数与性能设置  
        settings_group = QGroupBox("参数与加速配置")  
        settings_layout = QVBoxLayout()  
  
        # 摄像头选择  
        cam_layout = QHBoxLayout()  
        cam_layout.addWidget(QLabel("设备:"))  
        self.camera_combo = QComboBox()  
        self.camera_combo.addItems([f"Camera {i}" for i in range(3)])  
        cam_layout.addWidget(self.camera_combo)  
        settings_layout.addLayout(cam_layout)  
  
        # 置信度  
        conf_layout = QHBoxLayout()  
        conf_layout.addWidget(QLabel("阈值:"))  
        self.conf_spinbox = QSpinBox()  
        self.conf_spinbox.setRange(10, 95)  
        self.conf_spinbox.setValue(40)  
        self.conf_spinbox.setSuffix("%")  
        self.conf_spinbox.valueChanged.connect(self.update_confidence)  
        conf_layout.addWidget(self.conf_spinbox)  
        settings_layout.addLayout(conf_layout)  
  
        # --- 新增功能：ONNX 与 测速 ---        self.onnx_check = QCheckBox("启用 ONNX Runtime 加速")  
        self.onnx_check.setToolTip("使用 ONNX 格式推理。首次勾选会自动导出模型。")  
        settings_layout.addWidget(self.onnx_check)  
  
        self.realtime_check = QCheckBox("视频原速播放")  
        self.realtime_check.setChecked(True)  
        self.realtime_check.setToolTip("勾选：正常观看\n不勾选：全速跑（用于测试最大FPS）")  
        settings_layout.addWidget(self.realtime_check)  
        # ---------------------------  
  
        settings_group.setLayout(settings_layout)  
        control_panel.addWidget(settings_group)  
  
        # 3. 实时统计  
        stats_group = QGroupBox("检测数据")  
        stats_layout = QVBoxLayout()  
  
        self.fps_label = QLabel("FPS: --")  
        self.fps_label.setFont(QFont("Arial", 12, QFont.Bold))  
        self.fps_label.setStyleSheet("color: #00ff00;")  
        stats_layout.addWidget(self.fps_label)  
  
        self.detection_label = QLabel("当前目标: 0")  
        self.detection_label.setFont(QFont("Arial", 11))  
        stats_layout.addWidget(self.detection_label)  
  
        self.progress_label = QLabel("进度: --")  
        stats_layout.addWidget(self.progress_label)  
  
        self.source_label = QLabel("来源: 就绪")  
        self.source_label.setWordWrap(True)  
        self.source_label.setStyleSheet("color: #aaaaaa; font-size: 10px;")  
        stats_layout.addWidget(self.source_label)  
  
        stats_group.setLayout(stats_layout)  
        control_panel.addWidget(stats_group)  
  
        # 4. 类别列表  
        class_group = QGroupBox("类别统计")  
        class_layout = QVBoxLayout()  
  
        scroll = QScrollArea()  
        scroll.setWidgetResizable(True)  
        scroll.setStyleSheet("background-color: #2b2b2b; border: none;")  
  
        self.class_list_widget = QWidget()  
        self.class_list_layout = QVBoxLayout()  
        self.class_list_layout.setSpacing(2)  
        self.class_list_widget.setLayout(self.class_list_layout)  
  
        scroll.setWidget(self.class_list_widget)  
        class_layout.addWidget(scroll)  
        class_group.setLayout(class_layout)  
        control_panel.addWidget(class_group, 1)  
  
        main_layout.addLayout(control_panel, 1)  
  
        # 状态栏  
        self.status_bar = QStatusBar()  
        self.setStatusBar(self.status_bar)  
        self.status_bar.showMessage("系统初始化完成 - 等待操作")  
  
        # 变量初始化  
        self.thread = None  
        self.current_frame = None  
        self.class_labels = {}  
  
    # --- 核心控制函数 ---    def start_detection(self):  
        if self.thread and self.thread.isRunning(): return  
  
        # 获取参数  
        cam_id = self.camera_combo.currentIndex()  
        conf = self.conf_spinbox.value() / 100.0  
        use_onnx = self.onnx_check.isChecked()  
        realtime = self.realtime_check.isChecked()  
  
        # 启动线程  
        self.thread = DetectionThread(  
            source=cam_id,  
            source_type="camera",  
            confidence=conf,  
            use_onnx=use_onnx,  
            realtime_mode=realtime  
        )  
        self.run_thread("摄像头实时检测")  
  
    def upload_video(self):  
        path, _ = QFileDialog.getOpenFileName(self, "选择视频", "", "Videos (*.mp4 *.avi *.mkv)")  
        if path:  
            self.stop_detection()  
            conf = self.conf_spinbox.value() / 100.0  
            use_onnx = self.onnx_check.isChecked()  
            realtime = self.realtime_check.isChecked() # 获取复选框状态  
  
            self.thread = DetectionThread(  
                source=path,  
                source_type="video",  
                confidence=conf,  
                use_onnx=use_onnx,  
                realtime_mode=realtime  
            )  
            self.run_thread(f"视频: {os.path.basename(path)}")  
  
    def upload_image(self):  
        path, _ = QFileDialog.getOpenFileName(self, "选择图片", "", "Images (*.jpg *.png *.bmp)")  
        if path:  
            self.stop_detection()  
            conf = self.conf_spinbox.value() / 100.0  
            use_onnx = self.onnx_check.isChecked()  
  
            self.thread = DetectionThread(  
                source=path,  
                source_type="image",  
                confidence=conf,  
                use_onnx=use_onnx  
            )  
            self.run_thread(f"图片: {os.path.basename(path)}")  
  
    def run_thread(self, source_text):  
        self.setup_connections()  
        self.thread.start()  
        self.update_ui(detecting=True)  
        self.source_label.setText(source_text)  
        self.status_bar.showMessage(f"正在启动: {source_text} ...")  
  
    def stop_detection(self):  
        if self.thread:  
            self.thread.stop()  
            self.thread = None  
        self.update_ui(detecting=False)  
        self.status_bar.showMessage("检测已停止")  
        self.fps_label.setText("FPS: --")  
  
    def setup_connections(self):  
        self.thread.change_pixmap_signal.connect(self.update_image)  
        self.thread.fps_signal.connect(lambda x: self.fps_label.setText(f"FPS: {x:.1f}"))  
        self.thread.detection_signal.connect(lambda x: self.detection_label.setText(f"当前目标: {x}"))  
        self.thread.class_count_signal.connect(self.update_class_stats)  
        self.thread.error_signal.connect(lambda x: self.status_bar.showMessage(x))  
        self.thread.progress_signal.connect(self.update_progress)  
        self.thread.finished_signal.connect(self.on_finished)  
  
    # --- UI 更新逻辑 ---    def update_ui(self, detecting):  
        self.start_btn.setEnabled(not detecting)  
        self.upload_video_btn.setEnabled(not detecting)  
        self.upload_image_btn.setEnabled(not detecting)  
        self.camera_combo.setEnabled(not detecting)  
        # 检测中禁止切换加速模式，防止崩溃  
        self.onnx_check.setEnabled(not detecting)  
        self.realtime_check.setEnabled(not detecting)  
  
        self.stop_btn.setEnabled(detecting)  
        self.snapshot_btn.setEnabled(detecting or self.current_frame is not None)  
  
    def update_image(self, qt_img):  
        self.current_frame = qt_img  
        self.image_label.setPixmap(QPixmap.fromImage(qt_img))  
  
    def update_class_stats(self, counts):  
        # 清除旧标签  
        for w in self.class_labels.values(): w.deleteLater()  
        self.class_labels.clear()  
  
        if not counts: return  
  
        for name, count in sorted(counts.items(), key=lambda x: x[1], reverse=True):  
            frame = QFrame()  
            frame.setStyleSheet("background-color: #3b3b3b; border-radius: 4px; margin: 1px;")  
            layout = QHBoxLayout(frame)  
            layout.setContentsMargins(5, 2, 5, 2)  
  
            lbl_name = QLabel(name)  
            lbl_name.setStyleSheet("color: #14a085; font-weight: bold;")  
            lbl_count = QLabel(str(count))  
            lbl_count.setStyleSheet("background-color: #0d7377; border-radius: 8px; padding: 0 6px;")  
  
            layout.addWidget(lbl_name)  
            layout.addStretch()  
            layout.addWidget(lbl_count)  
  
            self.class_list_layout.addWidget(frame)  
            self.class_labels[name] = frame  
  
    def update_progress(self, curr, total):  
        if total > 0:  
            self.progress_label.setText(f"进度: {int(curr/total*100)}%")  
  
    def update_confidence(self, val):  
        if self.thread and self.thread.isRunning():  
            self.thread.update_confidence(val / 100.0)  
  
    def save_snapshot(self):  
        if self.current_frame:  
            name = f"snapshot_{datetime.now().strftime('%H%M%S')}.png"  
            self.current_frame.save(name)  
            self.status_bar.showMessage(f"截图已保存: {name}", 3000)  
  
    def on_finished(self):  
        self.status_bar.showMessage("任务完成")  
        self.update_ui(detecting=False)  
  
    def closeEvent(self, event):  
        self.stop_detection()  
        event.accept()  
  
if __name__ == "__main__":  
  
    multiprocessing.freeze_support()  # 新增这一行,打包前新增，防止windows多进程问题  
  
    app = QApplication(sys.argv)  
    window = MainWindow()  
    window.show()  
    sys.exit(app.exec_())
```