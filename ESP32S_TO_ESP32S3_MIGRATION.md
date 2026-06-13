# ESP32S to ESP32-S3 Migration Guide

## 完成的适配与优化

本文档记录了从旧的 ESP32S 项目到 ESP32-S3 项目的迁移和优化工作。

### ✅ 已应用的改进

#### 1. 摄像头参数优化 (esp32cam.cpp)
**源自旧项目的更好参数设置：**
- `set_brightness(+2)` - 提升亮度 2 档
- `set_special_effect(2)` - 灰度效果处理  
- `set_aec2(1)` - 启用 AEC2（自动曝光二代）
- `set_aec_value(800)` - 曝光目标值从 400→800
- `set_agc_gain(10)` - AGC 增益从 0→10
- `set_bpc(1)` - 启用坏点补正
- `set_lenc(0)` - 禁用镜头失真补正（原为启用）
- `set_wb_mode(1)` - 自动白平衡模式改为 Sunny（原为 Auto）

**效果：** 更好的图像质量和曝光效果

#### 2. 设备标定函数 (esp32cam.cpp + .h)
**新增函数：**
- `sanitizeDeviceFlag()` - 验证设备标志有效性
- `updateDeviceFlag()` - 安全地更新设备位置配置

**优势：** 更健壮的配置管理

#### 3. 序列消息处理改进 (serialMsg.cpp)
- 添加 POSITION_CFG 消息处理支持
- 更完善的错误提示信息
- 为日志函数预留接口

#### 4. 无线消息协议扩展 (wlanMsg.cpp)
- 添加对多种心跳包类型的支持
- 增加 DEVICEINFO 消息作为备用心跳源
- 更灵活的设备识别机制

#### 5. 启动日志增强 (main.cpp)
- 明确的启动分隔标记
- ESP32-S3 设备标识
- 更清晰的初始化流程

#### 6. Heartbeat Timeout 修复 (已在之前的会话完成)
- `bHeartbeatTimeout` 初值改为 `false`
- WiFi 连接正常时不阻止程序运行
- 只在真正失连时进行重连

#### 7. 消息长度校验修复 (已在之前的会话完成)
- 修正 TLV 消息长度计算
- 避免 4 字节的偏差

---

## 硬件差异对比

| 特性 | ESP32S | ESP32-S3 |
|------|--------|----------|
| 核心 | Dual Core | Dual Core |
| FLASH | 4MB | 16MB (已配置) |
| PSRAM | 可选 | 8MB 可选 (已启用) |
| 高速 I/O | GPIO | GPIO + USB |
| 摄像头性能 | 标准 | 提升 |
| 内存管理 | 受限 | 扩展 |

---

## 编译配置调整

### platformio.ini 关键参数

```ini
[env:esp32-s3-devkitc-1]
platform = espressif32
board = esp32-s3-devkitc-1
framework = arduino

# 分区表：16MB FLASH
board_build.arduino.partitions = default_16MB.csv

# 内存配置：QIO+OPI 模式（PSRAM）
board_build.arduino.memory_type = qio_opi

# 编译标志：启用 PSRAM
build_flags = -DBOARD_HAS_PSRAM

# FLASH 容量
board_upload.flash_size = 16MB

# 波特率
monitor_speed = 115200
```

---

## 测试建议

### 编译验证
```bash
cd eyeTracker/software
pio run -t clean
pio run -t build
```

### 上传测试
```bash
pio run -t upload
pio device monitor --baud 115200
```

### 功能测试清单
- [ ] 串口初始化和日志输出
- [ ] WiFi 连接和断线重连
- [ ] 摄像头初始化和图像采集
- [ ] 消息格式正确（无长度校验错误）
- [ ] 心跳超时处理正常
- [ ] 设备位置配置可更新

---

## 未来优化方向

### 可选升级项目
1. **V2 消息协议** - 采用旧项目的 MSG_WLAN_IMAGE_V2_S（更灵活的框架）
2. **传输模式切换** - USB/WiFi 双模式支持
3. **完整日志系统** - 结构化的 SERIAL_MSG_LOG_S 日志格式
4. **固件版本字符串** - 添加 VERSION_STR 宏

### 性能优化
1. 利用 PSRAM 进行多帧缓冲
2. 优化 UDP 包分片策略
3. 动态帧率调整

---

## 关键文件清单

已修改的文件：
- ✅ `src/main.cpp` - 初始化流程
- ✅ `src/esp32cam.cpp` - 摄像头参数和标定函数
- ✅ `src/esp32cam.h` - 函数声明
- ✅ `src/wlanMsg.cpp` - 心跳包处理
- ✅ `src/serialMsg.cpp` - 消息回调处理
- ✅ `platformio.ini` - 编译配置

---

## 变更日志

| 日期 | 改动 | 说明 |
|------|------|------|
| 2024 | Heartbeat timeout 修复 | 初版 |
| 2024 | ESP32S 代码移植 | 完成本次适配 |

---

## 参考资源

- [ESP32-S3 官方文档](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/)
- [PlatformIO ESP32 指南](https://docs.platformio.org/en/latest/boards/espressif32/index.html)
- [原项目 GitHub](https://github.com/Dominocs/Project_Cymple)
