<style lang="less">
    @import './login.less';
</style>

<template>
    <div class="login" @keydown.enter="handleSubmit">
        <div class="login-con">
            <Card :bordered="false">
                <p slot="title">
                    <Icon type="log-in"></Icon>
                    欢迎登录
                </p>
                <div class="form-con">
                    <Form ref="loginForm" :model="form" :rules="rules">
                        <FormItem prop="userName">
                            <Input v-model="form.userName" placeholder="请输入用户名">
                                <span slot="prepend">
                                    <Icon :size="16" type="person"></Icon>
                                </span>
                            </Input>
                        </FormItem>
                        <FormItem prop="password">
                            <Input type="password" v-model="form.password" placeholder="请输入密码">
                                <span slot="prepend">
                                    <Icon :size="14" type="locked"></Icon>
                                </span>
                            </Input>
                        </FormItem>
                         <FormItem prop="identifyCode">
                                <Input  v-model="identifyCodeInput" placeholder="验证码">
                                    <!-- <div  slot="prepend">验证码</div> -->
                               
                                <div @click="refreshCode" slot="append"  >
                                        <s-identify :identifyCode="identifyCode"></s-identify>
                                </div>
                             </Input>
                        </FormItem>
                        <div style="color:red">{{errorMessage}}</div>
                        <FormItem >
                            <Button @click="handleSubmit" :loading="logining" type="primary" long>登录</Button>
                        </FormItem>
                    </Form>
                    <p class="login-tip"></p>
                </div>
            </Card>

        </div>
    </div>
</template>

<script>
import Cookies from 'js-cookie';
import SIdentify from './identifyCode/identify.vue'
export default {
    components:{
        SIdentify
    },
    data () {
     
        return {
             identifyCodes: "1234567890",
            identifyCode: "1234",
            identifyCodeInput:'',
            form: {
                userName: '',
                password: ''
            },
            rules: {
                userName: [
                    { required: true, message: '账号不能为空', trigger: 'blur' }
                ],
                password: [
                    { required: true, message: '密码不能为空', trigger: 'blur' }
                ]
            },
            logining: false,
            errorMessage: ''
        };
    },
   mounted() {
        this.identifyCode = "";
        this.makeCode(this.identifyCodes, 4);
    },
    methods: {
        handleSubmit () {
              this.errorMessage="";
                if(this.identifyCodeInput!=this.identifyCode){
                     this.errorMessage="请输入正确的验证码";
                    return;
                }
                this.refreshCode();
            this.$refs.loginForm.validate((valid) => {
                if (valid) {
                    this.logining=true;
                    this.$http.post('/user/login', {
                        "account": this.form.userName ,
                        "password": this.form.password
                    })
                    .then(response=> {
                      //如果接口走成功就执行这里
                      // console.log(JSON.stringify(response));
                      var data = response.data;
                      if(data.code == 0) {
                        this.loginSuccess();
                      }else {
                        this.errorMessage=data.message;
                      }
                      this.logining=false;

                    }).catch(error=> {
                        this.errorMessage="网络连接错误";
                        this.logining=false;
                    });
                }
            });
        },
        loginSuccess(){
            Cookies.set('user', this.form.userName);
            Cookies.set('account',this.form.userName);
            Cookies.set('password', this.form.password);
            
            this.$store.commit('setAvator', 'https://ss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=3448484253,3685836170&fm=27&gp=0.jpg');
            Cookies.set('access', 0);
            // if (this.form.userName === 'iview_admin') {
            //     Cookies.set('access', 0);
            // } else {
            //     Cookies.set('access', 1);
            // }
            this.$router.push({
                name: 'home_index'
            });
        },
         randomNum(min, max) {
            return Math.floor(Math.random() * (max - min) + min);
        },
        refreshCode() {
          this.identifyCode = "";
          this.makeCode(this.identifyCodes, 4);
        },
        makeCode(o, l) {
          for (let i = 0; i < l; i++) {
            this.identifyCode += this.identifyCodes[
              this.randomNum(0, this.identifyCodes.length)
            ];
          }
          // console.log(this.identifyCode);
        }
    }
};
</script>

<style>

</style>
