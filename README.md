# [ARCHIVED] 如何快速集成一个在线教室 v2

**⚠️ This project is no longer maintained and has been archived.**  
Please note that this repository is now in a read-only state and will not receive any further updates or support.

We recommend using to the latest version of the **Agora Web SDK**: [Learn more](https://www.agora.io/en/products/video-call/) 

For documentation and support, please visit the [Agora Documentation](https://docs.agora.io/en/).

---

一、现在市场上的在线教室都是怎么样的![ff.png](https://cdn.nlark.com/yuque/0/2019/png/102649/1554879028921-0f212185-e4b6-4e81-8b19-79dfe3f3ed91.png#align=left&display=inline&height=492&name=ff.png&originHeight=1188&originWidth=1800&size=384823&status=done&width=746)

市面上所有的知名在线教育公司的教室都是由RTC、白板、IM 三部分组成
* 音视频：远程上课的基石。
* 白板：老师上课的课件载体。
* IM：师生信息交流的平台。

整体就是 RTC + IM +白板，从底层看来就是实时信令+实时音视频媒体。

<a name="4c7c86a4"></a>
## 二、快速实现一个实用的在线教室
<a name="5fe23e6a"></a>
### 1、引入音视频 SDK
音视频方案选择声网作为本次的技术方案，先从 https://www.agora.io/cn/download/ 下载声网最新的 SDK 备用。<br />![ee.png](https://cdn.nlark.com/yuque/0/2019/png/102649/1554878947897-b7e15504-343a-4ad2-b394-6b3db8387104.png#align=left&display=inline&height=416&name=ee.png&originHeight=844&originWidth=1512&size=395841&status=done&width=746)<br />

1. 我们选择【视频通话/视频直播 SDK】的 web 版本进行下载，本教程写作时最新版本是 v2.6.0 版本。下载下来进行解压，其中有这样一些文件：

```
├── AgoraRTCSDK-2.6.0.js

├── assets

│   ├── mute48.png

│   └── sound48.png

├── index.html

└── vendor

   ├── bootstrap.min.css

   └── jquery.js

2 directories, 6 files
```


1. 其中 AgoraRTCSDK-2.6.0.js 是 SDK 主体文件，附带还有一个简单的 demo 工程，我们可以用 chrome 浏览器打开 index.html 文件，浏览器显示如下页面

![dd.png](https://cdn.nlark.com/yuque/0/2019/png/102649/1554878856174-9106e7de-f485-4d0e-a2b8-58769236e74e.png#align=left&display=inline&height=230&name=dd.png&originHeight=480&originWidth=1554&size=119101&status=done&width=746)

1. 需要一个声网的 AppId 才可以进行下一步试验，去 https://dashboard.agora.io/cn/signup/ 注册一个项目然后创建一个测试项目，就可以获取到这个 AppId 了。

1. 去官网注册好之后，我们回到这个页面，复制 AppId 到这个输入框内，首先点击 Join 按钮，加入该 AppId 指定的测试项目的某个 channel ，channel 默认是 1000 ，这里我们使用默认值。

1. 点击后会提示是否可以使用麦克风和摄像头权限，这是为了保护用户的隐私，这里我们点击【允许】。我们发现本地摄像头的内容显示在了屏幕的右侧。

另外再打开一个浏览器窗口，重复 4 ～ 5 步骤，比较有趣的事情就发生了，我们在两个浏览器窗口上分别看到了两个视频画面，其实一个是本地画面，一个远端的画面。我们可以想象成一对一教学的场景，老师和学生可以互相看到对方的脸听到对方的声音。

<a name="fc7dfa1f"></a>
### 2、引入白板 SDK
白板方案选择 white sdk 作为本次的技术方案，文档中心：[https://developer.herewhite.com/#/](https://developer.herewhite.com/#/)
1. 我们用 cdn 引入的方式引入白板 SDK 的 javascript 文件和 css 文件。在当前的 index.html 文件的 head 标签中加入

```html
<title>Agora Web Sample</title>
<!-- 新加入代码开始 -->
<link rel="stylesheet" href="https://sdk.herewhite.com/white-web-sdk/2.0.0-beta.3.css">
<script src="https://sdk.herewhite.com/white-web-sdk/2.0.0-beta.3.js"></script>
<!-- 新加入代码结束 -->
```

1. 加入一个特定的白板需要 uuid 和 token 两个参数，我们在 body 标签中放置两个 input 用于输入这两个参数，加入一个按钮用于加入房间，在原有的 button 下面加入如下代码：

```html
<button id="unpublish" class="btn btn-primary" onclick="unpublish()">Unpublish</button>
<!-- 新加入代码开始 -->
Room UUID: <input id="room_uuid" type="text" size="32"></input>
Room Token: <input id="room_token" type="text" size="32"></input>
<button id="join_room" class="btn btn-primary" onclick="join_room()">Join Whiteborad</button>
<!-- 新加入代码结束 -->
```

1. 在原有 javascript 代码中加入 join_room 函数，逻辑也是比较简单的：
  1. 创建 WhiteWebSdk 对象
  1. 调用 joinRoom 方法加入某个特定的白板，这个白板由前面两个 input 框中的参数确定，uuid 为全局确定一个白板，token 则是加入这个白板的必备验证信息，当调用成功结束后得到 room 对象，room 对象持有对白板操作的一系列 API ，这里把他 room 绑定在 id 为 whiteboard 的 div 上。
```javascript
function join_room() {
  document.getElementById("join_room").disabled = true;
  var whiteWebSdk = new WhiteWebSdk();
  whiteWebSdk.joinRoom({
    uuid: room_uuid.value,
    roomToken: room_token.value,
  }).then(function(room) {
    room.bindHtmlElement(document.getElementById('whiteboard'));
  });
}
```
1. 我们在 body 中加入一个 div 用来容纳白板吧，白板成功加入后就会显示在这个 400px 宽、300px 高的 div 中了。

```html
<body>
<!-- 新加入代码开始 -->
<div id="whiteboard" style="width: 400px; height: 300px;"></div>
<!-- 新加入代码结束 -->
```

1. 步骤 3 中的 uuid 和 room token 是从哪里来的呢？首先请前往 https://console.herewhite.com 注册一个开发者账户，你就获取到一个 sdk token ，通过 sdk token 就可以调用 REST API 创建一个房间了。我们在 javascript 文件的开头加上如下代码。
  1. 通过 REST API  [https://cloudcapiv4.herewhite.com/room](https://cloudcapiv4.herewhite.com/room) 创建一个房间，返回值就是熟悉的 uuid 和 room token 了
  1. 我们把他们赋给前面的两个 input 框，方便查看和记录。
```javascript
<script language="javascript">
// 新加入代码开始
var sdkToken = 'WHITEcGFydG5lcl9pZD1DYzlFNTJhTVFhUU5TYmlHNWJjbkpmVThTNGlNVXlJVUNwdFAmc2lnPTE3Y2ZiYzg0ZGM5N2FkNDAxZmY1MTM0ODMxYTdhZTE2ZGQ3MTdmZjI6YWRtaW5JZD00JnJvbGU9bWluaSZleHBpcmVfdGltZT0xNTY2MDQwNjk4JmFrPUNjOUU1MmFNUWFRTlNiaUc1YmNuSmZVOFM0aU1VeUlVQ3B0UCZjcmVhdGVfdGltZT0xNTM0NDgzNzQ2Jm5vbmNlPTE1MzQ0ODM3NDYzMzYwMA';

var url = 'https://cloudcapiv4.herewhite.com/room?token=' + sdkToken;
var requestInit = {
  method: 'POST',
  headers: {
    "content-type": "application/json",
  },
  body: JSON.stringify({
    name: '我的第一个 White 房间',
    limit: 100, // 房间人数限制
  }),
};

fetch(url, requestInit)
  .then(function (response) {
    return response.json();
  })
  .then(function (json) {
    room_uuid.value = json.msg.room.uuid;
    room_token.value = json.msg.roomToken;
    console.log("room uuid", json.msg.room.uuid, json.msg.roomToken);
  })
//  新加入代码结束
```

1. 重新用浏览器打开 index.html ，上半部分的空白则是白板的部分，我们点击【Join Whiteborad】按钮，成功加入白板后就可以使用鼠标在白板上进行涂写了。

<a name="aa7b688d"></a>
### 3、Demo 效果
<a name="56363c3c"></a>
#### 1、加入前
![cc.png](https://cdn.nlark.com/yuque/0/2019/png/102649/1554878710061-8c7eaa80-578a-4fb8-af76-ac2c2605321a.png#align=left&display=inline&height=340&name=cc.png&originHeight=389&originWidth=854&size=60554&status=done&width=746)<br />
<a name="f41f45e5"></a>
#### 2、加入后

<br />![bb.png](https://cdn.nlark.com/yuque/0/2019/png/102649/1554878733913-07591207-54c1-4f71-9081-9430141af420.png#align=left&display=inline&height=509&name=bb.png&originHeight=584&originWidth=856&size=83446&status=done&width=746)<br />

<a name="c8ad4c49"></a>
#### 3、体验互动课堂
1、我们打开浏览器的另一个窗口，将上一窗口中的 room uuid 和 room token 复制并覆盖新窗口中的值，点击新窗口中的【Join Whiteborad】按钮，则两个窗口加入到同一块白板中，任何一个窗口的涂写都瞬间在另一个窗口中显现。<br />2、我们看看最终的效果吧。<br />![aa.png](https://cdn.nlark.com/yuque/0/2019/png/102649/1554878746542-9ce201dd-038c-48f8-9c55-25baccb63f7a.png#align=left&display=inline&height=484&name=aa.png&originHeight=707&originWidth=1089&size=392072&status=done&width=746)<br />

<a name="5d88b18a"></a>
##  三、资料链接
[https://github.com/netless-io/Edu-Classroom-Demo](https://github.com/netless-io/Edu-Classroom-Demo)
