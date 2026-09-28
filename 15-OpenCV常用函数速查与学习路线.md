---
tags:
  - OpenCV
  - Python
  - 计算机视觉
  - 香橙派
aliases:
  - OpenCV常用函数
  - cv2速查表
  - OpenCV学习路线
created: 2026-08-16
updated: 2026-08-17
---

# OpenCV 常用函数速查与学习路线

> [!info] 这篇笔记有两种使用方式
> - 前面的第 1～26 节用于建立整体流程和快速查找。
> - **第 27 节是函数级详解**，包含调用格式、参数、返回值、常见坑和带中文注释的代码。第一次使用某个函数时，直接跳到第 27 节查阅。

> [!tip] 不需要把整个 OpenCV 背完
> OpenCV 有几千个接口，但日常视觉项目里，真正高频使用的只有几十个。先掌握“采集 → 预处理 → 提取目标 → 画结果 → 输出”这条主线，遇到特殊需求再查文档。

## 1. 一张图看懂学习顺序

```text
读取图片 / 摄像头
        ↓
查看尺寸、裁剪 ROI、颜色空间转换
        ↓
滤波降噪、阈值分割、形态学处理
        ↓
边缘 / 轮廓 / 色块 / 直线 / 圆
        ↓
计算位置、面积、角度、中心点
        ↓
绘制结果、保存视频、网页显示
```

推荐优先掌握：

1. `imread`、`imwrite`、`VideoCapture`
2. `resize`、`cvtColor`、NumPy 切片
3. `inRange`、`threshold`、`GaussianBlur`
4. `erode`、`dilate`、`morphologyEx`
5. `Canny`、`findContours`
6. `contourArea`、`boundingRect`、`moments`
7. `rectangle`、`circle`、`putText`

---

## 2. 基础模板

```python
import cv2
import numpy as np
```

OpenCV 读取的彩色图片默认通道顺序是 **BGR**，不是 RGB。

```python
image = cv2.imread("test.jpg")

if image is None:
    raise FileNotFoundError("图片读取失败")

print(image.shape)   # (高度, 宽度, 通道数)
print(image.dtype)   # 通常是 uint8
```

---

## 3. 图片读取、显示和保存

| 函数 | 用途 | 常用写法 |
|---|---|---|
| `cv2.imread()` | 读取图片 | `img = cv2.imread("a.jpg")` |
| `cv2.imwrite()` | 保存图片 | `cv2.imwrite("out.jpg", img)` |
| `cv2.imshow()` | 桌面窗口显示 | `cv2.imshow("image", img)` |
| `cv2.waitKey()` | 等待键盘输入 | `key = cv2.waitKey(1)` |
| `cv2.destroyAllWindows()` | 关闭全部窗口 | `cv2.destroyAllWindows()` |

读取模式：

```python
color = cv2.imread("a.jpg", cv2.IMREAD_COLOR)
gray = cv2.imread("a.jpg", cv2.IMREAD_GRAYSCALE)
unchanged = cv2.imread("a.png", cv2.IMREAD_UNCHANGED)
```

> [!warning] Remote SSH 注意
> 通过 VS Code Remote SSH 运行时，`cv2.imshow()` 通常不会在 Windows 上弹窗。香橙派项目应使用浏览器视频流、保存图片，或者接显示器运行。

---

## 4. 图像属性、复制和 ROI

```python
height, width = image.shape[:2]
channels = image.shape[2]
pixel_count = image.size
data_type = image.dtype

copy_image = image.copy()
```

NumPy 裁剪 ROI：

```python
# image[y1:y2, x1:x2]
roi = image[100:300, 200:500]
```

修改一个区域：

```python
image[100:200, 100:300] = (0, 0, 255)  # BGR 红色
```

> [!important] 坐标顺序
> OpenCV 绘图坐标写作 `(x, y)`，但 NumPy 数组索引写作 `[y, x]`。

通道拆分与合并：

```python
b, g, r = cv2.split(image)
merged = cv2.merge([b, g, r])
```

高性能场景也可以直接使用：

```python
b = image[:, :, 0]
g = image[:, :, 1]
r = image[:, :, 2]
```

---

## 5. 尺寸、翻转和几何变换

### 5.1 缩放

```python
small = cv2.resize(image, (640, 480))

# 按比例缩放
half = cv2.resize(
    image,
    None,
    fx=0.5,
    fy=0.5,
    interpolation=cv2.INTER_AREA,
)
```

常见插值：

| 插值 | 适用情况 |
|---|---|
| `cv2.INTER_AREA` | 缩小图片 |
| `cv2.INTER_LINEAR` | 普通放大，默认选项 |
| `cv2.INTER_CUBIC` | 高质量放大，但更慢 |
| `cv2.INTER_NEAREST` | 标签图、掩膜，不产生新颜色 |

### 5.2 翻转和旋转

```python
horizontal = cv2.flip(image, 1)   # 水平翻转
vertical = cv2.flip(image, 0)     # 垂直翻转
both = cv2.flip(image, -1)        # 水平和垂直

rotated_90 = cv2.rotate(image, cv2.ROTATE_90_CLOCKWISE)
```

任意角度旋转：

```python
h, w = image.shape[:2]
matrix = cv2.getRotationMatrix2D((w / 2, h / 2), 30, 1.0)
rotated = cv2.warpAffine(image, matrix, (w, h))
```

### 5.3 透视变换

```python
src = np.float32([[100, 100], [500, 100], [80, 400], [520, 400]])
dst = np.float32([[0, 0], [640, 0], [0, 480], [640, 480]])

matrix = cv2.getPerspectiveTransform(src, dst)
result = cv2.warpPerspective(image, matrix, (640, 480))
```

---

## 6. 颜色空间转换

```python
gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
hsv = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)
rgb = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
lab = cv2.cvtColor(image, cv2.COLOR_BGR2LAB)
```

| 颜色空间 | 常见用途 |
|---|---|
| BGR | OpenCV 默认彩色图 |
| RGB | Matplotlib、部分深度学习模型 |
| GRAY | 边缘、阈值、轮廓 |
| HSV | 色块识别，受亮度影响较小 |
| LAB | 颜色校正、亮度增强 |

HSV 范围（OpenCV）：

```text
H: 0~179
S: 0~255
V: 0~255
```

---

## 7. 绘制图形和文字

```python
canvas = image.copy()

cv2.line(canvas, (10, 10), (300, 200), (255, 0, 0), 2)
cv2.rectangle(canvas, (100, 100), (300, 250), (0, 255, 0), 2)
cv2.circle(canvas, (320, 240), 50, (0, 0, 255), 2)

points = np.array([[100, 100], [200, 80], [250, 180]], dtype=np.int32)
cv2.polylines(canvas, [points], True, (0, 255, 255), 2)

cv2.putText(
    canvas,
    "target",
    (100, 80),
    cv2.FONT_HERSHEY_SIMPLEX,
    0.8,
    (0, 255, 0),
    2,
    cv2.LINE_AA,
)
```

颜色示例（BGR）：

```python
BLUE = (255, 0, 0)
GREEN = (0, 255, 0)
RED = (0, 0, 255)
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
```

---

## 8. 掩膜与位运算

创建掩膜：

```python
mask = np.zeros(image.shape[:2], dtype=np.uint8)
cv2.circle(mask, (320, 240), 100, 255, -1)

masked = cv2.bitwise_and(image, image, mask=mask)
```

常见函数：

| 函数 | 用途 |
|---|---|
| `cv2.inRange()` | 按颜色范围生成二值掩膜 |
| `cv2.bitwise_and()` | 保留共同区域 |
| `cv2.bitwise_or()` | 合并区域 |
| `cv2.bitwise_xor()` | 保留不同区域 |
| `cv2.bitwise_not()` | 黑白反转 |

HSV 色块掩膜：

```python
hsv = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)
lower = np.array([35, 80, 80])
upper = np.array([85, 255, 255])
mask = cv2.inRange(hsv, lower, upper)
```

---

## 9. 滤波与降噪

```python
mean = cv2.blur(image, (5, 5))
gaussian = cv2.GaussianBlur(image, (5, 5), 0)
median = cv2.medianBlur(image, 5)
bilateral = cv2.bilateralFilter(image, 9, 75, 75)
```

| 函数 | 特点 | 常见场景 |
|---|---|---|
| `blur` | 简单平均，速度快 | 普通降噪 |
| `GaussianBlur` | 自然、稳定 | 边缘检测前预处理 |
| `medianBlur` | 去椒盐噪声效果好 | 二值图、脉冲噪声 |
| `bilateralFilter` | 保边降噪，但较慢 | 需要保留边缘的图像 |

卷积：

```python
kernel = np.array([[0, -1, 0], [-1, 5, -1], [0, -1, 0]], dtype=np.float32)
sharpened = cv2.filter2D(image, -1, kernel)
```

---

## 10. 阈值分割

### 10.1 固定阈值

```python
gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
_, binary = cv2.threshold(gray, 127, 255, cv2.THRESH_BINARY)
```

常见模式：

```text
cv2.THRESH_BINARY
cv2.THRESH_BINARY_INV
cv2.THRESH_TRUNC
cv2.THRESH_TOZERO
```

### 10.2 Otsu 自动阈值

```python
_, binary = cv2.threshold(
    gray,
    0,
    255,
    cv2.THRESH_BINARY + cv2.THRESH_OTSU,
)
```

### 10.3 自适应阈值

```python
binary = cv2.adaptiveThreshold(
    gray,
    255,
    cv2.ADAPTIVE_THRESH_GAUSSIAN_C,
    cv2.THRESH_BINARY,
    11,
    2,
)
```

光照不均时，优先尝试自适应阈值。

---

## 11. 形态学处理

```python
kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (5, 5))

eroded = cv2.erode(mask, kernel, iterations=1)
dilated = cv2.dilate(mask, kernel, iterations=1)
opened = cv2.morphologyEx(mask, cv2.MORPH_OPEN, kernel)
closed = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, kernel)
gradient = cv2.morphologyEx(mask, cv2.MORPH_GRADIENT, kernel)
```

记忆方法：

| 操作 | 效果 |
|---|---|
| 腐蚀 `erode` | 白色区域变小，去小白点 |
| 膨胀 `dilate` | 白色区域变大，连接断点 |
| 开运算 `MORPH_OPEN` | 先腐蚀后膨胀，去小噪点 |
| 闭运算 `MORPH_CLOSE` | 先膨胀后腐蚀，填小孔洞 |

---

## 12. 边缘检测

```python
gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
blurred = cv2.GaussianBlur(gray, (5, 5), 0)
edges = cv2.Canny(blurred, 50, 150)
```

其他梯度函数：

```python
sobel_x = cv2.Sobel(gray, cv2.CV_64F, 1, 0, ksize=3)
sobel_y = cv2.Sobel(gray, cv2.CV_64F, 0, 1, ksize=3)
laplacian = cv2.Laplacian(gray, cv2.CV_64F)
```

一般项目先掌握 `Canny` 即可。

---

## 13. 轮廓检测与目标测量

```python
contours, hierarchy = cv2.findContours(
    binary,
    cv2.RETR_EXTERNAL,
    cv2.CHAIN_APPROX_SIMPLE,
)

for contour in contours:
    area = cv2.contourArea(contour)
    if area < 500:
        continue

    perimeter = cv2.arcLength(contour, True)
    x, y, w, h = cv2.boundingRect(contour)

    cv2.rectangle(image, (x, y), (x + w, y + h), (0, 255, 0), 2)
```

高频函数：

| 函数 | 用途 |
|---|---|
| `findContours` | 找轮廓 |
| `drawContours` | 绘制轮廓 |
| `contourArea` | 计算面积 |
| `arcLength` | 计算周长 |
| `boundingRect` | 水平外接矩形 |
| `minAreaRect` | 最小旋转矩形 |
| `minEnclosingCircle` | 最小外接圆 |
| `approxPolyDP` | 多边形拟合 |
| `moments` | 计算中心点等矩特征 |

轮廓中心：

```python
m = cv2.moments(contour)

if m["m00"] != 0:
    center_x = int(m["m10"] / m["m00"])
    center_y = int(m["m01"] / m["m00"])
    cv2.circle(image, (center_x, center_y), 5, (0, 0, 255), -1)
```

形状近似：

```python
perimeter = cv2.arcLength(contour, True)
polygon = cv2.approxPolyDP(contour, 0.02 * perimeter, True)

corners = len(polygon)
```

---

## 14. 色块识别完整例程

```python
import cv2
import numpy as np

cap = cv2.VideoCapture(0, cv2.CAP_V4L2)

if not cap.isOpened():
    raise RuntimeError("无法打开 /dev/video0")

cap.set(cv2.CAP_PROP_FOURCC, cv2.VideoWriter_fourcc("M", "J", "P", "G"))
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 1280)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 720)
cap.set(cv2.CAP_PROP_FPS, 30)

lower_green = np.array([35, 80, 80])
upper_green = np.array([85, 255, 255])
kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (5, 5))

try:
    while True:
        ret, frame = cap.read()
        if not ret:
            print("读取画面失败")
            break

        hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
        mask = cv2.inRange(hsv, lower_green, upper_green)
        mask = cv2.morphologyEx(mask, cv2.MORPH_OPEN, kernel)
        mask = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, kernel)

        contours, _ = cv2.findContours(
            mask,
            cv2.RETR_EXTERNAL,
            cv2.CHAIN_APPROX_SIMPLE,
        )

        if contours:
            target = max(contours, key=cv2.contourArea)
            area = cv2.contourArea(target)

            if area > 500:
                x, y, w, h = cv2.boundingRect(target)
                center = (x + w // 2, y + h // 2)

                cv2.rectangle(frame, (x, y), (x + w, y + h), (0, 255, 0), 2)
                cv2.circle(frame, center, 5, (0, 0, 255), -1)
                cv2.putText(
                    frame,
                    f"center={center} area={area:.0f}",
                    (x, max(30, y - 10)),
                    cv2.FONT_HERSHEY_SIMPLEX,
                    0.7,
                    (0, 255, 0),
                    2,
                )

        # 接显示器时可以使用：
        cv2.imshow("frame", frame)
        cv2.imshow("mask", mask)

        if cv2.waitKey(1) & 0xFF == ord("q"):
            break
finally:
    cap.release()
    cv2.destroyAllWindows()
```

色块识别的核心只有五步：

```text
BGR 转 HSV → inRange → 形态学降噪 → findContours → 计算位置
```

---

## 15. 直线和圆检测

### 15.1 霍夫直线

```python
lines = cv2.HoughLinesP(
    edges,
    rho=1,
    theta=np.pi / 180,
    threshold=80,
    minLineLength=50,
    maxLineGap=10,
)

if lines is not None:
    for line in lines:
        x1, y1, x2, y2 = line[0]
        cv2.line(image, (x1, y1), (x2, y2), (0, 255, 0), 2)
```

### 15.2 霍夫圆

```python
circles = cv2.HoughCircles(
    gray,
    cv2.HOUGH_GRADIENT,
    dp=1.2,
    minDist=50,
    param1=100,
    param2=30,
    minRadius=10,
    maxRadius=100,
)
```

实际项目中，如果物体轮廓清晰，也可以用 `findContours` 加 `minEnclosingCircle`，通常更容易调参。

---

## 16. 图像运算、差分和融合

```python
added = cv2.add(image1, image2)
subtracted = cv2.subtract(image1, image2)
difference = cv2.absdiff(image1, image2)

blended = cv2.addWeighted(image1, 0.7, image2, 0.3, 0)
```

简单运动检测：

```python
diff = cv2.absdiff(previous_frame, current_frame)
gray = cv2.cvtColor(diff, cv2.COLOR_BGR2GRAY)
_, motion_mask = cv2.threshold(gray, 25, 255, cv2.THRESH_BINARY)
```

---

## 17. 直方图与对比度

```python
hist = cv2.calcHist([gray], [0], None, [256], [0, 256])
equalized = cv2.equalizeHist(gray)
```

局部对比度增强 CLAHE：

```python
clahe = cv2.createCLAHE(clipLimit=2.0, tileGridSize=(8, 8))
enhanced = clahe.apply(gray)
```

---

## 18. 模板匹配

```python
result = cv2.matchTemplate(image, template, cv2.TM_CCOEFF_NORMED)
_, max_value, _, max_location = cv2.minMaxLoc(result)

if max_value > 0.8:
    h, w = template.shape[:2]
    x, y = max_location
    cv2.rectangle(image, (x, y), (x + w, y + h), (0, 255, 0), 2)
```

模板匹配对旋转、缩放和光照变化比较敏感，适合目标大小和角度基本固定的场景。

---

## 19. 连通区域分析

```python
count, labels, stats, centers = cv2.connectedComponentsWithStats(binary)

for label in range(1, count):
    x, y, w, h, area = stats[label]
    center_x, center_y = centers[label]
```

对于二值图中的多个独立色块，连通区域分析有时比轮廓更直接。

---

## 20. 特征点与匹配（进阶）

角点：

```python
corners = cv2.goodFeaturesToTrack(
    gray,
    maxCorners=100,
    qualityLevel=0.01,
    minDistance=10,
)
```

ORB 特征：

```python
orb = cv2.ORB_create(nfeatures=500)
keypoints, descriptors = orb.detectAndCompute(gray, None)

output = cv2.drawKeypoints(image, keypoints, None, color=(0, 255, 0))
```

特征匹配：

```python
matcher = cv2.BFMatcher(cv2.NORM_HAMMING, crossCheck=True)
matches = matcher.match(descriptors1, descriptors2)
matches = sorted(matches, key=lambda item: item.distance)
```

刚开始学色块和轮廓时，可以先跳过本节。

---

## 21. 摄像头和视频

### 21.1 摄像头采集

```python
cap = cv2.VideoCapture(0, cv2.CAP_V4L2)

if not cap.isOpened():
    raise RuntimeError("摄像头打开失败")

while True:
    ret, frame = cap.read()
    if not ret:
        break

cap.release()
```

常用参数：

```python
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 1280)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 720)
cap.set(cv2.CAP_PROP_FPS, 30)
cap.set(cv2.CAP_PROP_FOURCC, cv2.VideoWriter_fourcc("M", "J", "P", "G"))

actual_width = cap.get(cv2.CAP_PROP_FRAME_WIDTH)
actual_height = cap.get(cv2.CAP_PROP_FRAME_HEIGHT)
actual_fps = cap.get(cv2.CAP_PROP_FPS)
```

> [!important] `set()` 只是提出请求
> 摄像头不一定接受指定参数。设置后必须用 `get()` 检查实际结果。标准 720p 是 `1280×720`，标准 1080p 是 `1920×1080`。

Linux 查看摄像头支持格式：

```bash
v4l2-ctl --list-devices
v4l2-ctl -d /dev/video0 --list-formats-ext
```

### 21.2 保存视频

```python
fourcc = cv2.VideoWriter_fourcc("M", "J", "P", "G")
writer = cv2.VideoWriter("output.avi", fourcc, 30, (1280, 720))

writer.write(frame)

writer.release()
```

`VideoWriter` 的尺寸必须与写入帧的尺寸一致。

---

## 22. 香橙派浏览器显示

如果项目中继续使用 `xbhdcc_tools.WebStreamer`：

```python
from xbhdcc_tools import WebStreamer

streamer = WebStreamer(port=8081)

while True:
    ret, frame = cap.read()
    if not ret:
        break

    # 左侧显示原图
    streamer.update_frame(0, frame)

    # 右侧显示处理结果
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    gray_bgr = cv2.cvtColor(gray, cv2.COLOR_GRAY2BGR)
    streamer.update_frame(1, gray_bgr)
```

浏览器访问 VS Code 转发的 `localhost:8081`，或在局域网访问 `http://192.168.5.12:8081`。

一只摄像头通常只能由一个程序独占打开。运行学习脚本前，确认 WebRTC 服务没有占用：

```bash
fuser -v /dev/video0
```

---

## 23. 常见错误排查

### 图片读取为 `None`

```python
image = cv2.imread(path)
print(path, image is None)
```

检查路径、中文文件名、当前工作目录和权限。

### 摄像头打不开

```bash
ls -l /dev/video*
v4l2-ctl --list-devices
fuser -v /dev/video0
```

常见原因：设备编号错误、摄像头被其他进程占用、USB 断开、用户不在 `video` 组。

### 画面延迟越来越大

- 降低分辨率或帧率。
- 使用摄像头的 MJPEG 格式。
- 每帧处理不要堆积任务。
- 只显示最新帧，不要建立无限队列。

### 颜色识别不稳定

- 使用 HSV，不要直接比较 BGR。
- 在真实光照下重新标定上下限。
- 先模糊，再 `inRange`。
- 使用开运算和闭运算清理掩膜。
- 固定曝光和白平衡（摄像头支持时）。

### 代码能运行但 VS Code 报红

- 检查选择的 Python 解释器。
- 确认 OpenCV 安装在同一个虚拟环境。
- Pylance 类型提示偶尔会对 OpenCV 的动态接口误报。
- 运行错误以终端 traceback 为准，编辑器提示用于辅助检查。

---

## 24. 高频常量速查

```python
# 颜色转换
cv2.COLOR_BGR2GRAY
cv2.COLOR_BGR2HSV
cv2.COLOR_BGR2RGB
cv2.COLOR_GRAY2BGR

# 阈值
cv2.THRESH_BINARY
cv2.THRESH_BINARY_INV
cv2.THRESH_OTSU

# 形态学
cv2.MORPH_OPEN
cv2.MORPH_CLOSE
cv2.MORPH_GRADIENT
cv2.MORPH_RECT
cv2.MORPH_ELLIPSE

# 轮廓
cv2.RETR_EXTERNAL
cv2.RETR_LIST
cv2.RETR_TREE
cv2.CHAIN_APPROX_SIMPLE
cv2.CHAIN_APPROX_NONE

# 摄像头
cv2.CAP_V4L2
cv2.CAP_PROP_FRAME_WIDTH
cv2.CAP_PROP_FRAME_HEIGHT
cv2.CAP_PROP_FPS
cv2.CAP_PROP_FOURCC
```

---

## 25. 建议练习顺序

### 第一阶段：图片基础

- [ ] 读取、缩放并保存图片
- [ ] BGR 转灰度、HSV
- [ ] 裁剪 ROI
- [ ] 绘制矩形、圆和文字

### 第二阶段：二值图

- [ ] 固定阈值和 Otsu 阈值
- [ ] HSV 识别单一色块
- [ ] 开运算去噪、闭运算填孔

### 第三阶段：目标测量

- [ ] 找最大轮廓
- [ ] 计算面积和中心点
- [ ] 绘制外接矩形
- [ ] 判断三角形、矩形和圆

### 第四阶段：实时摄像头

- [ ] 读取 USB 摄像头
- [ ] 显示实际分辨率和 FPS
- [ ] 在浏览器同时显示原图和处理结果
- [ ] 记录处理后的视频

### 第五阶段：小项目

- [ ] 绿色色块跟踪
- [ ] 运动检测
- [ ] 圆形目标检测
- [ ] 十字线或赛道边缘检测
- [ ] 串口输出目标中心坐标

---

## 26. 查函数的正确方法

不记得参数时，在 Python 中查看：

```python
help(cv2.resize)
help(cv2.VideoCapture)
```

VS Code 中：

- 输入 `cv2.` 查看补全。
- 鼠标悬停函数查看参数。
- 按 `Ctrl+Space` 手动调出补全。
- 按 `F12` 查看类型定义。

搜索时使用“任务 + OpenCV”，例如：

```text
OpenCV HSV color tracking Python
OpenCV find largest contour
OpenCV perspective transform four points
```

> [!summary] 最终记住这件事
> 学 OpenCV 不是背 API，而是学会把视觉任务拆成步骤。先观察原图，再逐步输出 `gray`、`mask`、`edges` 和 `contours` 等中间结果；哪一步不对，就只调那一步。

---

## 27. 常用函数逐个详解：参数、返回值与注释示例

> [!note] 阅读约定
> 下面写法以 OpenCV 4.x 的 Python 接口为准。签名中的方括号表示可选参数；示例里的 `image` 一般表示 BGR 彩色图，`gray` 表示单通道灰度图，`mask` 或 `binary` 表示只有 0 和 255 的二值图。

### 27.1 `cv2.imread()`：读取图片

调用格式：

```python
image = cv2.imread(filename, flags)
```

参数：

| 参数 | 类型 | 说明 |
|---|---|---|
| `filename` | `str` | 图片路径，可以是相对路径或绝对路径 |
| `flags` | `int` | 读取模式，省略时为 `cv2.IMREAD_COLOR` |

常用 `flags`：

| 常量 | 效果 |
|---|---|
| `cv2.IMREAD_COLOR` | 读取为 BGR 三通道彩色图，不保留透明通道 |
| `cv2.IMREAD_GRAYSCALE` | 直接读取为单通道灰度图 |
| `cv2.IMREAD_UNCHANGED` | 保留原始通道，包括 PNG 的 Alpha 透明通道 |

返回值：读取成功时返回 NumPy 数组，失败时返回 `None`，失败时不会自动抛异常。

```python
import cv2

# 使用绝对路径最不容易受“当前工作目录”影响
image_path = "/root/opencv/images/test.jpg"

# 以默认 BGR 彩色格式读取图片
image = cv2.imread(image_path, cv2.IMREAD_COLOR)

# imread 失败时返回 None，所以必须主动检查
if image is None:
    raise FileNotFoundError(f"无法读取图片：{image_path}")

# shape 的顺序是：高度、宽度、通道数
height, width, channels = image.shape
print(f"尺寸={width}x{height}，通道数={channels}")
```

常见坑：

- 相对路径相对于“程序当前工作目录”，不一定相对于 Python 文件所在目录。
- OpenCV 默认返回 BGR，而不是 RGB。
- 文件不存在、权限不足或格式损坏时，返回 `None`。

### 27.2 `cv2.imwrite()`：保存图片

调用格式：

```python
success = cv2.imwrite(filename, image, params)
```

参数：

| 参数 | 说明 |
|---|---|
| `filename` | 保存路径，扩展名决定编码格式，如 `.jpg`、`.png` |
| `image` | 要保存的 NumPy 图像数组 |
| `params` | 可选的编码参数列表，格式为“参数常量, 数值” |

返回值：保存成功为 `True`，失败为 `False`。

```python
# JPEG 质量范围通常为 0～100，数值越大质量越高、文件越大
jpeg_params = [cv2.IMWRITE_JPEG_QUALITY, 95]
jpeg_ok = cv2.imwrite("result.jpg", image, jpeg_params)

# PNG 压缩等级范围为 0～9，数值越大压缩越强、保存越慢
png_params = [cv2.IMWRITE_PNG_COMPRESSION, 3]
png_ok = cv2.imwrite("result.png", image, png_params)

if not jpeg_ok or not png_ok:
    print("图片保存失败，请检查目录是否存在以及是否有写权限")
```

### 27.3 `cv2.imshow()`、`cv2.waitKey()`：桌面窗口显示

调用格式：

```python
cv2.imshow(window_name, image)
key_code = cv2.waitKey(delay_ms)
```

参数：

- `window_name`：窗口名称；同名窗口会更新画面，不会重复创建。
- `image`：要显示的图像。
- `delay_ms`：等待键盘输入的毫秒数；`0` 表示一直等待。

```python
while True:
    # 把当前帧显示到名为 camera 的窗口
    cv2.imshow("camera", frame)

    # 等待 1 ms，并取键值低 8 位，兼容不同平台
    key = cv2.waitKey(1) & 0xFF

    # 按 q 退出循环
    if key == ord("q"):
        break

# 退出前销毁 OpenCV 创建的全部 GUI 窗口
cv2.destroyAllWindows()
```

> [!warning] 香橙派 Remote SSH
> 远程 SSH 环境通常没有可用的桌面显示变量，`imshow()` 可能报错或不弹窗。此时用浏览器视频流、保存图片，或者接显示器后在图形桌面会话运行。

### 27.4 `cv2.VideoCapture()`：打开摄像头或视频文件

摄像头调用格式：

```python
cap = cv2.VideoCapture(index, api_preference)
```

视频文件调用格式：

```python
cap = cv2.VideoCapture(filename)
```

参数：

| 参数 | 说明 |
|---|---|
| `index` | 摄像头编号，`0` 通常对应 `/dev/video0` |
| `api_preference` | 指定视频后端；Linux 常用 `cv2.CAP_V4L2` |
| `filename` | 视频文件路径或网络流地址 |

```python
import cv2

# 在 Linux 上用 V4L2 后端打开第 0 个摄像头
cap = cv2.VideoCapture(0, cv2.CAP_V4L2)

# 构造对象不代表一定打开成功，必须检查
if not cap.isOpened():
    raise RuntimeError("无法打开摄像头 /dev/video0")

try:
    while True:
        # read() 返回两个值：是否成功、当前帧
        ret, frame = cap.read()

        # 摄像头断开或读取失败时 ret 为 False
        if not ret:
            print("读取摄像头帧失败")
            break

        # 在这里编写每一帧的 OpenCV 处理代码
        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
finally:
    # 即使程序异常，也释放摄像头句柄
    cap.release()
```

常用方法：

| 方法 | 返回值 | 用途 |
|---|---|---|
| `cap.isOpened()` | `bool` | 判断设备或视频是否打开成功 |
| `cap.read()` | `(bool, frame)` | 解码并读取下一帧 |
| `cap.grab()` | `bool` | 只抓取帧，不立即解码 |
| `cap.retrieve()` | `(bool, frame)` | 解码最近 `grab()` 的帧 |
| `cap.set(id, value)` | `bool` | 请求设置参数 |
| `cap.get(id)` | `float` | 查询参数实际值 |
| `cap.release()` | 无 | 释放设备或文件 |

### 27.5 `cap.set()`、`cap.get()`：设置和读取摄像头参数

调用格式：

```python
accepted = cap.set(property_id, value)
actual_value = cap.get(property_id)
```

常用属性：

| 属性 | 含义 |
|---|---|
| `cv2.CAP_PROP_FRAME_WIDTH` | 画面宽度 |
| `cv2.CAP_PROP_FRAME_HEIGHT` | 画面高度 |
| `cv2.CAP_PROP_FPS` | 帧率 |
| `cv2.CAP_PROP_FOURCC` | 像素/压缩格式 |
| `cv2.CAP_PROP_BRIGHTNESS` | 亮度，是否支持取决于摄像头 |
| `cv2.CAP_PROP_EXPOSURE` | 曝光，范围和含义取决于驱动 |
| `cv2.CAP_PROP_AUTO_EXPOSURE` | 自动曝光，具体数值因后端而异 |

```python
# 用四个明确字符创建 MJPG 的 FOURCC 编码值
fourcc = cv2.VideoWriter_fourcc("M", "J", "P", "G")

# 先请求 MJPG，再请求分辨率和帧率
cap.set(cv2.CAP_PROP_FOURCC, fourcc)
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 1280)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 720)
cap.set(cv2.CAP_PROP_FPS, 30)

# set() 成功不代表硬件最终采用该值，所以用 get() 验证
actual_width = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
actual_height = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))
actual_fps = cap.get(cv2.CAP_PROP_FPS)

print(f"实际参数：{actual_width}x{actual_height} @ {actual_fps:.1f} FPS")
```

### 27.6 `cv2.VideoWriter()`：保存视频

调用格式：

```python
writer = cv2.VideoWriter(filename, fourcc, fps, frame_size, is_color)
```

参数：

| 参数 | 说明 |
|---|---|
| `filename` | 输出文件名，如 `output.avi` |
| `fourcc` | 视频编码的四字符编码值 |
| `fps` | 输出视频帧率 |
| `frame_size` | `(宽度, 高度)`，注意不是 `(高度, 宽度)` |
| `is_color` | `True` 表示彩色视频；通常可省略 |

```python
# 创建 MJPG 编码器
fourcc = cv2.VideoWriter_fourcc("M", "J", "P", "G")

# 输出尺寸必须和 writer.write() 收到的每一帧完全一致
writer = cv2.VideoWriter(
    "output.avi",   # 输出文件
    fourcc,          # 编码格式
    30.0,            # 视频帧率
    (1280, 720),     # 宽度、高度
    True,            # 彩色视频
)

if not writer.isOpened():
    raise RuntimeError("VideoWriter 创建失败")

# frame 必须是 1280x720 的 BGR 图像
writer.write(frame)

# 所有帧写完后释放并完成文件封装
writer.release()
```

### 27.7 `cv2.resize()`：缩放图片

调用格式：

```python
dst = cv2.resize(src, dsize, fx=0, fy=0, interpolation=cv2.INTER_LINEAR)
```

参数：

| 参数 | 说明 |
|---|---|
| `src` | 输入图像 |
| `dsize` | 目标 `(宽度, 高度)`；设为 `None` 时使用比例 |
| `fx`、`fy` | 横向和纵向缩放比例 |
| `interpolation` | 插值算法 |

```python
# 直接指定输出宽度和高度
fixed = cv2.resize(image, (640, 480), interpolation=cv2.INTER_AREA)

# 按 50% 比例缩小，dsize 使用 None
half = cv2.resize(
    image,
    None,
    fx=0.5,
    fy=0.5,
    interpolation=cv2.INTER_AREA,
)

# 按目标宽度等比例缩放，避免图像变形
target_width = 640
height, width = image.shape[:2]
scale = target_width / width
target_height = int(height * scale)
keep_ratio = cv2.resize(image, (target_width, target_height))
```

常见坑：`dsize` 顺序是 `(宽度, 高度)`，而 `shape[:2]` 顺序是 `(高度, 宽度)`。

### 27.8 `cv2.cvtColor()`：转换颜色空间

调用格式：

```python
dst = cv2.cvtColor(src, conversion_code)
```

```python
# BGR 三通道转灰度单通道
gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)

# BGR 转 HSV，常用于颜色分割
hsv = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)

# BGR 转 RGB，给 Matplotlib 或某些模型使用
rgb = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)

# 灰度转回三通道；颜色仍是灰色，但 shape 变为 HxWx3
gray_bgr = cv2.cvtColor(gray, cv2.COLOR_GRAY2BGR)
```

常见坑：不能对单通道灰度图再次使用 `COLOR_BGR2GRAY`，否则通道数量不匹配。

### 27.9 `cv2.split()`、`cv2.merge()`：拆分和合并通道

```python
# OpenCV 的三通道顺序为 B、G、R
blue, green, red = cv2.split(image)

# 合并时传入通道列表，顺序决定输出通道含义
merged = cv2.merge([blue, green, red])

# 构造只保留红色通道的图像
zeros = np.zeros_like(red)
red_only = cv2.merge([zeros, zeros, red])
```

参数与返回值：

- `split(m)` 接收多通道数组，返回各通道组成的元组。
- `merge(mv)` 接收尺寸相同、类型相同的单通道数组列表，返回多通道数组。

### 27.10 NumPy ROI：裁剪图像

ROI 不是 OpenCV 函数，但实际使用频率非常高：

```python
# OpenCV 坐标通常写 (x, y)，NumPy 索引却写 [y, x]
x1, y1 = 100, 80
x2, y2 = 500, 380

# 裁剪范围：先写 y，再写 x；右边界和下边界不包含在结果中
roi = image[y1:y2, x1:x2]

# 切片得到的是视图；如果要独立修改，使用 copy()
roi_copy = image[y1:y2, x1:x2].copy()
```

### 27.11 `cv2.flip()`、`cv2.rotate()`：翻转和直角旋转

```python
# flipCode=1：沿竖直轴翻转，也就是左右镜像
horizontal = cv2.flip(image, 1)

# flipCode=0：沿水平轴翻转，也就是上下颠倒
vertical = cv2.flip(image, 0)

# flipCode=-1：上下和左右同时翻转
both = cv2.flip(image, -1)

# 顺时针旋转 90 度
clockwise_90 = cv2.rotate(image, cv2.ROTATE_90_CLOCKWISE)

# 旋转 180 度
rotated_180 = cv2.rotate(image, cv2.ROTATE_180)
```

### 27.12 `getRotationMatrix2D()`、`warpAffine()`：任意角度旋转

```python
# 获取原图高度和宽度
height, width = image.shape[:2]

# 旋转中心使用图像中心坐标
center = (width / 2, height / 2)

# angle=30 表示逆时针 30 度；scale=1.0 表示不缩放
matrix = cv2.getRotationMatrix2D(center, 30, 1.0)

# 根据 2x3 仿射矩阵生成输出图像
rotated = cv2.warpAffine(
    image,                         # 输入图像
    matrix,                        # 2x3 变换矩阵
    (width, height),               # 输出尺寸：宽、高
    flags=cv2.INTER_LINEAR,        # 插值方式
    borderMode=cv2.BORDER_CONSTANT,
    borderValue=(0, 0, 0),         # 旋转后空白区域填黑色
)
```

注意：固定输出尺寸时，旋转后的四角可能被裁掉。

### 27.13 `getPerspectiveTransform()`、`warpPerspective()`：透视矫正

```python
# 原图中按“左上、右上、左下、右下”准备四个点
source_points = np.float32([
    [120, 80],
    [520, 100],
    [80, 420],
    [560, 400],
])

# 把四边形映射为 640x480 的规则矩形
destination_points = np.float32([
    [0, 0],
    [639, 0],
    [0, 479],
    [639, 479],
])

# 计算 3x3 透视变换矩阵
matrix = cv2.getPerspectiveTransform(source_points, destination_points)

# 应用透视变换；dsize 仍然是“宽、高”
warped = cv2.warpPerspective(image, matrix, (640, 480))
```

四个点的对应顺序必须一致，否则画面会翻折或旋转。

### 27.14 `cv2.line()`、`rectangle()`、`circle()`：基础绘图

```python
# 在原图副本上绘制，避免破坏原始数据
result = image.copy()

# line(图像, 起点, 终点, BGR颜色, 线宽, 线型)
cv2.line(result, (50, 50), (300, 200), (255, 0, 0), 2, cv2.LINE_AA)

# rectangle(图像, 左上角, 右下角, 颜色, 线宽)
cv2.rectangle(result, (100, 100), (400, 300), (0, 255, 0), 2)

# thickness=-1 表示填充整个矩形
cv2.rectangle(result, (10, 10), (80, 50), (0, 0, 255), -1)

# circle(图像, 圆心, 半径, 颜色, 线宽)
cv2.circle(result, (320, 240), 60, (0, 255, 255), 3)

# thickness=-1 表示实心圆，常用于标记中心点
cv2.circle(result, (320, 240), 5, (0, 0, 255), -1)
```

所有坐标使用 `(x, y)`；颜色使用 `(B, G, R)`。

### 27.15 `cv2.polylines()`：绘制多边形

```python
# 点数组通常使用 int32，形状可为 Nx2
points = np.array(
    [[100, 100], [250, 80], [320, 220], [150, 260]],
    dtype=np.int32,
)

cv2.polylines(
    image,             # 要绘制的图像
    [points],          # 多边形列表，所以外面再套一层列表
    True,              # True 表示连接最后一点与第一点
    (0, 255, 0),       # BGR 绿色
    2,                 # 线宽
    cv2.LINE_AA,       # 抗锯齿线型
)
```

填充多边形使用：

```python
cv2.fillPoly(image, [points], (0, 255, 0))
```

### 27.16 `cv2.putText()`：绘制文字

调用格式：

```python
cv2.putText(image, text, origin, font, font_scale, color, thickness, line_type)
```

```python
# origin 是文字左下角，而不是左上角
cv2.putText(
    image,
    "Target: 1",                   # 文字内容；内置字体不支持中文
    (50, 80),                      # 文字基线左下角坐标
    cv2.FONT_HERSHEY_SIMPLEX,       # 字体
    0.8,                           # 字体缩放比例
    (0, 255, 0),                   # BGR 绿色
    2,                             # 笔画粗细
    cv2.LINE_AA,                   # 抗锯齿
)
```

需要中文时，通常使用 Pillow 的中文字体绘制，再转回 NumPy 数组。

### 27.17 `cv2.inRange()`：按颜色范围生成掩膜

调用格式：

```python
mask = cv2.inRange(src, lower_bound, upper_bound)
```

参数：

- `src`：输入图像，色块识别通常传 HSV 图。
- `lower_bound`：每个通道的下限，包含边界。
- `upper_bound`：每个通道的上限，包含边界。

返回值：单通道 `uint8` 掩膜；范围内像素为 255，其余为 0。

```python
# 先将 BGR 转 HSV，减少亮度变化对颜色判断的影响
hsv = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)

# 绿色的示例范围，实际项目需要在现场光照下标定
lower_green = np.array([35, 80, 80], dtype=np.uint8)
upper_green = np.array([85, 255, 255], dtype=np.uint8)

# 生成黑白二值掩膜
mask = cv2.inRange(hsv, lower_green, upper_green)
```

红色在 OpenCV HSV 中跨越 0，需要合并两段：

```python
mask_low = cv2.inRange(hsv, np.array([0, 100, 80]), np.array([10, 255, 255]))
mask_high = cv2.inRange(hsv, np.array([170, 100, 80]), np.array([179, 255, 255]))
red_mask = cv2.bitwise_or(mask_low, mask_high)
```

### 27.18 `cv2.bitwise_and()` 等位运算

调用格式：

```python
dst = cv2.bitwise_and(src1, src2, mask=mask)
```

```python
# image 与自身按位与；真正控制保留区域的是 mask
selected = cv2.bitwise_and(image, image, mask=mask)

# 合并两个二值掩膜
combined = cv2.bitwise_or(mask1, mask2)

# 求两个掩膜共同存在的区域
intersection = cv2.bitwise_and(mask1, mask2)

# 黑白反转：0 变 255，255 变 0
inverted = cv2.bitwise_not(mask)

# 求只出现在其中一个掩膜的区域
difference = cv2.bitwise_xor(mask1, mask2)
```

作为 `mask=` 传入的数组必须为单通道 8 位图，并且尺寸与输入图一致。

### 27.19 `cv2.blur()`、`GaussianBlur()`：均值与高斯模糊

```python
# 5x5 均值滤波；每个像素替换为邻域平均值
mean_blur = cv2.blur(image, (5, 5))

# 5x5 高斯滤波；sigmaX=0 时由 OpenCV 根据核尺寸自动计算
gaussian = cv2.GaussianBlur(
    image,
    (5, 5),   # 核宽和核高通常必须为正奇数
    0,        # X 方向高斯标准差
)
```

参数选择：核越大，去噪越强，但细节损失越多。边缘检测前常从 `(5, 5)` 开始。

### 27.20 `cv2.medianBlur()`：中值滤波

```python
# ksize 只能是大于 1 的奇数，如 3、5、7
median = cv2.medianBlur(image, 5)
```

中值滤波把中心像素替换为邻域中值，对随机黑白点（椒盐噪声）特别有效。

### 27.21 `cv2.bilateralFilter()`：保边滤波

调用格式：

```python
dst = cv2.bilateralFilter(src, d, sigma_color, sigma_space)
```

```python
filtered = cv2.bilateralFilter(
    image,
    9,      # 邻域直径；越大越慢
    75,     # 颜色差异范围；越大越容易混合不同颜色
    75,     # 空间距离范围；越大影响范围越广
)
```

它能降噪并保留边缘，但在香橙派上计算量明显高于高斯滤波。

### 27.22 `cv2.filter2D()`：自定义卷积核

```python
# 锐化核：中心加强，周围相减
kernel = np.array([
    [0, -1, 0],
    [-1, 5, -1],
    [0, -1, 0],
], dtype=np.float32)

sharpened = cv2.filter2D(
    image,    # 输入图像
    -1,       # ddepth=-1：输出深度与输入相同
    kernel,   # 卷积核
)
```

### 27.23 `cv2.threshold()`：固定阈值与 Otsu 阈值

调用格式：

```python
used_threshold, binary = cv2.threshold(src, threshold, max_value, threshold_type)
```

参数：

| 参数 | 说明 |
|---|---|
| `src` | 通常为单通道灰度图 |
| `threshold` | 手动阈值；Otsu 模式中填 0 |
| `max_value` | 二值图白色像素值，通常为 255 |
| `threshold_type` | 阈值模式 |

```python
# 大于 127 的像素变成 255，其余变成 0
used, binary = cv2.threshold(
    gray,
    127,
    255,
    cv2.THRESH_BINARY,
)

# Otsu 根据灰度直方图自动计算阈值
otsu_value, otsu_binary = cv2.threshold(
    gray,
    0,
    255,
    cv2.THRESH_BINARY + cv2.THRESH_OTSU,
)

print(f"Otsu 自动选择的阈值：{otsu_value}")
```

### 27.24 `cv2.adaptiveThreshold()`：自适应阈值

调用格式：

```python
binary = cv2.adaptiveThreshold(
    src, max_value, adaptive_method, threshold_type, block_size, c
)
```

```python
binary = cv2.adaptiveThreshold(
    gray,                              # 8 位单通道灰度图
    255,                               # 白色像素值
    cv2.ADAPTIVE_THRESH_GAUSSIAN_C,    # 邻域加权高斯平均
    cv2.THRESH_BINARY,                 # 输出普通二值图
    11,                                # 邻域尺寸，必须是大于 1 的奇数
    2,                                 # 从邻域阈值中减去的常数
)
```

`block_size` 越大，阈值参考的范围越广；`c` 越大，越多像素容易变黑。

### 27.25 `cv2.getStructuringElement()`：创建形态学核

```python
# 矩形核：连接水平和竖直方向都较强
rect_kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (5, 5))

# 椭圆核：处理圆形色块时边缘更自然
ellipse_kernel = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (5, 5))

# 十字核：只强调上下左右方向
cross_kernel = cv2.getStructuringElement(cv2.MORPH_CROSS, (5, 5))
```

参数：第一个参数是形状，第二个参数是 `(核宽, 核高)`。

### 27.26 `cv2.erode()`、`cv2.dilate()`：腐蚀和膨胀

```python
kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (3, 3))

# 腐蚀让白色区域缩小，可去除孤立白点
eroded = cv2.erode(
    mask,
    kernel,
    iterations=1,   # 重复次数越多，效果越强
)

# 膨胀让白色区域扩大，可连接断裂区域
dilated = cv2.dilate(
    mask,
    kernel,
    iterations=1,
)
```

### 27.27 `cv2.morphologyEx()`：开、闭、梯度运算

```python
kernel = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (5, 5))

# 开运算：先腐蚀后膨胀，去除小白点
opened = cv2.morphologyEx(mask, cv2.MORPH_OPEN, kernel, iterations=1)

# 闭运算：先膨胀后腐蚀，填补白色目标里的小黑洞
closed = cv2.morphologyEx(opened, cv2.MORPH_CLOSE, kernel, iterations=1)

# 形态学梯度：膨胀图减腐蚀图，得到目标边界
gradient = cv2.morphologyEx(mask, cv2.MORPH_GRADIENT, kernel)
```

色块识别常用顺序：`inRange → OPEN → CLOSE → findContours`。

### 27.28 `cv2.Canny()`：Canny 边缘检测

调用格式：

```python
edges = cv2.Canny(image, threshold1, threshold2, apertureSize=3, L2gradient=False)
```

```python
# 先转灰度并模糊，减少噪声边缘
gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
blurred = cv2.GaussianBlur(gray, (5, 5), 0)

edges = cv2.Canny(
    blurred,
    50,          # 低阈值：弱边缘候选
    150,         # 高阈值：强边缘
    apertureSize=3,
    L2gradient=True,
)
```

调参经验：高阈值常取低阈值的 2～3 倍；噪声多时先加强模糊，不要只提高阈值。

### 27.29 `cv2.Sobel()`、`cv2.Laplacian()`：梯度边缘

```python
# CV_64F 保留正负梯度，避免直接截断
sobel_x_64 = cv2.Sobel(gray, cv2.CV_64F, 1, 0, ksize=3)
sobel_y_64 = cv2.Sobel(gray, cv2.CV_64F, 0, 1, ksize=3)

# 将绝对值缩放回 8 位图，便于显示
sobel_x = cv2.convertScaleAbs(sobel_x_64)
sobel_y = cv2.convertScaleAbs(sobel_y_64)

# 合成水平与垂直梯度
sobel = cv2.addWeighted(sobel_x, 0.5, sobel_y, 0.5, 0)

# Laplacian 同时响应多个方向的快速灰度变化
laplacian_64 = cv2.Laplacian(gray, cv2.CV_64F, ksize=3)
laplacian = cv2.convertScaleAbs(laplacian_64)
```

`Sobel` 的 `dx=1, dy=0` 检测沿 x 方向变化，对竖直边缘响应明显。

### 27.30 `cv2.findContours()`：查找轮廓

调用格式（OpenCV 4）：

```python
contours, hierarchy = cv2.findContours(binary, retrieval_mode, approximation_method)
```

参数：

| 参数 | 常用值 | 说明 |
|---|---|---|
| `binary` | 二值图 | 白色目标、黑色背景效果最直观 |
| `retrieval_mode` | `cv2.RETR_EXTERNAL` | 只找最外层轮廓 |
| `retrieval_mode` | `cv2.RETR_TREE` | 找所有轮廓并建立父子层级 |
| `approximation_method` | `cv2.CHAIN_APPROX_SIMPLE` | 压缩直线上的冗余点，通常使用它 |
| `approximation_method` | `cv2.CHAIN_APPROX_NONE` | 保留边界上的每一个点 |

```python
# findContours 可能修改输入数据，保守写法传入 copy()
contours, hierarchy = cv2.findContours(
    binary.copy(),
    cv2.RETR_EXTERNAL,
    cv2.CHAIN_APPROX_SIMPLE,
)

print(f"找到 {len(contours)} 个外轮廓")

# 按面积从大到小排序
contours = sorted(contours, key=cv2.contourArea, reverse=True)
```

### 27.31 `cv2.drawContours()`：绘制轮廓

```python
# contourIdx=-1 表示绘制列表中的全部轮廓
cv2.drawContours(
    image,
    contours,
    -1,
    (0, 255, 0),
    2,
)

# 只绘制第 0 个轮廓
if contours:
    cv2.drawContours(image, contours, 0, (0, 0, 255), 3)
```

### 27.32 `contourArea()`、`arcLength()`：面积和周长

```python
for contour in contours:
    # 轮廓内部的像素面积
    area = cv2.contourArea(contour)

    # closed=True 表示按闭合曲线计算周长
    perimeter = cv2.arcLength(contour, True)

    # 过滤面积太小的噪声轮廓
    if area < 500:
        continue

    print(f"面积={area:.1f}，周长={perimeter:.1f}")
```

`contourArea(contour, oriented=True)` 会根据轮廓点方向返回带正负号面积，一般项目保持默认 `False`。

### 27.33 `cv2.boundingRect()`：水平外接矩形

```python
# 返回左上角 x、y，以及矩形宽度 w、高度 h
x, y, width, height = cv2.boundingRect(contour)

# 右下角坐标为 x+width、y+height
cv2.rectangle(
    image,
    (x, y),
    (x + width, y + height),
    (0, 255, 0),
    2,
)

# 矩形中心点
center_x = x + width // 2
center_y = y + height // 2
```

### 27.34 `minAreaRect()`、`boxPoints()`：旋转外接矩形

```python
# rect 的结构为：((中心x, 中心y), (宽, 高), 旋转角度)
rect = cv2.minAreaRect(contour)
center, size, angle = rect

# 把旋转矩形转换成四个顶点坐标
box = cv2.boxPoints(rect)
box = np.int32(box)

# 以多边形方式画出旋转矩形
cv2.polylines(image, [box], True, (255, 0, 0), 2)

print(f"中心={center}，尺寸={size}，角度={angle}")
```

注意：`angle` 的范围和宽高定义会随矩形方向变化，做姿态控制前要统一角度规则。

### 27.35 `cv2.minEnclosingCircle()`：最小外接圆

```python
# center 为浮点圆心，radius 为浮点半径
(center_x, center_y), radius = cv2.minEnclosingCircle(contour)

# 绘图前转换为整数像素坐标
center = (int(center_x), int(center_y))
radius_int = int(radius)

cv2.circle(image, center, radius_int, (0, 255, 255), 2)
```

它计算的是“能够包住整个轮廓的最小圆”，并不代表目标本身一定是圆。

### 27.36 `cv2.approxPolyDP()`：轮廓多边形拟合

```python
# 先计算闭合轮廓周长
perimeter = cv2.arcLength(contour, True)

# epsilon 决定拟合精度；数值越大，保留的顶点越少
epsilon = 0.02 * perimeter
polygon = cv2.approxPolyDP(contour, epsilon, True)

# 顶点数量可用于粗略判断形状
corner_count = len(polygon)

if corner_count == 3:
    shape_name = "triangle"
elif corner_count == 4:
    shape_name = "quadrilateral"
else:
    shape_name = "circle_or_other"

cv2.drawContours(image, [polygon], -1, (0, 255, 0), 2)
```

只看顶点数容易受噪声影响，应先滤波、形态学处理，并结合长宽比和圆度判断。

### 27.37 `cv2.moments()`：计算轮廓中心

```python
# 计算轮廓的空间矩
moments = cv2.moments(contour)

# m00 对轮廓来说通常等于面积；为 0 时不能做除法
if moments["m00"] != 0:
    center_x = int(moments["m10"] / moments["m00"])
    center_y = int(moments["m01"] / moments["m00"])

    cv2.circle(image, (center_x, center_y), 5, (0, 0, 255), -1)
```

### 27.38 圆度、长宽比等自定义特征

```python
area = cv2.contourArea(contour)
perimeter = cv2.arcLength(contour, True)
x, y, width, height = cv2.boundingRect(contour)

# 防止除以 0
aspect_ratio = width / height if height > 0 else 0

# 圆度接近 1 时更像圆；形状不规则时会更小
circularity = 0.0
if perimeter > 0:
    circularity = 4 * np.pi * area / (perimeter * perimeter)

print(f"长宽比={aspect_ratio:.2f}，圆度={circularity:.2f}")
```

### 27.39 `cv2.HoughLinesP()`：概率霍夫直线

调用格式：

```python
lines = cv2.HoughLinesP(image, rho, theta, threshold, minLineLength, maxLineGap)
```

```python
# HoughLinesP 要求输入为单通道二值边缘图
lines = cv2.HoughLinesP(
    edges,
    1,                # rho：距离分辨率，单位像素
    np.pi / 180,      # theta：角度分辨率，1 度
    80,               # 累加器阈值，越大筛选越严格
    minLineLength=50, # 线段最短长度
    maxLineGap=10,    # 同一直线上允许连接的最大间隙
)

if lines is not None:
    for item in lines:
        # 每个元素通常形如 [[x1, y1, x2, y2]]
        x1, y1, x2, y2 = item[0]
        cv2.line(image, (x1, y1), (x2, y2), (0, 255, 0), 2)
```

### 27.40 `cv2.HoughCircles()`：霍夫圆检测

```python
# 通常先对灰度图做中值滤波，减少噪声
blurred = cv2.medianBlur(gray, 5)

circles = cv2.HoughCircles(
    blurred,
    cv2.HOUGH_GRADIENT,
    dp=1.2,         # 累加器分辨率与原图的反比
    minDist=50,     # 两个圆心之间的最小距离
    param1=100,     # 内部 Canny 的高阈值
    param2=30,      # 圆心累加器阈值；越小检测越多，也越容易误检
    minRadius=10,   # 最小半径
    maxRadius=100,  # 最大半径
)

if circles is not None:
    # 四舍五入并转成整数，方便绘图
    circles = np.uint16(np.around(circles))

    for center_x, center_y, radius in circles[0]:
        cv2.circle(image, (center_x, center_y), radius, (0, 255, 0), 2)
        cv2.circle(image, (center_x, center_y), 3, (0, 0, 255), -1)
```

### 27.41 `cv2.addWeighted()`：图像加权融合

调用格式：

```python
dst = cv2.addWeighted(src1, alpha, src2, beta, gamma)
```

计算公式：`dst = src1 × alpha + src2 × beta + gamma`。

```python
# 两张图必须具有相同尺寸和数据类型
blended = cv2.addWeighted(
    image1,
    0.7,     # 第一张图权重
    image2,
    0.3,     # 第二张图权重
    0,       # 整体增加的亮度偏移
)
```

它常用于透明叠加检测结果、掩膜热图或两帧画面。

### 27.42 `cv2.absdiff()`：绝对差分

```python
# 对应像素相减后取绝对值，输出不会出现负数
difference = cv2.absdiff(previous_frame, current_frame)

# 转灰度并二值化，得到发生明显变化的区域
gray_difference = cv2.cvtColor(difference, cv2.COLOR_BGR2GRAY)
_, motion_mask = cv2.threshold(
    gray_difference,
    25,
    255,
    cv2.THRESH_BINARY,
)
```

两张输入图必须尺寸和类型一致。

### 27.43 `cv2.normalize()`：数值归一化

```python
# 把输入数据线性映射到 0～255
normalized = cv2.normalize(
    source,
    None,
    alpha=0,
    beta=255,
    norm_type=cv2.NORM_MINMAX,
)

# 显示前转为 uint8
normalized_u8 = np.uint8(normalized)
```

它适合把深度图、梯度图或浮点计算结果映射到可视范围。

### 27.44 `cv2.calcHist()`：计算直方图

调用格式：

```python
hist = cv2.calcHist(images, channels, mask, hist_size, ranges)
```

```python
# images 必须传列表；[gray] 表示一张输入图
hist = cv2.calcHist(
    [gray],       # 输入图像列表
    [0],          # 统计第 0 通道
    None,         # 不使用掩膜，统计整张图
    [256],        # 分成 256 个灰度柱
    [0, 256],     # 像素范围，右端点不包含
)

print(hist.shape)  # (256, 1)
```

只统计 ROI 或目标区域时，将单通道掩膜传给 `mask`。

### 27.45 `equalizeHist()`、`createCLAHE()`：对比度增强

```python
# 全局直方图均衡化，只接受 8 位单通道图
global_equalized = cv2.equalizeHist(gray)

# CLAHE 把图像分块处理，可限制噪声被过度放大
clahe = cv2.createCLAHE(
    clipLimit=2.0,        # 对比度限制，越大增强越强
    tileGridSize=(8, 8),  # 分块网格数量
)

local_equalized = clahe.apply(gray)
```

### 27.46 `cv2.matchTemplate()`、`minMaxLoc()`：模板匹配

```python
# 模板尺寸必须小于或等于待搜索图像
score_map = cv2.matchTemplate(
    image,
    template,
    cv2.TM_CCOEFF_NORMED,
)

# 在得分图中找到最小值、最大值及对应坐标
min_value, max_value, min_location, max_location = cv2.minMaxLoc(score_map)

# TM_CCOEFF_NORMED 模式下，最大值位置是最佳匹配位置
if max_value >= 0.8:
    template_height, template_width = template.shape[:2]
    top_left = max_location
    bottom_right = (
        top_left[0] + template_width,
        top_left[1] + template_height,
    )
    cv2.rectangle(image, top_left, bottom_right, (0, 255, 0), 2)
```

若使用 `cv2.TM_SQDIFF` 或 `TM_SQDIFF_NORMED`，最佳位置应取最小值位置 `min_location`。

### 27.47 `connectedComponentsWithStats()`：连通区域分析

```python
component_count, labels, stats, centroids = cv2.connectedComponentsWithStats(
    binary,
    connectivity=8,         # 8 邻域；也可以使用 4 邻域
    ltype=cv2.CV_32S,       # 标签数组的数据类型
)

# 标签 0 是背景，所以从 1 开始遍历目标
for label in range(1, component_count):
    x = stats[label, cv2.CC_STAT_LEFT]
    y = stats[label, cv2.CC_STAT_TOP]
    width = stats[label, cv2.CC_STAT_WIDTH]
    height = stats[label, cv2.CC_STAT_HEIGHT]
    area = stats[label, cv2.CC_STAT_AREA]

    center_x, center_y = centroids[label]

    if area < 500:
        continue

    cv2.rectangle(image, (x, y), (x + width, y + height), (0, 255, 0), 2)
```

返回值：

- `component_count`：包含背景在内的标签数量。
- `labels`：每个像素所属标签。
- `stats`：每个区域的左、上、宽、高、面积。
- `centroids`：每个区域的浮点中心坐标。

### 27.48 `cv2.goodFeaturesToTrack()`：Shi-Tomasi 角点

```python
corners = cv2.goodFeaturesToTrack(
    gray,
    maxCorners=100,      # 最多返回多少个角点
    qualityLevel=0.01,   # 最低质量比例，越大筛选越严格
    minDistance=10,      # 角点之间的最小距离
    blockSize=3,         # 计算局部统计的窗口大小
)

if corners is not None:
    # 返回形状通常为 Nx1x2，先转为整数坐标
    corners = np.int32(corners)

    for corner in corners:
        x, y = corner.ravel()
        cv2.circle(image, (x, y), 4, (0, 0, 255), -1)
```

### 27.49 `cv2.ORB_create()`：ORB 特征点和描述子

```python
# nfeatures 控制最多保留的特征点数量
orb = cv2.ORB_create(nfeatures=500)

# mask=None 表示在整张灰度图中检测
keypoints, descriptors = orb.detectAndCompute(gray, None)

# keypoints 是关键点对象列表；descriptors 通常为 uint8 二进制描述子
print(f"关键点数量：{len(keypoints)}")

# 画出关键点的位置和方向
visualized = cv2.drawKeypoints(
    image,
    keypoints,
    None,
    color=(0, 255, 0),
    flags=cv2.DRAW_MATCHES_FLAGS_DRAW_RICH_KEYPOINTS,
)
```

### 27.50 `cv2.BFMatcher()`：暴力特征匹配

```python
# ORB 是二进制描述子，所以使用 NORM_HAMMING
matcher = cv2.BFMatcher(
    cv2.NORM_HAMMING,
    crossCheck=True,
)

# 将图 1 的描述子与图 2 的描述子匹配
matches = matcher.match(descriptors1, descriptors2)

# distance 越小表示描述子越相似
matches = sorted(matches, key=lambda match: match.distance)

# 只绘制最好的前 30 个匹配
matched_image = cv2.drawMatches(
    image1,
    keypoints1,
    image2,
    keypoints2,
    matches[:30],
    None,
    flags=cv2.DrawMatchesFlags_NOT_DRAW_SINGLE_POINTS,
)
```

使用 `knnMatch(k=2)` 时通常配合 Lowe 比率测试：

```python
matcher = cv2.BFMatcher(cv2.NORM_HAMMING)
pair_matches = matcher.knnMatch(descriptors1, descriptors2, k=2)

good_matches = []
for best, second_best in pair_matches:
    if best.distance < 0.75 * second_best.distance:
        good_matches.append(best)
```

### 27.51 `cv2.minMaxLoc()`：寻找最小值和最大值

```python
min_value, max_value, min_location, max_location = cv2.minMaxLoc(gray)

print(f"最暗像素值={min_value}，位置={min_location}")
print(f"最亮像素值={max_value}，位置={max_location}")
```

如果输入多通道图，应先提取单通道或转灰度。

### 27.52 `cv2.countNonZero()`：统计白色像素

```python
# 统计单通道数组中所有不为 0 的像素数量
white_pixel_count = cv2.countNonZero(mask)

# 用白色像素数除以总像素数，得到目标覆盖比例
coverage = white_pixel_count / mask.size
print(f"目标覆盖率：{coverage:.2%}")
```

### 27.53 `cv2.copyMakeBorder()`：给图像加边框

```python
padded = cv2.copyMakeBorder(
    image,
    20,                    # 上边框像素数
    20,                    # 下边框像素数
    40,                    # 左边框像素数
    40,                    # 右边框像素数
    cv2.BORDER_CONSTANT,   # 使用固定颜色填充
    value=(0, 0, 0),       # BGR 黑色
)
```

常见边界模式还有 `BORDER_REPLICATE`（复制边缘）和 `BORDER_REFLECT`（镜像边缘）。

### 27.54 `cv2.convertScaleAbs()`：缩放并转 8 位绝对值

调用公式近似为：`dst = saturate_uint8(abs(src × alpha + beta))`。

```python
# 把浮点 Sobel 梯度取绝对值并转成可显示的 uint8
display_gradient = cv2.convertScaleAbs(
    sobel_64,
    alpha=1.0,
    beta=0,
)

# 简单调整图片对比度和亮度
adjusted = cv2.convertScaleAbs(
    image,
    alpha=1.2,  # 对比度倍率
    beta=20,    # 亮度偏移
)
```

### 27.55 一段带完整注释的“摄像头色块识别 + 浏览器输出”模板

```python
import cv2
import numpy as np

# 这是你项目中的网页双路显示工具，不属于 OpenCV 官方库
from xbhdcc_tools import WebStreamer


def main() -> None:
    """采集 USB 摄像头，识别绿色色块，并把原图和掩膜发到网页。"""

    # 0 对应 /dev/video0；Linux 明确选择 V4L2 后端
    cap = cv2.VideoCapture(0, cv2.CAP_V4L2)

    # 摄像头不存在、被占用或权限不足时会打开失败
    if not cap.isOpened():
        raise RuntimeError("无法打开摄像头 /dev/video0")

    # 使用摄像头支持的 MJPG 格式，可降低 USB 带宽压力
    fourcc = cv2.VideoWriter_fourcc("M", "J", "P", "G")
    cap.set(cv2.CAP_PROP_FOURCC, fourcc)

    # 请求标准 720p 和 30 FPS
    cap.set(cv2.CAP_PROP_FRAME_WIDTH, 1280)
    cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 720)
    cap.set(cv2.CAP_PROP_FPS, 30)

    # 打印驱动实际采用的参数，不能只相信 set() 的请求值
    actual_width = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
    actual_height = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))
    actual_fps = cap.get(cv2.CAP_PROP_FPS)
    print(f"Camera: {actual_width}x{actual_height} @ {actual_fps:.1f} FPS")

    # 创建监听 8081 端口的网页视频流服务器
    streamer = WebStreamer(port=8081)

    # HSV 绿色上下限；需要根据现场灯光重新标定
    lower_green = np.array([35, 80, 80], dtype=np.uint8)
    upper_green = np.array([85, 255, 255], dtype=np.uint8)

    # 5x5 椭圆核用于清理掩膜噪声
    kernel = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (5, 5))

    try:
        while True:
            # 从摄像头读取一帧 BGR 图像
            ret, frame = cap.read()

            # 读取失败时退出，避免 continue 形成高速空循环占满 CPU
            if not ret:
                print("读取摄像头画面失败")
                break

            # BGR 转 HSV，便于按颜色范围分割
            hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

            # 范围内像素设为 255，其余设为 0
            mask = cv2.inRange(hsv, lower_green, upper_green)

            # 开运算去除小白噪点
            mask = cv2.morphologyEx(mask, cv2.MORPH_OPEN, kernel)

            # 闭运算填补目标内部的小黑洞
            mask = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, kernel)

            # 只寻找最外层轮廓，并压缩轮廓点
            contours, _ = cv2.findContours(
                mask,
                cv2.RETR_EXTERNAL,
                cv2.CHAIN_APPROX_SIMPLE,
            )

            # 有轮廓时，选择面积最大的轮廓作为当前目标
            if contours:
                target = max(contours, key=cv2.contourArea)
                area = cv2.contourArea(target)

                # 面积过滤可以排除残留的小噪声
                if area >= 500:
                    # 获取水平外接矩形
                    x, y, width, height = cv2.boundingRect(target)

                    # 根据矩形计算目标中心
                    center_x = x + width // 2
                    center_y = y + height // 2

                    # 在原图上绘制目标矩形
                    cv2.rectangle(
                        frame,
                        (x, y),
                        (x + width, y + height),
                        (0, 255, 0),
                        2,
                    )

                    # 用红点标记目标中心
                    cv2.circle(
                        frame,
                        (center_x, center_y),
                        5,
                        (0, 0, 255),
                        -1,
                    )

                    # 把中心坐标和面积写到画面上
                    cv2.putText(
                        frame,
                        f"center=({center_x}, {center_y}) area={area:.0f}",
                        (x, max(30, y - 10)),
                        cv2.FONT_HERSHEY_SIMPLEX,
                        0.7,
                        (0, 255, 0),
                        2,
                        cv2.LINE_AA,
                    )

            # 通道 0 显示绘制检测结果后的彩色原图
            streamer.update_frame(0, frame)

            # 网页工具期望三通道图，所以把单通道 mask 转成 BGR
            mask_bgr = cv2.cvtColor(mask, cv2.COLOR_GRAY2BGR)

            # 通道 1 显示二值掩膜，方便观察颜色分割是否准确
            streamer.update_frame(1, mask_bgr)
    finally:
        # 无论正常退出还是异常退出，都释放摄像头和网页服务
        cap.release()
        streamer.stop()


if __name__ == "__main__":
    main()
```

这段模板调试时按以下顺序观察：

1. 原图是否正常、曝光是否合适。
2. `mask` 是否只保留目标颜色。
3. 开闭运算是否去除了噪声且没有吃掉目标。
4. 最大轮廓是否真的是目标。
5. 中心坐标是否稳定，再考虑串口或控制算法。
