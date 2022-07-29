<template>
  <div class="login_contariner">
    <div class="login_box">
      <div class="rider">
        <img src="../assets/logo.png" alt="" />
      </div>
      <div class="login-form">
        <el-form
          label-width="80px"
          :model="userinfo"
          :rules="loginrules"
          ref="loginFormRef"
        >
          <el-form-item label="账号：" prop="username">
            <el-input v-model="userinfo.username"></el-input>
          </el-form-item>
          <el-form-item label="密码：" prop="password">
            <el-input type="password" v-model="userinfo.password"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="onSubmit">立即登录</el-button>
            <el-button @click="resetLoginForm">重置</el-button>
          </el-form-item>
        </el-form>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      userinfo: {
        username: "",
        password: "",
      },
      loginrules: {
        username: [
          { required: true, message: "请输入登录名称", trigger: "blur" },
          {
            min: 3,
            max: 10,
            message: "长度在 3 到 10 个字符",
            trigger: "blur",
          },
        ],
        password: [
          { required: true, message: "请输入登录密码", trigger: "blur" },
          {
            min: 6,
            max: 10,
            message: "长度在 6 到 10 个字符",
            trigger: "blur",
          },
        ],
      },
    };
  },
  methods: {
    resetLoginForm() {
      this.$refs.loginFormRef.resetFields();
    },
    onSubmit() {
      this.$refs.loginFormRef.validate((val) => {
        if (!val) return;
        this.$http.post("login", this.userinfo).then((res) => {
          // console.log(res.data.meta.status);
          if (res.data.meta.status !== 200) {
            return this.$message.error("登录失败");
          } else {
            this.$message.success("登录成功");
            // console.log(res.data.data.token);
            window.sessionStorage.setItem("token", res.data.data.token);
            this.$router.push("/home");
          }
        });
      });
    },
  },
};
</script>

<style lang="less" scoped>
.login_contariner {
  height: 100vh;
  background-color: #2b4b6b;
  display: flex;
  justify-content: center;
  align-items: center;
}

.login_box {
  width: 400px;
  height: 400px;
  background-color: #fff;
  border-radius: 2px;
  position: relative;
}
.rider {
  width: 120px;
  height: 120px;
  border: 10px solid #fffafa;
  border-radius: 50%;
  position: absolute;
  overflow: hidden;
  box-shadow: 0 0 10px #ddd;
  top: -50px;
  left: 120px;
  // padding: 10px;
  // background-color: #fff;
  img {
    width: 100%;
    height: 100%;
    background-color: #eee;
  }
}
.login-form {
  width: 300px;
  margin: 120px auto;
}
</style>
