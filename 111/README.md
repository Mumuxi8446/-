# 装甲板灯条识别

基于 OpenCV 的装甲板灯条识别程序，用于识别机器人装甲板上的灯条并匹配成装甲板。
## 开发环境

编译器：Visual Studio 版本（VS2026）、平台 x64 Debug
依赖库：OpenCV（4.13.0）
系统：Windows

## 功能特点

- 识别图像中的灯条（高亮长条形区域）
- 自动匹配左右灯条形成装甲板
- 绘制灯条轮廓和装甲板中心点
- 支持倾斜角度识别

## 环境要求

- OpenCV 4.x
- C++11 或更高版本

## 编译运行

用 VS 打开 111.sln
配置项目属性：配置 OpenCV 包含目录、库目录、链接器依赖
编译：选择 x64-Debug，生成解决方案
运行：程序运行结果保存在 armor_result 文件夹
x64 文件夹为编译缓存，可定期删除清理

### 编译
```bash
g++ -o armor_detector 112.cpp `pkg-config --cflags --libs opencv4` -std=c++11
