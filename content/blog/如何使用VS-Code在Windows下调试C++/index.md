---
title: '如何使用VS Code在Windows下调试C++'
date: 2021-10-24 12:43:00 +0800
tags: 
- VSCode
- C++
img: null
draft: true
---

本文系原创，首发于[blog.smagic.top](https://blog.smagic.top/)，同步发布于cnblogs与知乎，作者SMagic保留一切著作权，转载请注明来源与出处。

## 前言

最近在用VS Code写C++，发现VS Code官方文档中没有自动用VS的C++编译器cl.exe编译并且在VS Code中调试程序的方案，在网上也找不到合适的。无奈之下只有自己琢磨，通过翻阅微软的文档，DIY出了一套方案，于是写篇博客记录一下。其实主要原因是懒得为了写点水代码，专门用VS开一个项目（逃

这篇文章最初写于2021-6-25，写完了以后没有发布，后来VS Code官方文档于2021-8-19更新了[VS Code在Windows下调试C++的相关教程](https://code.visualstudio.com/docs/cpp/config-msvc)，看了以后感觉官方的实现更优雅一点。但是我文章都写完了，如果就这么作废了，心里会很不舒服，所以还是打算发出来，给大家献个丑。

## 为什么使用 Visual Studio 开发人员命令行工具

关于调试C++程序，VS Code 官方给出的解决方案是使用MinGW，而MinGW有个问题，用它生成的exe文件如果需要在别的电脑上运行，就需要带着MinGW的一堆库一起走。相比之下，使用Visual Studio编译C++源代码得到的exe，运行所需要的“Microsoft Visual C++ Redistributable”在绝大多数Windows电脑上都是自带的。即使不使用 Visual Studio 进行开发工作，也可以只下载[Visual Studio 2019 生成工具](https://visualstudio.microsoft.com/zh-hans/downloads/#build-tools-for-visual-studio-2019)，或者只下载[Microsoft C++ 生成工具](https://visualstudio.microsoft.com/zh-hans/visual-cpp-build-tools/)，然后使用命令行工具。这些就是我推荐使用它的理由。

“Visual Studio 开发人员命令行工具”是Visual Studio自带的几个创建了一堆临时环境变量的脚本。运行这个脚本会打开一个命令行界面，可以在其中使用Visual Studio的各种语言的编译工具，比如解决方案(*.sln)生成工具`msbuild`，C++编译器`cl.exe`，C#编译器`csc.exe`，Visual Basic编译器`vbc.exe`，而无需为其配置任何环境变量。

常规的做法是在开始菜单——Visual Studio 所在的文件夹中打开它们，然后切换到项目的目录，运行编译指令。如果想要实现在VS Code 中点击“调试“按钮后自动编译程序，仅仅这样是行不通的。我们需要让 cl.exe 在批处理脚本(\*.bat,\*.cmd)中可用，然后才能在VS Code的调试配置文件中使用 cl.exe 编译c++源码，以及使用VS Code 调试 C++ 程序。

## 配置 VS 命令行编译环境

打开Visual Studio 开发人员 PowerShell的路径，一般为`C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\Common7\Tools`，具体路径因人而异，在该路径下看到`Launch-VsDevShell.ps1`文件即可。

接下来将这个路径加入系统变量Path。点击上方地址栏，复制完整路径，在桌面上右击【此电脑】-【属性】，在弹出的窗口最下方点击【高级系统设置】，在弹出的【系统属性】窗口中点击【高级】选项卡，然后点击【环境变量】，在下方的【系统变量】中点击【Path】-【编辑】，在弹出的【编辑环境变量】窗口中点击【新建】，将复制的路径粘贴进去，保存退出。

完成后，打开命令提示符（cmd.exe），输入`call vcvars64.bat`，如果出现如下提示信息，说明配置成功

```powershell
C:\Users\SMagic>call vcvars64.bat
**********************************************************************
** Visual Studio 2019 Developer Command Prompt v16.10.2
** Copyright (c) 2021 Microsoft Corporation
**********************************************************************
[vcvarsall.bat] Environment initialized for: 'x64'

C:\Users\SMagic>
```

> 说到这儿，最近抄袭文章的挺多的，再次声明一下，本文系原创，首发于[blog.smagic.top](https://blog.smagic.top/)，同步发布于cnblogs与知乎，作者SMagic保留一切著作权，转载请注明来源与出处。

## 配置VS Code 调试环境

打开VS Code，点击【文件】-【打开文件夹】，将项目所在的文件夹打开，作为工作区。

在文件夹中创建文件`make.bat`，将下列代码粘贴在其中：

```powershell
call vcvars64.bat
cl /Zi /GX /Fe:main.exe *.cpp
```

`/Zi`的作用是启用调试信息，`/Fe:main.exe`表明输出文件为`main.exe`，可以自行更改。

点击【运行】-【添加配置】，VS Code会自动创建`.vscode\launch.json`文件，或者手动创建。将下列代码粘贴在其中：

```json
{
    "configurations": [
        {
            "name": "(Windows) 启动",
            "type": "cppvsdbg",
            "request": "launch",
            "program": "${workspaceFolder}/main.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "console": "integratedTerminal",
            "preLaunchTask": "build"
        }
    ]
}
```

`${workspaceFolder}`是工作区目录，即当前打开的文件夹。main.exe需与`make.bat`中保持一致。

在`.vscode`文件夹下创建`tasks.json`，将下列代码粘贴在其中：

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "build",
            "type": "shell",
            "command": "cd ${workspaceFolder} ; .\\make.bat",
            "group": {
                "kind": "build",
                "isDefault": true
            }
        },
    ]
}
```

`"label": "build"`中的`build`可以自行更改，但需要与`launch.json`中的`"preLaunchTask": "build"`保持一致。

现在就可以使用VS Code调试代码了。点击左侧【运行与调试】，点击上方的绿色三角形开始调试。可以在代码文件中设置断点，观察VS Code调试器是否可以命中断点。

## 参考

[适用于开发人员的命令行 shell 和提示 - Visual Studio](https://docs.microsoft.com/zh-cn/visualstudio/ide/reference/command-prompt-powershell?view=vs-2019)

[通过命令行使用 Microsoft C++ 工具集](https://docs.microsoft.com/zh-cn/cpp/build/building-on-the-command-line?view=msvc-160#command-line-tools)

[Configure VS Code for Microsoft C++](https://code.visualstudio.com/docs/cpp/config-msvc)