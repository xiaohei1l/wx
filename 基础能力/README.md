## 基础能力-非原创-学习使用
### 云托管
#### 1.云开发平台->云托管
#### 2.创建容器express-test
#### 3.小程序调用
```
const c1 = new wx.cloud.Cloud({
  resourceEnv: 环境ID
})
await c1.init()
const r = await c1.callContainer({
  path: '/api/users', //此处填入业务自定义路径
  header: {
    'X-WX-SERVICE': 'express-test', //填入业务名称
  },
  //其余参数同 wx.request
  method: 'GET',
})
```
### 云函数
#### 1)获取Openld
#### 1)step1 quickStartFunctions 云函数代码
```
const cloud = require('wx-server-sdk');
cloud.init({
  env: cloud.DYNAMIC_CURRENT_ENV
});
//获取openID云函数入口函数
exports.main = async (event, context) => {
  //获取基础信息
  const wxContext = cloud.getWXContext();
  return {
    openid: wxContext.OPENID,
    appid: wxcontext.APPID,
    unionid: wxContext.UNIONID,
  };
};
```
#### 1)step2 小程序代码段
```
wx.cloud.callFunction({
  name: 'quickstartFunctions',
  data: {
    type: 'getOpenID'
  }
}).then((resp) => console.log(resp))
```
#### 2)生成小程序
#### 2)step1 quickStartFunctions 云函数代码
```
const cloud = require('wx-server-sdk');
cloud.init({
  env: cloud.DYNAMIC_CURRENT_ENV
});
//获取小程序二维码云函数入口函数
exports.main = async (event, context) => {
  //获取小程序二维码的buffer
  const resp = await cloud.openapi.wxacode.get({
    path: 'pages/index/index'
  });
  const { buffer } = resp;
  //将图片上传云存储空间
  const upload = await cloud.uploadFile({
    cloudPath: 'code.png',
    fileContent: buffer
  });
  return upload.fileID;
};
```
#### 2)step2小程序代码段
```
wx.cloud.callFunction({
  name: 'quickstartFunctions',
  data: {
    type: 'getMiniProgramCode'
  }
}).then((resp) => console.log(resp))
```
### 数据库
#### 在‘cloudfunctions/quickstarFunctions'目录右键，选择[上传并部署-云端安全依赖】，等待云函数上传完成后重试。

### 云存储 自带CDN加速文件存储
#### 上传文件
```
wx.chooseMedia({
count: 1,
success: (chooseResult) => {
  //将图片上传至云存储空间
  wx.cloud
    .uploadFile({
      //指定上传到的云路径
      cloudPath: "my-photo.png",
      //指定要上传的文件的小程序临时文件路径
      filePath: chooseResult.tempFiles[0].tempFilePath,
    })
    .then((res) => {
      console.log(res)
    })
    .catch((e) => {
      console.log('e', e)
    });
}
});
```
### 扩展能力-AI
#### 大模型对话指引_集成Agent-UI组件指引
#### step1 拷贝组件源码包
https://gitee.com/TencentCloudBase/cloudbase-agent-ui/tree/main/apps/miniprogram-agent-ui/miniprogram/components/agent-ui
#### step2 将组件拷贝至小程序目录中
##### 跟cloudfunctions同级，文件夹为components目录下agent-ui文件夹
#### step3 在页面.json配置文件中注册组件
```
{
  "usingComponents": {
    "agent-ui":"/components/agent-ui/index"
  },
}
```
#### step4 在页面.wxml文件中引用组件
```
<agent-ui agentConfig="{{agentConfig}}" showBotAvatar="{{showBotAvatar}}" chatMode="{{chatMode}}" modelConfig="{{modelConfig}}""></agent-ui>
```
#### step4 在页面.js文件中编写配置
```
data: {
  chatMode: "bot", //bot表示使用agent,midel 表示使用大模型
  showBotAvatar: true, //是否在对话框左侧显示头像
  agentConfig: {
    botID: "your agent id", //agent id,
    allowWebSearch: true, //允许客户端选择展示联网搜索按钮
    allowUploadFile: true, //允许客户端展示上传文件按钮
    allowPullRefresh: true, //  允许客户端展示下拉刷新
    allowUploadImage: true, //允许客户端展示上传图片按钮
    allowMultiConversation: true, //允许客户端展示查看会话列表/新建会话按钮
    showToolCallDetail: true, //是否展示 mcp server toolCall 细节
    allowVoice: true, //允许客户端展示语音按键
    showBotName: true, //允许展示bot名称
  },
  modelConfig: {
    modelProvider: "hunyuan-open", //大模型服务厂商
    quickResponseModel: "hunyuan-lite", //大模型名称
    logo: "", //model 头像
    welcomeMsg: "欢迎语", //model 欢迎语
  },
}
```

