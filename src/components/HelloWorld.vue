<template>
  <div class="hello">
    <el-row>
      <el-input
        placeholder="请输入用户名"
        v-model="form.name"
        style="width:300px;"
        clearable>
      </el-input>
      <el-button @click="getToRoom" type="primary">进入聊天室</el-button>
    </el-row>
  </div>
</template>

<script>
  export default {
    name: 'HelloWorld',
    data() {
      return {
        msg: 'Welcome to Your Vue.js App',
        path: "http://" + window.location.host + "/",
        // path: "http://127.0.0.1:8888",
        form: {
          name: null
        }
      }
    },
    methods: {
      getToRoom() {
        if (this.form.name == null || this.form.name == "" || this.form.name.length == 0) {
          alert("请输入用户名!");
          return false;
        } else {
          var _that = this;
          this.$axios
            .post(this.path + "/verfUSerName?username=" + this.form.name)
            .then(function (res) {
              console.log(res.data);
              if(res.data==true || res.data=="true"){
                _that.$router.push({path: '/zhangzq', query: {name: _that.form.name}});
              }else{
                alert("用户已存在！");
                return false;
              }
            })
            .catch(function (error) {
              // 请求失败处理
              console.log(error);
            });
        }
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  h1, h2 {
    font-weight: normal;
  }

  ul {
    list-style-type: none;
    padding: 0;
  }

  li {
    display: inline-block;
    margin: 0 10px;
  }

  a {
    color: #42b983;
  }
</style>
