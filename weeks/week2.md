# Day 3 基于BossEvent包的minecraft-BedrockServer自定义阶段场景实现
## **背景前言**
minecraft-BedrockServer是游戏我的世界基岩版版本的一款测试服务端软件，BossEvent数据包是游戏中劫掠或者带有boss标签的实体使用的交互数据包，在vanilla游戏中通过这个数据包来告诉玩家Boss名称和血量或者劫掠事件当前的状态。
## **目的**
在游戏中，劫掠可以显示和监听剩余掠夺者的数量，告诉我们当前劫掠的进度，我们希望可以利用这个数据包，实现类似劫掠或者自定义度更贵的的阶段化场景功能
- window：https://releases.llvm.org/download.html
- ubuntu：sudo apt install clang-format
## **基本使用方式**

```javascript
clang-format -style=可选格式名 -dump-config > .clang-format
```
