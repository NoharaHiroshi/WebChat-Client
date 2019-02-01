<template>
  <div id="app">
    <div class="msg-box">
      <div class="sidebar">
        <div class="user">
          <div class="name">
            Lands
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

        </div>
        <div class="main-toolbar">
          <span class="emoticon"></span>
          <span class="upload"></span>
        </div>
        <textarea class="main-input"></textarea>
        <div class="main-action">
          <button class="submit-msg">发送</button>
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

    }
  },
  methods: {

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
  .msg-box {
    position: relative;
    width: 70%;
    height: 550px;
    margin: 0 auto;
    background: aliceblue;
    border-radius: 5px;
  }
  .sidebar {
    position: absolute;
    left: 0;
    top: 0;
    display: inline-block;
    width: 30%;
    height: 100%;
    background: #2e3238;
    color: #fff;
    box-sizing: border-box;
  }
  .sidebar .user {
    padding: 18px;
  }
  .sidebar .name {
    font-size: 16px;
    float: left;
    margin-bottom: 10px;
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
    padding: 12px 15px;
    border-bottom: 1px solid #292c33;
    cursor: pointer;
    -webkit-transition: background-color .1s;
    transition: background-color .1s;
  }
  .sidebar .chat-item:hover {
    background-color: hsla(0,0%,100%,.03);
  }
  .sidebar .chat-item-selected {
    background-color: hsla(0,0%,100%,.1);
  }
  .main {
    display: inline-block;
    position: absolute;
    width: 70%;
    left: 30%;
    top: 0;
    height: 100%;
    background: #eee;
  }
  .main .main-header {
    position: relative;
    padding: 10px 0;
    margin: 0 19px;
    border-bottom: 1px solid #d6d6d6;
    background-color: #eee;
    font-size: 14px;
    height: 5%;
    line-height: 30px;
  }
  .main .main-content {
    position: relative;
    height: 60%;
    border-bottom: 1px solid #d6d6d6;
  }
  .main .main-toolbar {
    padding: 5px 17px;
    box-sizing: border-box;
    overflow: hidden;
    text-align: left;
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
    padding-left: 20px;
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
    padding: 10px;
    text-align: right;
  }
  .submit-msg {
    background-color: #fff;
    color: #222;
    display: inline-block;
    border: 1px solid #c1c1c1;
    padding: 3px 20px;
    font-size: 14px;
    border-radius: 4px;
    height: 30px;
  }
</style>
