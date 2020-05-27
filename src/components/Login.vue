<template>
  <div id="login">
    <el-form status-icon ref="form"  :rules="rules" :model='form' label-width="80px">
      <img src="../assets/机器猫.jpeg" alt="">
      <el-form-item label="用户名" prop='username'>
        <el-input v-model="form.username" placeholder="请输入用户名" @keyup.enter.native='login'></el-input>
      </el-form-item>
      <el-form-item label="密码" prop='password'>
        <el-input v-model="form.password" placeholder="请输入密码" type='password' @keyup.enter.native='login'></el-input>
      </el-form-item>
      <el-form-item>
        <el-button class="login" type="primary" @click='login'>登录</el-button>
        <el-button @click="resetForm('form')">重置</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  data () {
    return {
      form: {
        username: '',
        password: ''
      },
      rules: {
        username: [
          { required: true, message: '请输入用户名', trigger: ['blur', 'change'] },
          { min: 3, max: 12, message: '用户名长度在 3 到 12 个字符', trigger: ['blur', 'change'] }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: ['blur', 'change'] },
          { min: 3, max: 12, message: '密码长度在 3 到 12 个字符', trigger: ['blur', 'change'] }
        ]
      }
    }
  },
  methods: {
    resetForm (formName) {
      this.$refs[formName].resetFields()
    },
    async login () {
      try {
        await this.$refs.form.validate()
        const { meta, data } = await this.$axios.post('login', this.form)
        if (meta.status === 200) {
          localStorage.setItem('token', data.token)
          this.$message.success(meta.msg)
          this.$router.push('/index')
        } else {
          this.$message.error(meta.msg)
        }
      } catch (e) {
        console.log(e)
      }
    }
  }
}
</script>

<style lang='scss'>
  #login {
    overflow: hidden;
    width: 100%;
    height: 100%;
    background-color: #2d434c;
    .el-form {
      width: 400px;
      background-color: #fff;
      margin: 200px auto;
      position: relative;
      padding: 75px 20px 20px;
      border-radius: 15px;
      .login {
        margin-right: 70px;
      }
      img {
        width: 120px;
        height: 120px;
        position: absolute;
        transform: translateX(-50%);
        left: 50%;
        top: -60px;
        border-radius: 50%;
        border: 4px solid #fff;
      }
    }
  }
</style>
