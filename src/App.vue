<template>
  <div id="app">
    <div v-if="!user.name">
      <div class="user-box">
        <input class="user-input" placeholder="请输入用户名" v-model="inputName">
        <button class="base-button" @click="createUser(inputName)">确定</button>
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
          <div class="chat-list">
            <div class="chat-item chat-item-selected">聊天1</div>
            <div class="chat-item">聊天2</div>
          </div>
        </div>
        <div class="main">
          <div class="main-header">
            聊天1
          </div>
          <div class="main-content">
            <div class="msg-list" v-for="msg in msgList">
              <div class="my-msg-item" v-if="msg.sendUser.name === user.name">
                <span class="my-msg bubble">{{ msg.content }}</span>
                <span class="my-name">{{ user.name }}</span>
              </div>
              <div class="other-msg-item" v-if="msg.sendUser.name !== user.name">
                <span class="other-name">{{ user.name }}</span>
                <span class="other-msg">{{ msg.content }}</span>
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
      user: {
        name: ''
      },
      msgContent: '',
      msgList: [],
    }
  },
  methods: {
    createUser(name) {
      this.user.name = name;
    },
    sendMsg() {
      let date = new Date();
      let msg = {
        content: this.msgContent,
        sendUser: this.user,
        homeNo: 0,
        date: date
      };
      this.$socket.emit("sendMsg", msg);
      this.msgList.push(msg);
      console.log(this.msgList);
      this.msgContent = '';
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
    width: 80%;
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
    padding: 18px;
  }
  .sidebar .name {
    font-size: 16px;
    text-align: left;
    height: 40px;
    line-height: 40px;
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
    border-bottom: 1px solid #d6d6d6;
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
    padding: 10px 15px;
    box-sizing: border-box;
  }
  .my-msg-item  .my-msg{
    background: #b2e281;
    color: #fff;
  }
  .my-msg-item .my-name {
    line-height: 30px;
    height: 30px;
  }
  .bubble {
    max-width: 90%;
    height: 30px;
    display: inline-block;
    vertical-align: top;
    position: relative;
    text-align: left;
    font-size: 14px;
    line-height: 30px;
    border-radius: 3px;
    -moz-border-radius: 3px;
    -webkit-border-radius: 3px;
    margin: 0 10px;
    padding: 0 10px;
  }
</style>
