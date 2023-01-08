# Day 3 了解Clangformat用法并配置成功生成项目
## **什么是clang-format?**
Clang-Format可用于格式化（排版）多种不同语言的代码。
其自带的排版格式主要有：LLVM, Google, Chromium, Mozilla, WebKit等
## **安装**
clang-format本质是一个命令行工具
- window：https://releases.llvm.org/download.html
- ubuntu：sudo apt install clang-format
## **基本使用方式**
```javascript
clang-format [options] [<file> ...]
clang-format --help //建议至少浏览一遍帮助信息。
```
## **Clang-format导出格式**
在工程项目中，可以使用.clang-format or _clang-format格式的文件来自定义规范格式，将file放入项目里代码文件相同的根目录下：<br>
- 1.目录运行cmd 输入clang-format -style=file.
- 2.如果是VS等支持clang-format的，则不需要额外设置，VS会自动检查项目根目录下的clangformat文件
一个创建clang-format文件的简单方式是：
```javascript
clang-format -style=可选格式名 -dump-config > .clang-format
```
