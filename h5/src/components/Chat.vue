<template>
  <div class="chat">
    <!-- <textarea class="chat-content" contentEditable="true" readonly v-text="chatContent" ></textarea> -->
    <ul ref="chatContent" class="chat-content" :style="chatContentStyle">
      <li v-for="(item, index) in chatRecords">
        <span class="sender sender-system" v-if="1 == item.type"><span v-text="item.sender"></span>: </span>
        <span class="sender sender-chat" v-if="2 == item.type"><span v-text="item.sender"></span>: </span>
        <span v-text="item.content"></span>
      </li>
    </ul>
    <form class="input-area" @submit.prevent="sendContent">
      <div class="send-input-box">
        <input type="text" v-model="inputContent"/>
        <button class="btn-send"></button>
      </div>
    </form>
  </div>
</template>

<script>
import WS from '../utils/ws';
export default {
  name: "Chat",
  props: {
    room: {
    },
    rows: {
      type: Number,
      default: 10,
    },
  },
  data() {
    return {
      // WebSocket 连接对象
      wsConn: null,
      // 输入区内容
      inputContent: '',
      // 聊天记录
      chatRecords: [],
      // 聊天内容区样式
      chatContentStyle: '',
      timer: null,
    };
  },
  mounted() {
    this.wsConn = new WS
    this.wsConn.onAction('im.receive', this.onReceive)
    this.wsConn.onClose((evt)=>{
      this.chatRecords.push({
        action: 'im.receive',
        sender: '系统消息',
        type: 1,
        content: '断线重连...',
      });
      if(false !== this.timer)
      {
        // this.timer = setTimeout(this.openConn, 3000);
      }
    })
    this.openConn()
    this.chatContentStyle = 'height:' + (24 * this.rows + 20) + 'px';
  },
  beforeDestroy(){
    if(this.timer)
    {
      clearTimeout(this.timer)
    }
    this.timer = false;
    this.wsConn.close();
  },
  methods: {
    openConn(){
      this.wsConn.open(process.env.VUE_APP_IM_WEBSOCKET_URL, ()=>{
        this.wsConn.sendEx('im.joinRoom', {
          roomId: this.room,
        });
      });
    },
    // 发送内容
    sendContent(){
      if('' === this.inputContent)
      {
        return;
      }
      this.wsConn.sendEx('im.send', {
        content: this.inputContent,
        roomId: this.room,
      });
      this.inputContent = '';
    },
    // 接收内容
    onReceive(data, allowRetry = true){
      const chatContent = this.$refs.chatContent;
      if(!chatContent)
      {
        if(!allowRetry)
        {
          return;
        }
        this.$nextTick(()=>{
          this.onReceive(data, false)
        })
        return;
      }
      const willScroll = chatContent.scrollHeight - chatContent.scrollTop === chatContent.clientHeight;
      if(this.chatRecords.length >= 100)
      {
        this.chatRecords.shift();
      }
      this.chatRecords.push(data);
      if(willScroll)
      {
        this.$nextTick(function(){
          chatContent.scrollTop = chatContent.scrollHeight;
        })
      }
    },
  },
};
</script>

<style lang="less" scoped>
.chat {
  display: flex;
  flex-direction:column;
  .chat-content{
    width: 100%;
    box-sizing: border-box;
    flex:auto;
    border:none;
    background:rgba(0,0,0,0.5);
    border-radius:8px;
    outline: none;
    padding: 10px;
    list-style: none;
    color: #fff;
    overflow: auto;
    margin-bottom: 6px;
    margin-top: 0;
    li{
      word-break: break-word;
      line-height: 24px;
      span{
        &.sender{
          color: #FFA81E;
          &.sender-system{
            color: #E74856;
          }
        }
      }
    }
  }
  .input-area{
    .send-input-box{
      line-height: 48px;
      display: flex;
      flex-flow:row;
      border-radius:10px;
      background-color: #E9ECF3;
      input{
        flex: 1;
        width: 100%;
        line-height: 48px;
        outline: none;
        border:none;
        font-size: 24px;
        color: #323F49;
        border-radius:12px;
        background-color: #E9ECF3;
        box-sizing: border-box;
        padding-left: 20px;
        padding-right: 0;
      }
      .btn-send{
        width: 64px;
        border:none;
        outline: none;
        background-image: url(../assets/send.png);
        background-repeat: no-repeat;
        background-position: center;
        background-size: 32px;
        border-radius:12px;
        background-color: #E9ECF3;
      }
    }
  }
}
</style>
