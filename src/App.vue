<template>
  <div id="app">
    <div v-if="!user.name">
      <div class="user-box">
        <input class="user-input" placeholder="请输入用户名" v-model="inputName">
        <input class="user-input" type="password" placeholder="请输入密码" v-model="inputPassword">
        <button class="base-button" @click="login()">确定</button>
      </div>
    </div>
    <div v-if="user.name">
      <div class="msg-box">
        <div class="sidebar">
          <div class="user">
            <div class="name">
              <span>{{user.name}}</span>
            </div>
            <div class="search">
              <input class="base-input" placeholder="搜索">
            </div>
          </div>
          <div class="tab">
            <div class="tab-item">
              <span :class="tab === 1?'tab-chat-icon-selected':'tab-chat-icon'" @click="tab = 1"></span>
            </div>
            <div class="tab-item">
              <span :class="tab === 0?'tab-user-icon-selected':'tab-user-icon'" @click="tab = 0"></span>
            </div>
          </div>
          <div class="chat-list" v-if="tab === 0">
            <div v-for="user in onlineUserList">
              <div class="chat-item" @click="chatWithUser(user.id)">{{user.name}}</div>
            </div>
          </div>
          <div class="chat-list" v-if="tab === 1">
            <div v-for="home in historyHomeIds">
              <div class="chat-item">{{home.userName}}</div>
            </div>
          </div>
        </div>
        <div class="main">
          <div class="main-header">
            {{currentHomeName}}
          </div>
          <div class="main-content">
            <div class="msg-list" v-for="msg in msgList">
              <div class="my-msg-item" v-if="msg.fromUserId === user.id">
                <span class="my-msg bubble">{{ msg.msgContent }}</span>
                <span class="my-name">{{ user.name }}</span>
              </div>
              <div class="other-msg-item" v-if="msg.fromUserId !== user.id">
                <span class="other-name">{{ msg.fromUserName }}</span>
                <span class="other-msg bubble">{{ msg.msgContent }}</span>
              </div>
            </div>
          </div>
          <div class="main-toolbar">
            <span class="emoticon"></span>
            <span class="upload"></span>
          </div>
          <textarea class="main-input" v-model="msgContent"></textarea>
          <div class="main-action">
            <button class="submit-msg" @click="sendMsg()">发送</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      inputName: '',
      inputPassword: '',
      user: {
        name: '',
        id: '',
        status: 0
      },
      onlineUserList: [],
      msgContent: '',
      msg: {
        msgContent: '',
        fromUserId: '',
        fromUserName: '',
        sendMsgDate: '',
        homeId: ''
      },
      msgList: [],
      // 0用户，1聊天
      tab: 0,
      // 当前聊天窗口打开的会话
      currentHomeId: '',
      currentHomeName: '',
      historyHomeIds: [],
      webSocket: null
    }
  },
  created() {

  },
  methods: {
    login() {
      let params = {
        name: this.inputName,
        password: this.inputPassword
      };
      let url = "/api/login";
      let self = this;
      this.axios.post(url, params).then(function (res) {
        let data = res.data;
        if (data.code === 0){
          console.log(data);
          self.user = {
            name: data.result.name,
            id: data.result.id
          };
          console.log(self.user);
          let socketUrl = "ws://127.0.0.1:8088/webSocket/" + self.user.id;
          self.initWebSocket(socketUrl);
        }else {
          console.log("登录失败");
        }
      }).catch(function (error) {

      });
    },
    initWebSocket(socketUrl) {
      let self = this;
      self.webSocket = new WebSocket(socketUrl);
      self.webSocket.onmessage = self.receiveMsg;
      self.user.status = 1;
      self.getHistoryChat();
    },
    sendMsg() {
      if(this.currentHomeId){
        let date = new Date();
        this.msg = {
          msgContent: this.msgContent,
          fromUserId: this.user.id,
          fromUserName: this.user.name,
          sendMsgDate: date,
          homeId: this.currentHomeId
        };
        this.webSocket.send(JSON.stringify(this.msg));
        this.msgList.push(this.msg);
        console.log(this.msg);
        this.msgContent = '';
      } else {
        console.log("请选择会话");
      }
    },
    // 用户登录推送
    loginMsgHandler(msg){
      let user = {
        id: msg.user.id,
        name: msg.user.name
      };
      this.onlineUserList.push(user);
    },
    // 获取当前登录用户
    onlineUserHandler(msg){
      for(let item of msg.userList){
        let user = {
          id: item.id,
          name: item.name
        };
        this.onlineUserList.push(user);
      }
    },
    // 用户退出推送
    logoutMsgHandler(msg){
      let self = this;
      let user = {
        id: msg.user.id,
        name: msg.user.name
      };
      for(let i=0; i<self.onlineUserList.length; i++){
        if(self.onlineUserList[i].id === user.id){
          self.onlineUserList.splice(i, 1);
        }
      }
    },
    receiveMsg(e) {
      let msg = JSON.parse(e.data);
      if(msg.msgType){
        if(msg.msgType === "login"){
          this.loginMsgHandler(msg);
        }
        if(msg.msgType === "logout"){
          this.logoutMsgHandler(msg);
        }
        if(msg.msgType === "onlineUser"){
          this.onlineUserHandler(msg);
        }
      }else{
        this.msgList.push(msg);
      }
    },
    // 获取历史会话
    getHistoryChat() {
      let self = this;
      let params = {
        userId: this.user.id
      };
      let url = "/api/chat/getHistoryHome";
      this.axios.post(url, params).then(function (res) {
        let data = res.data;
        if(data.code === 0){
          self.historyHomeIds = data.result;
        }
      });
    },
    // 创建聊天
    chatWithUser(userId) {
      let self = this;
      let params = {
        userId: this.user.id,
        oUserId: userId
      };
      let url = "/api/chat/connectChatHome";
      this.axios.post(url, params).then(function (res) {
        let data = res.data;
        if(data.code === 0){
          let homeInfo = data.result;
          let existHomeIds = [];
          for(let existHome of self.historyHomeIds){
            existHomeIds.push(existHome.homeId);
          }
          if(existHomeIds.indexOf(homeInfo.homeId) === -1){
            self.historyHomeIds.push(homeInfo);
          }
          // 创建房间后，自动进入房间界面
          self.tab = 1;
          self.currentHomeId = homeInfo.homeId;
          self.currentHomeName = homeInfo.userName;
        }
      });
    }
  }
}
</script>

<style>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
  .base-button {
    border: none;
    border-radius: 2px;
    width: 80px;
    height: 30px;
    background: #0fb2cc;
    color: #fff;
  }
  .user-box {
    position: relative;
    width: 50%;
    height: 50px;
    line-height: 50px;
    margin: 0 auto;
    border-radius: 5px;
    background: #2e3238;
  }
  .user-box .user-input {
    padding: 0 10px;
    width: 40%;
    font-size: 12px;
    color: #fff;
    height: 30px;
    line-height: 30px;
    border: 0;
    border-radius: 4px;
    outline: none;
    background-color: transparent;
    box-sizing: border-box;
  }
  .msg-box {
    position: relative;
    width: 70%;
    height: 570px;
    margin: 0 auto;
    border-radius: 5px;
  }
  .msg-box .base-input {
    padding: 0 10px;
    width: 100%;
    font-size: 12px;
    color: #fff;
    height: 30px;
    line-height: 30px;
    border: 0;
    border-radius: 4px;
    outline: none;
    background-color: #26292e;
    box-sizing: border-box;
  }
  .sidebar {
    float: left;
    width: 30%;
    height: 100%;
    background: #2e3238;
    color: #fff;
    box-sizing: border-box;
    border-radius: 5px 0 0 5px;
  }
  .sidebar .user {
    padding: 18px 18px 10px;
  }
  .sidebar .name {
    font-size: 16px;
    text-align: left;
    height: 40px;
    line-height: 40px;
  }
  .sidebar .tab {
    overflow: hidden;
    padding: 0 18px 10px;
  }
  .sidebar .tab-item {
    float: left;
    display: inline-block;
    width: 50%;
    padding: 10px;
    box-sizing: border-box;
  }
  .sidebar .tab-chat-icon {
    background: url("./assets/chat_white.png");
    background-size: 100% 100%;
    width: 25px;
    height: 25px;
    display: inline-block;
    vertical-align: middle;
  }
  .sidebar .tab-chat-icon:hover {
    background: url("./assets/chat_hover.png");
    background-size: 100% 100%;
  }
  .tab-chat-icon-selected {
    background: url("./assets/chat_hover.png");
    background-size: 100% 100%;
    width: 25px;
    height: 25px;
    display: inline-block;
    vertical-align: middle;
  }
  .sidebar .tab-user-icon {
    background: url("./assets/user_white.png");
    background-size: 100% 100%;
    width: 25px;
    height: 25px;
    display: inline-block;
    vertical-align: middle;
  }
  .sidebar .tab-user-icon:hover {
    background: url("./assets/user_hover.png");
    background-size: 100% 100%;
  }
  .tab-user-icon-selected {
    background: url("./assets/user_hover.png");
    background-size: 100% 100%;
    width: 25px;
    height: 25px;
    display: inline-block;
    vertical-align: middle;
  }

  .sidebar .base-input {
    padding: 0 10px;
    width: 100%;
    font-size: 12px;
    color: #fff;
    height: 30px;
    line-height: 30px;
    border: 0;
    border-radius: 4px;
    outline: none;
    background-color: #26292e;
    box-sizing: border-box;
  }
  .sidebar .chat-item {
    height: 40px;
    line-height: 40px;
    border-bottom: 1px solid #292c33;
    cursor: pointer;
    -webkit-transition: background-color .1s;
    transition: background-color .1s;
  }
  .sidebar .chat-item:hover {
    background-color: hsla(0,0%,100%,.03);
  }
  .sidebar .chat-item-selected {
    background-color: hsla(0,0%,100%,.1) !important;
  }
  .main {
    float: left;
    width: 70%;
    height: 100%;
    background: #eee;
    border-radius: 0 5px 5px 0;
  }
  .main .main-header {
    position: relative;
    height: 50px;
    line-height: 50px;
    border-bottom: 1px solid #d6d6d6;
    font-size: 14px;
  }
  .main .main-content {
    position: relative;
    height: 350px;
    padding: 10px;
    border-bottom: 1px solid #d6d6d6;
    box-sizing: border-box;
  }
  .main .main-toolbar {
    height: 30px;
    line-height: 30px;
    box-sizing: border-box;
    overflow: hidden;
    text-align: left;
    margin: 5px 0;
  }
  .main .main-input {
    height: 15%;
    overflow-y: auto;
    overflow-x: hidden;
    outline: none;
    border: 0;
    font-size: 16px;
    width: 100%;
    box-sizing: border-box;
    padding-left: 15px;
    background: transparent;
    resize: none;
  }
  .main .emoticon {
    width: 30px;
    height: 30px;
    background: url(./assets/weixin-ui.png) no-repeat -404px -398px;
    -webkit-background-size: 487px 462px;
    background-size: 487px 462px;
    display: inline-block;
    vertical-align: middle;
    cursor: pointer;
    margin-left: 10px;
  }
  .main .upload {
    width: 30px;
    height: 30px;
    background: url(./assets/weixin-ui.png) no-repeat -120px -432px;
    -webkit-background-size: 487px 462px;
    background-size: 487px 462px;
    display: inline-block;
    vertical-align: middle;
    cursor: pointer;
  }
  .main .main-action {
    height: 30px;
    line-height: 30px;
    text-align: right;
  }
  .submit-msg {
    background-color: #fff;
    color: #222;
    display: inline-block;
    border: 1px solid #c1c1c1;
    font-size: 14px;
    border-radius: 4px;
    height: 30px;
    width: 90px;
    margin-right: 20px;
  }
  .msg-list {
    margin: 0 auto;
  }
  .msg-list .my-msg-item {
    width: 100%;
    text-align: right;
    padding: 5px 10px;
    box-sizing: border-box;
  }
  .my-msg-item  .my-msg{
    background: #b2e281;
    color: #fff;
  }
  .my-msg-item .my-name {
    line-height: 35px;
    height: 35px;
  }
  .msg-list .other-msg-item {
    width: 100%;
    text-align: left;
    padding: 5px 10px;
    box-sizing: border-box;
  }
  .other-msg-item .other-name {
    line-height: 35px;
    height: 35px;
  }
  .other-msg-item .other-msg {
    background: #fff;
    color: #333;
  }
  .bubble {
    max-width: 90%;
    height: 35px;
    display: inline-block;
    vertical-align: top;
    position: relative;
    text-align: left;
    font-size: 14px;
    line-height: 35px;
    border-radius: 3px;
    -moz-border-radius: 3px;
    -webkit-border-radius: 3px;
    margin: 0 10px;
    padding: 0 13px;
  }
</style>
