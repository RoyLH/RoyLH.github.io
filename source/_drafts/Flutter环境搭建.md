---
title: Flutter环境搭建
categories:
  - APP开发
tags:
  - Flutter
  - 跨平台APP开发
abbrlink: 50431
---

![](http://img2.mukewang.com/5b4c075b000198c216000586.jpg)

## 你需要准备的环境
---
OS：macOS (64-bit)
磁盘空间：700 MB 
工具包：Flutter依赖以下工具包：
- bash
- curl
- git 2.x
- mkdir
- rm
- unzip
- which
---

## （一）安装Flutter SDK
---
1. Flutter SDK 获取
    官网下载最新稳定版本Flutter SDK的zip包。
    如： flutter_macos_v1.5.4-hotfix.2-stable.zip


2. 解压缩
    下载好的Flutter SDK 默认会存放在~/Downloads文件夹。
    如： ~/Downloads/flutter_macos_v1.5.4-hotfix.2-stable.zip
    
    命令行执行：
    ```
    cd ~/development
    unzip ~/Downloads/flutter_macos_v1.5.4-hotfix.2-stable.zip

    ```
    注意：这个development文件夹自定义命名（我命名的是flutter文件夹）


3. 添加flutter到环境变量
    ```
    export PATH="$PATH:`pwd`/flutter/bin"
    ```
    这行命令将环境变量添加到$HOME/.bash_profile文件中
    ```
    source $HOME/.bash_profile
    echo $PATH
    ```
    这样你就可以看到你所添加的环境变量了 说明添加成功
---

## （二）IOS平台

---
1. 安装Xcode
   - 去Apple store 下载安装
   - 命令行运行以下命令， 配置Xcode命令行工具以使用新安装的Xcode版本:
   ```
   sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
   ```
   - 确保Xcode许可协议，通过打开一次Xcode或通以下命令同意过了.
    ```
    sudo xcodebuild -license
    ```
2. 设置IOS模拟器
   - 在Mac上，通过Spotlight或使用以下命令找到模拟器:
   ```
   open -a Simulator
   ```
   - 通过检查模拟器 硬件>设备 菜单中的设置，确保您的模拟器正在使用64位设备（iPhone 5s或更高版本）
   - 根据您的开发机器的屏幕大小，模拟的高清屏iOS设备可能会使您的屏幕溢出。在模拟器的 Window> Scale 菜单下设置设备比例
   - 运行 flutter run启动您的应用.

3. 安装到iOS设备（真机调试）
   前提：您需要一些额外的工具和一个Apple帐户，您还需要在Xcode中进行设置。
   - 安装 homebrew 
   - 打开终端并运行这些命令来安装用于将Flutter应用安装到iOS设备的工具
  ```
  brew update
  brew install --HEAD libimobiledevice
  brew install ideviceinstaller ios-deploy cocoapods
  pod setup
  ```
  - 遵循Xcode签名流程来配置您的项目:
    - 在你Flutter项目目录中通过 open ios/Runner.xcworkspace 打开默认的Xcode workspace
    - 在Xcode中，选择导航面板左侧中的Runner项目
    - 在Runner target设置页面中，确保在 常规>签名>团队 下选择了您的开发团队。当您选择一个团队时，Xcode会创建并下载开发证书，向您的设备注册您的帐户，并创建和下载配置文件（如果需要）
      - 要开始您的第一个iOS开发项目，您可能需要使用您的Apple ID登录Xcode. 任何Apple ID都支持开发和测试。需要注册Apple开发者计划才能将您的应用分发到App Store.
      - 当您第一次attach真机设备进行iOS开发时，您需要同时信任你的Mac和该设备上的开发证书。首次将iOS设备连接到Mac时,请在对话框中选择 Trust。然后，转到iOS设备上的设置应用程序，选择 常规>设备管理 并信任您的证书。
      - 如果Xcode中的自动签名失败，请验证项目的 General > Identity > Bundle Identifier 值是否唯一.
  - 运行启动您的应用程序 flutter run

---