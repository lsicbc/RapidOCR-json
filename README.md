#### 离线OCR组件 系列项目：
- [PaddleOCR-json](https://github.com/hiroi-sora/PaddleOCR-json)
- **RapidOCR-json**

|                  | PaddleOCR-json                                  | RapidOCR-json        |
| ---------------- | ----------------------------------------------- | -------------------- |
| CPU要求          | CPU必须具有AVX指令集。不支持以下CPU：           | 无特殊要求 👍         |
|                  | 凌动Atom，安腾Itanium，赛扬Celeron，奔腾Pentium |                      |
| 推理加速库       | mkldnn 👍                                        | 无                   |
| 识别速度         | 快（启用mkldnn加速）👍                           | 中等                 |
|                  | 极慢（不启用mkldnn）                            |                      |
| 初始化耗时       | 约2s，慢                                        | 0.1s内，快 👍         |
| 组件体积（压缩） | 52MB                                            | ~18MB 👍              |
| 组件体积（部署） | 250MB                                           | ~22MB 👍              |
| CPU占用          | 高，榨干硬件性能                                | 较低，对低配机器友好 |
| 内存占用峰值     | >2000MB（启用mkldnn）                           | ~500MB 👍             |
|                  | ~600MB（不启用mkldnn）                          |                      |

---

# RapidOCR-json v0.9.0

这是一个基于 [RapidOcrOnnx](https://github.com/RapidAI/RapidOcrOnnx) 的离线图片OCR文字识别程序。通过管道等方式输入本地图片路径，输出识别结果json字符串。适用于 `Win7 x64` 及以上的系统。

本项目旨在提供一个封装好的OCR引擎组件，使得没有C++编程基础的用户也可以用别的语言来简单地调用OCR，享受到更快的运行效率、更便捷的打包&部署手段。

## 模型升级说明 (v0.9.0)

本版本已升级推理引擎和模型：

| 模块     | 模型                                      | 大小    | 说明         |
| -------- | ----------------------------------------- | ------- | ------------ |
| 检测     | `ch_PP-OCRv5_det_mobile.onnx`            | 4.6 MB  | PP-OCRv5 检测 |
| 方向分类 | `ch_ppocr_mobile_v2.0_cls_infer.onnx`    | 0.56 MB | PP-OCRv2 分类 |
| 识别     | `ch_PP-OCRv5_rec_mobile.onnx`            | 15.9 MB | PP-OCRv5 识别 |
| 字典     | `ppocrv5_dict.txt`                        | 0.07 MB | 18383 字符集 |
| 推理引擎 | `onnxruntime.dll` (v1.26.0)               | 14.2 MB | 2025年5月版  |

启动时会显示当前加载的模型信息：

```
RapidOCR-json v0.9.0
Models loaded:
  det: models/ch_PP-OCRv5_det_mobile.onnx
  cls: models/ch_ppocr_mobile_v2.0_cls_infer.onnx
  rec: models/ch_PP-OCRv5_rec_mobile.onnx
  keys: models/ppocrv5_dict.txt
OCR init completed.
```

## 准备工作

下载发布包并解压，即可使用。无需安装任何依赖。

### 简单试用

方式一：命令行单次识别

打开控制台，输入：
```
RapidOCR-json.exe --image_path=D:/test.png
```

方式二：管道交互模式

直接双击打开 `RapidOCR-json.exe`（或命令行不加 `--image_path`）。等初始化完毕输出 `OCR init completed.` 后，通过 stdin 输入 JSON 指令：

```
{"image_path": "D:/test.png"}
```

也支持 base64 图片：
```
{"image_base64": "iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNk+M9QDwADhgGAWjR9awAAAABJRU5ErkJggg=="}
```

### Python API

```python
from RapidOCR_api import OcrAPI

ocr = OcrAPI('RapidOCR-json.exe')
res = ocr.run('test.png')
print(res)
ocr.stop()
```

## 指令说明

### 命令行参数

| 参数           | 说明                                 | 默认值                                |
| -------------- | ------------------------------------ | ------------------------------------- |
| `--image_path` | 初始图片路径（单次识别后退出）       | （空，进入交互模式）                  |
| `--models`     | 模型目录地址                         | `models`                              |
| `--det`        | 检测模型文件名                       | `ch_PP-OCRv5_det_mobile.onnx`        |
| `--cls`        | 方向分类模型文件名                   | `ch_ppocr_mobile_v2.0_cls_infer.onnx` |
| `--rec`        | 识别模型文件名                       | `ch_PP-OCRv5_rec_mobile.onnx`        |
| `--keys`       | 字典文件名                           | `ppocrv5_dict.txt`                    |
| `--numThread`  | 线程数                               | 4                                     |
| `--padding`    | 预处理白边宽度                       | 50                                    |
| `--maxSideLen` | 图片长边缩放值（提高大图速度）       | 1024                                  |
| `--boxScoreThresh` | 文字框置信度门限                  | 0.5                                   |
| `--boxThresh`  | 文字框阈值                           | 0.3                                   |
| `--unClipRatio`| 文字框扩展倍率                       | 1.6                                   |
| `--doAngle`    | 启用(1)/禁用(0) 文字方向检测         | 1                                     |
| `--mostAngle`  | 启用(1)/禁用(0) 角度投票             | 1                                     |
| `--GPU`        | GPU编号（-1禁用，0/1/…指定GPU）      | -1                                    |
| `--ensureAscii`| 启用(1)/禁用(0) ASCII转义输出        | 0                                     |
| `--ensureLogger`| 启用(1)/禁用(0) 日志和可视化输出    | 0                                     |
| `--version`    | 显示版本                             | -                                     |
| `--help`       | 显示帮助                             | -                                     |

### 使用示例

```bash
# 单次识别
RapidOCR-json.exe --image_path="D:/images/test.png"

# 指定模型
RapidOCR-json.exe --det=custom_det.onnx --rec=custom_rec.onnx --image_path=test.png

# 调整参数提高精度
RapidOCR-json.exe --padding=100 --boxScoreThresh=0.6 --image_path=test.png

# 交互模式（从 stdin 持续接收 JSON 指令）
RapidOCR-json.exe
```

## 返回值说明

每次OCR返回一个JSON对象，包含 `code` 和 `data` 两个字段。

### 状态码

| code | 含义 | data 格式 |
| ---- | ---- | --------- |
| `100` | 成功，识别到文字 | 数组，每项含 `text` `box` `score` |
| `101` | 成功，未识别到文字 | 字符串说明 |
| `200` | 图片路径不存在 | 字符串说明 |
| `201` | 路径编码转换失败 | 字符串说明 |
| `202` | 图片打开失败 | 字符串说明 |
| `203` | 图片解码失败 | 字符串说明 |
| `204` | JSON解析失败 | 字符串说明 |
| `210-217` | 剪贴板相关错误 | 字符串说明 |
| `299` | 未知错误 | 字符串说明 |

### 成功返回值示例 (code=100)

```json
{
  "code": 100,
  "data": [
    {
      "box": [[13, 5], [161, 5], [161, 27], [13, 27]],
      "score": 0.9996442794799805,
      "text": "飞舞的因果交流"
    }
  ]
}
```

- `text` : 识别文本
- `box` : 四点坐标 [左上, 右上, 右下, 左下]
- `score` : 平均字符置信度 (0~1)

## 文件结构

```
RapidOCR-json/
├── RapidOCR-json.exe           # 主程序 (~6.5 MB)
├── onnxruntime.dll             # ONNX 推理引擎 (~14 MB)
├── onnxruntime_providers_shared.dll
├── models/
│   ├── ch_PP-OCRv5_det_mobile.onnx       # 检测模型
│   ├── ch_ppocr_mobile_v2.0_cls_infer.onnx # 分类模型
│   ├── ch_PP-OCRv5_rec_mobile.onnx       # 识别模型
│   └── ppocrv5_dict.txt                  # 字符字典
├── api/
│   └── python/
│       ├── RapidOCR_api.py               # Python 封装库
│       └── demo1.py                      # 使用示例
└── README.md
```

## 编译构建

支持 Visual Studio 2026 (v18) 编译。详见 [cpp/README.md](cpp/README.md)。

构建要求：
- Visual Studio 2026 Community
- CMake >= 3.12
- C++17

```
cd cpp
cmake -G "Visual Studio 18 2026" -A x64 -DOCR_OUTPUT="BIN" -DOCR_BUILD_CRT="True" -DOCR_ONNX="CPU" -B build
cmake --build build --config Release --target RapidOcrOnnx
```

## 感谢

- [RapidAI/RapidOcrOnnx](https://github.com/RapidAI/RapidOcrOnnx) — 核心OCR引擎
- [nlohmann/json](https://github.com/nlohmann/json) — C++ JSON库
- [ONNX Runtime](https://github.com/microsoft/onnxruntime) — 推理引擎
- [PaddleOCR](https://github.com/PaddlePaddle/PaddleOCR) — 模型训练

## 更新日志

#### v0.9.0 `2026.5.29`
- 升级 ONNX Runtime 推理引擎至 v1.26.0
- 全线升级至 PP-OCRv5 模型 (检测 + 识别，字符集 18383 字)
- 自动显示当前加载的模型信息
- 适配 Visual Studio 2026 (v18) 编译
- 优化发布包体积至 ~22MB

#### v0.2.0 `2023.9.25`
- 路径识图的key由 `imagePath` 改为 `image_path`
- 新功能：base64识图，key为 `image_base64`

#### v0.1.0 `2023.4.29`
- 初始发布
