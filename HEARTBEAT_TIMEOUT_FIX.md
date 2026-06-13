# Heartbeat Timeout 问题诊断与解决

## 问题描述
运行 ESP32-S3 固件时出现 "Heartbeat timeout" 错误。

## 已应用的修复

### 1. platformio.ini 配置更新
已添加以下关键配置：
- `monitor_speed = 115200` - 设置串口监视器波特率
- `monitor_dtr = on` - 启用 DTR 信号（用于设备复位）
- `monitor_rts = on` - 启用 RTS 信号（用于设备复位）
- `upload_speed = 921600` - 提高上传速度
- `upload_flags` - 指定复位策略

## 故障排查步骤

### 步骤 1: 检查 USB 连接
```bash
# Windows: 打开设备管理器
# 查找 "USB-to-UART" 或类似的设备
# 确保没有黄色感叹号，端口通常是 COM3 或 COM4
```

### 步骤 2: 验证串口连接
```bash
# PlatformIO 中运行 monitor 命令
# 在 VSCode 中打开终端，运行：
pio device monitor --port COM3 --baud 115200
# 按 ESP32-S3 上的 RESET 按钮
# 应该看到启动日志
```

### 步骤 3: 清理构建并重新编译
```bash
pio run -t clean
pio run -t upload --verbose
```

### 步骤 4: 检查 setup() 函数性能
主程序的 setup() 中包含：
- EEPROM 初始化 (1000ms 延迟)
- Serial/WLAN 初始化
- Camera 初始化
- 额外 1000ms 延迟

**改进方案**：如果 setup() 执行时间过长，可以：
1. 减少 delay() 的时间
2. 将初始化分散到 loop() 中
3. 使用异步初始化

### 步骤 5: 强制 DFU 模式重新上传
如果上述方法都不行，可尝试强制 DFU 模式：
```bash
# 1. 按住 BOOT 按钮
# 2. 按一下 RESET 按钮
# 3. 释放 BOOT 按钮
# 4. 再次运行上传命令
pio run -t upload --verbose
```

## 常见原因列表

| 症状 | 原因 | 解决方案 |
|------|------|--------|
| 完全无法检测到设备 | USB 驱动或连接问题 | 更换 USB 数据线或 USB 口 |
| 上传成功但监视器超时 | 波特率不匹配 | 确认 platformio.ini 中的 monitor_speed |
| 间歇性超时 | USB 供电不足 | 使用 USB3.0 口或外接电源 |
| 固件加载慢导致超时 | 大型固件或复杂初始化 | 优化 setup() 或增加超时时间 |

## 增加超时时间（如需要）
编辑 `platformio.ini`，在 `[env:esp32-s3-devkitc-1]` 下添加：
```ini
monitor_wait_for_upload_port = 30  ; 等待端口出现 30 秒
```

## 参考资源
- [PlatformIO 串口监视器文档](https://docs.platformio.org/en/latest/core/userguide/device/monitor.html)
- [ESP32-S3 官方文档](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/)
