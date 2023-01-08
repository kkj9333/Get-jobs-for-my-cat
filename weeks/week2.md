# Day 3 基于BossEvent包的minecraft-BedrockServer自定义阶段场景实现
## **背景前言**
minecraft-BedrockServer是游戏我的世界基岩版版本的一款测试服务端软件，BossEvent数据包是游戏中劫掠或者带有boss标签的实体使用的交互数据包，在vanilla游戏中通过这个数据包来告诉玩家Boss名称和血量或者劫掠事件当前的状态。
## **目的**
在游戏中，劫掠可以显示和监听剩余掠夺者的数量，告诉我们当前劫掠的进度，我们希望可以利用这个数据包，实现类似劫掠或者自定义度更高的阶段化场景功能.
## **准备工作**
由于缺乏相应的api，我们首先需要对vanilla游戏的劫掠的收发包工作进行监听，为后面模仿实现功能提供数据支撑，这里我们选择用导出API进行监听。
我们首先对劫掠刚刚触发时的相关数据包进行监听： <br>
![image](https://user-images.githubusercontent.com/51207072/211181452-3c0e9313-ca24-4c72-a14c-fe27c3c6ca0a.png)<br>
不难看出劫掠中会首先广播一个StartShow的BossEvent包给客户端，同时更新村庄钟的状态表示已进入劫掠，客户端收到BossEvent包会向服务端发送一个请求包请求注册玩家到当前的劫掠事件中（注意bossUniqueID和Maybe_PlayerUniqueID是一样的)，随后服务端会会处理注册；然后向已注册事件的玩家发送Title和HealthPercentage来更新目前劫掠的标题和加载百分比，这个过程是一直持续的，作为告知玩家当前劫掠进度的方式，我们称其为状态更新机制，频率是每tick一次<br>
![image](https://user-images.githubusercontent.com/51207072/211183093-5af5a5df-5d13-4aba-98bc-c9232a9ecadf.png)<br>
当玩家离开劫掠区域时，服务端会主动发送玩家Hide类型的BossEvent包，玩家收到后会发送UnRegisterPlayer的回执给服务端，至此玩家就从劫掠事件中移除了。<br>
劫掠事件中也会监听相关实体变化从而更新劫掠状态，例如最后两个实体时发送的劫掠血量和标题名称会改变，这是之前说明的状态更新机制实现的 <br>
当玩家完成一波劫掠时，名称和标题都会发生更改，同时会更新村庄钟的状态： <br>
![image](https://user-images.githubusercontent.com/51207072/211183483-4a7328ac-a9dc-47b8-ad34-08030c1d46be.png) <br>
但劫掠胜利或失败时（这里选取失败） ：<br>
![image](https://user-images.githubusercontent.com/51207072/211183584-26e5b3bd-74c5-4e48-be04-fe9b1ea000be.png)<br>
失败和胜利时，状态更新机制告知了玩家，但是劫掠事件并没有立即结束，而是选择继续广播数据，并在一段事件后注销劫掠事件：<br>
![image](https://user-images.githubusercontent.com/51207072/211183627-c16b0e2d-18cc-4b44-ab14-60215dcff381.png)<br>
## **启示和反思**
其实不难发现在这个过程中bedrockserver存在很多迷惑操作，首先是状态更新机制，为什么在数据不变时要持续向客户端通知标题和进度百分比？这样是否额外浪费了资源？其次在事件结束时，状态更新机制仍在运作，但是后续标题和百分比并不会发生变化，理论上没必要进行状态更新告知；如果是担心新进入或者突然退出的玩家无法及时获取最新状态，只需在进入或退出的时候发生更新状态的包就行了，对于离开的玩家直接Hide就可以。
```javascript
clang-format -style=可选格式名 -dump-config > .clang-format
```
