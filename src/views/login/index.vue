<template>
    <div class="login-wrap">
         <div class="login-form-wrap">  
               <div class="login-head">
                   <img src="./logo_index.png" alt="黑马头条">
                </div> 
        <el-form ref="form" :model="form" :rules="rules" label-width="80px">
  <el-form-item label="手机号码" prop="mobile">
    <el-input v-model="form.mobile"></el-input>
  </el-form-item>
    <el-form-item label="验证码" prop="code">
    <el-col :span='10'>
          <el-input v-model="form.code"></el-input>
    </el-col>
    <el-col :span='10' :offset="2">
         <el-button @click="handleCodeSend">发送验证码</el-button>
    </el-col>
  </el-form-item>
  <el-form-item>
    <el-button 
    class="btn-lodin" 
    type="primary"
     @click="onSubmit"
     :loading="loginLoading"
     >立即登录</el-button>
  </el-form-item>
        </el-form>
    </div>
    </div>
</template>

<script>
import axios from 'axios'
import "@/vendor/gt"
export default {
    name:'AppLogin',
  data() {
      return {
        form: {
          mobile:'18830440236',
          code:''
        },
         loginLoading:false,
         rules: {
          mobile: [
            { required: true, message: '请输入电话号码', trigger: 'blur' },
            { min: 11, max: 11, message: '长度在 11 个字符', trigger: 'blur' }
          ],
           code: [
            { required: true, message: '请输入验证码', trigger: 'blur' },
            { min: 6, max: 6, message: '长度在 6 个字符', trigger: 'blur' }
          ],
         },
        captchaObj: null
      }
    },
    methods: {
      onSubmit() {
        this.$refs['form'].validate(valid =>{
          if(!valid) {
            return
          }
          this.login()
        })
       
      },
      login(){
        this.loginLoading=true
         axios({
          method:'POST',
          url:'http://ttapi.research.itcast.cn/mp/v1_0/authorizations',
          data:this.form
        }).then(res => {
          this.loginLoading = false
          this.$router.push({
            name: 'home'
          })
        }).catch(err =>{
            // if(err.response.status ===400){
              this.$message.error('登录失败，手机号或者验证码错误')
            // }
            this.loginLoading=false
          })
      },
      handleCodeSend () {
  const { mobile } = this.form
  axios({
    method: 'GET',
    url: `http://ttapi.research.itcast.cn/mp/v1_0/captchas/${mobile}`
  }).then(res => {
    const { data } = res.data
    console.log(data)
    window.initGeetest({
            // 以下配置参数来自服务端 SDK
            gt: data.gt,
            challenge: data.challenge,
            offline: !data.success,
            new_captcha: data.new_captcha,
             product: 'bind' 
        }, (captchaObj) => {
            // 这里可以调用验证实例 captchaObj 的实例方法
            this.captchaObj = captchaObj
             captchaObj.onReady(function () {
        // 验证码ready之后才能调用verify方法显示验证码
             captchaObj.verify()
      }).onSuccess(function () {
        // 人机交互验证通过
			 const {
          geetest_challenge: challenge,
          geetest_seccode: seccode,
          geetest_validate: validate } =
              captchaObj.getValidate()
               axios({
          method: 'GET',
          url: `http://ttapi.research.itcast.cn/mp/v1_0/sms/codes/${mobile}`,
          params: {
            challenge,
            validate,
            seccode
          }
        }).then(res => {
          console.log(res.data)
          // 开启倒计时效果
        })
      }).onError(function () {
        // your code
      })
        })
  })
}
    }
    }
</script>
<style lang="less" scoped>
.login-wrap {
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    background: #ccc;
    .login-head{
       display: flex;
       justify-content: center;
       padding:10px; 
    }
    .login-form-wrap{
        background: #fff;
        padding: 50px;
        .btn-login{
            width: 100%;
        }
    }
}
</style>
