<template>
  <div>
    <h1>{{msg}}</h1>
    <el-select v-model="value" filterable placeholder="请选择">
      <el-option
        v-for="item in options"
        :key="item.value"
        :label="item.label"
        :value="item.value">
      </el-option>
    </el-select>
    <el-input style="width:300px;" v-model="message" placeholder="请输入内容"></el-input>
    <el-button @click="sendMsg" type="primary">发送</el-button>
    <hr/>
    <el-table
      :data="tableData"
      style="width: 100%">
      <el-table-column
        :formatter="formatter"
        label="时间"
        width="180">
      </el-table-column>
      <el-table-column
        prop="fromusername"
        label="发送方"
        width="180">
      </el-table-column>
      <el-table-column
        prop="textMessage"
        label="内容">
      </el-table-column>
      <el-table-column
        prop="tousername"
        label="接收方"
        width="180">
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
  import SockJS from 'sockjs-client'
  import Stomp from 'webstomp-client'

  export default {
    name: "Zhangzq",
    data() {
      return {
        msg: "chat room",
        //path: "ws://"+ window.location.host +"/websocket/",
        path: "ws://"+ window.location.host +"/websocket/",
        websocket: null,
        message: '',
        username: this.$route.query.name,
        value: "All",
        tableData: [],
        options: []
      }
    },
    mounted() {
      if ('WebSocket' in window) {
        this.initWebSocket()

        var _that = this;

        this.websocket.onmessage = function (evt) {
          var received_msg = evt.data;
          console.log("数据已接收:" + received_msg);
          var obj = JSON.parse(received_msg);

          /*if(obj.onlineUsers==null){
            return false;
          }*/

          //1代表上线 2代表下线 3代表在线名单 4代表普通消息
          if (obj.messageType == 1) {

            _that.tableData.unshift({
              "t": _that.getTime(),
              "fromusername": "系统消息",
              "textMessage": obj.username + "上线了"
            })
          } else if (obj.messageType == 2) {

            _that.tableData.unshift({
              "t": _that.formartterTime(obj.t),
              "fromusername": "系统消息",
              "textMessage": obj.username + "下线了"
            })

            this.options = [];

            let data = [];
            data.push({
              "value": "All",
              "label": "--所有--"
            })
            for (var i = 0; i < obj.onlineUsers.length; i++) {
              if (obj.onlineUsers[i] != _that.username) {
                data.push({
                  "value": obj.onlineUsers[i],
                  "label": obj.onlineUsers[i]
                });
              }
            }
            console.log(data);
            _that.options = data;
          } else if (obj.messageType == 3) {
            this.options = [];

            let data = [];
            data.push({
              "value": "All",
              "label": "--所有--"
            })
            for (var i = 0; i < obj.onlineUsers.length; i++) {
              if (obj.onlineUsers[i] != _that.username) {
                data.push({
                  "value": obj.onlineUsers[i],
                  "label": obj.onlineUsers[i]
                });
              }
            }
            console.log(data);
            _that.options = data;
          } else if (obj.messageType == 4) {
            if(obj.fromusername!=_that.username){
              _that.tableData.unshift({
                "t": obj.t,
                "fromusername": obj.fromusername==_that.username ? "你" : obj.fromusername,
                "textMessage": obj.textMessage,
                "tousername": obj.tousername
              })
            }
          }
        }

      } else {
        alert("不支持 web socket")
      }
    },
    methods: {
      initWebSocket() {
        this.connection()
      },
      formatter(row, column) {
        var t = row.t;
        return this.formartterTime(t);
      },
      formartterTime(t) {
        if (t != undefined && t != "") {
          return t.substring(0, 4) + "/" + t.substring(4, 6) + "/" + t.substring(6, 8) + " " + t.substring(8, 10) + ":" + t.substring(10, 12) + ":" + t.substring(12, 14);
        }
        return "";
      },
      getTime() {
        var date = new Date();
        return date.getFullYear() + this.formartNum(date.getMonth() + 1) + this.formartNum(date.getDay()) + this.formartNum(date.getHours()) + this.formartNum(date.getMinutes()) + this.formartNum(date.getSeconds());
      },
      formartNum(num) {
        return num > 10 ? num : "0" + num;
      },
      sendMsg() {
        this.websocket.send(JSON.stringify({
          "message": this.message,
          "username": this.username,
          "to": this.value,
          "tousername": this.value
        }))
        this.tableData.unshift({
          "t": this.getTime(),
          "fromusername": "你",
          "textMessage": this.message,
          "tousername": this.value
        })
      },
      reloadSelect(obj) {

      },
      connection() {
        this.websocket = new WebSocket(this.path + this.username)

        this.websocket.onopen = function () {
          console.log("已经连通了websocket");
        }
      }
    }
  }
</script>

<style scoped>

</style>
