<template>
    <div>
        <div class="logo-header">
            <img src="../assets/img/icon_logo.png" alt="" class="logo">
            <img src="../assets/img/icon_phone.png" alt="" class="fr contact">
        </div>
        <div class="content-box">
            <el-form :model="loginForm" :rules="loginRules" ref="loginForm" label-position="left" label-width="0px" class="login-container" v-if="!showmodify">
                <!-- <div class="title">登录</div> -->
                <img src="../assets/img/lecheng_logo.png">
                <el-form-item prop="username">
                    <el-input type="text" v-model="loginForm.username" auto-complete="off" placeholder="账号" prefix-icon="  " @keyup.enter.native="loginSubmit" autofocus></el-input>
                    <img src="../assets/img/icon_weblogin_id.png" class="login-icon">
                </el-form-item>
                <el-form-item prop="password">
                    <el-input type="password" v-model="loginForm.password" auto-complete="off" placeholder="密码" prefix-icon="  "  @keyup.enter.native="loginSubmit"></el-input>
                    <img src="../assets/img/icon_weblogin_password.png" class="login-icon">
                </el-form-item>
                <el-checkbox fill="#FB841C"  v-model="checked" checked class="remember">记住密码</el-checkbox>      
                <el-form-item style="width:100%;">
                    <el-button type="primary" class="login-btn"  style="width:100%;" @click="loginSubmit" :loading="logining">登 录</el-button>
                </el-form-item>
            </el-form>

            <el-form :model="modifyForm" :rules="modifyRules" ref="modifyForm" label-position="left" label-width="0px" class="login-container" v-else>
                <div class="title">请您修改密码</div>
                <el-form-item prop="newPwd1">
                    <el-input type="password" v-model="modifyForm.newPwd1" auto-complete="off" placeholder="请输入新密码" prefix-icon="  "></el-input>
                    <img src="../assets/img/icon_weblogin_password.png" class="login-icon">
                </el-form-item>
                <el-form-item prop="newPwd2">
                    <el-input type="password" v-model="modifyForm.newPwd2" auto-complete="off" placeholder="请输入确认新密码" prefix-icon="  "></el-input>
                    <img src="../assets/img/icon_weblogin_password.png" class="login-icon">
                </el-form-item>
                <el-form-item style="width:100%;">
                    <el-button class="login-btn"  style="width:100%;" @click="modifySubmit" :loading="logining">确 定</el-button>
                </el-form-item>
                <el-form-item style="width:100%;">
                    <el-button  type="warning" plain style="width:100%;" @click="showmodify=false">取 消</el-button>
                </el-form-item>
            </el-form>
        </div>
    </div>
</template>

<script>

    export default {
        data() {
            return {
                showmodify:false,
                logining: false,
                loginForm: {
                    username: '',
                    password: ''
                },
                loginRules: {
                    username: [
                        { required: true, message: '请输入账号', trigger: 'blur' },
                    ],
                    password: [
                        { required: true, message: '请输入密码', trigger: 'blur' },
                    ]
                },
                checked: true,
                modifyForm: {
                    newPwd1: '',
                    newPwd2: ''
                },
                modifyRules: {
                    newPwd1: [
                        { required: true, message: '请输入新密码', trigger: 'blur' },
                    ],
                    newPwd2: [
                        { required: true, message: '请输入确认新密码', trigger: 'blur' },
                    ]
                }
                
            };
        },
        mounted(){
            let user=JSON.parse(unescape(localStorage.getItem('user')))
            if(user){
                this.checked=true
                this.loginForm.username=user.username;
                this.loginForm.password=user.password
                
            }else{
                this.checked=false
            }
        },
        methods:{
            loginSubmit() { 

                this.$refs.loginForm.validate(async (valid) => {
                    if (valid) {
                        this.logining = true;
                        let d={
                            username:this.loginForm.username,
                            password:this.loginForm.password     
                        };
                        let res = await this.$post('/couponSys/login/login.json',d);
                        if(res.errcode==0){
                            if(this.checked){
                                localStorage.setItem('user', escape(JSON.stringify(this.loginForm)));
                            }else{
                                localStorage.removeItem('user');
                            }
                            sessionStorage.setItem('user', escape(JSON.stringify(this.loginForm)));
                            
                            if(res.result=='password unmodified'){
                                this.logining = false;
                                this.showmodify=true;
                            }else{
                                sessionStorage.setItem('userinfo', escape(JSON.stringify(res.userInfo)));
                                this.$router.push({ path: '/home' });
                            }
                            
                        }else{
                            this.logining = false;
                            this.$message.error(res.msg);
                        }
                        
                    } else {
                        console.log('error submit!!');
                        return false;
                    }
                });
            },
            modifySubmit(){
                this.$refs.modifyForm.validate(async (valid) => {
                    if (valid) {
                        if(this.modifyForm.newPwd1!=this.modifyForm.newPwd2){
                            this.$message.error('两次输入密码不一致');
                            return
                        }
                        this.logining = true;
                        let d={
                            username:this.loginForm.username,
                            newPwd:this.modifyForm.newPwd1     
                        };
                        let res = await this.$post('/couponSys/login/changePwd.json',d);
                        if(res.errcode==0){
                            this.loginForm.password=this.modifyForm.newPwd1;
                            if(this.checked){
                                localStorage.setItem('user', escape(JSON.stringify(this.loginForm)));
                            }else{
                                localStorage.removeItem('user');
                            }
                            sessionStorage.setItem('user', escape(JSON.stringify(this.loginForm)));
                            sessionStorage.setItem('userinfo', escape(JSON.stringify(res.userInfo)));
                            this.$router.push({ path: '/home' });

                            
                                
                                
                            
                        }else{
                            this.logining = false;
                            this.$message.error(res.msg);
                        }
                        
                    } else {
                        console.log('error submit!!');
                        return false;
                    }
                });
            }
        }
    }
</script>

<style lang="scss" scoped>
    .login-container {
        border-radius: 20px;
        background-clip: padding-box;
        position: absolute;
        top: 45%;
        right: 13%;
        transform: translateY(-50%);
        -o-transform: translateY(-50%);
        -moz-transform: translateY(-50%);
        -ms-transform: translateY(-50%);
        width: 280px;
        padding: 25px 35px 15px 35px;
        background: #fff;
        box-shadow: 0 0 15px -2px #000;
        .login-icon{
            width: 18px;
            top: 10px;
            left: 7px;
            margin: 0;
            position:absolute
        }
        .title {
            font-size: 20px;
            margin: 0px auto 40px auto;
            text-align: center;
            color: #505458;
        }
        .remember {
            margin: 0px 0px 35px 0px;
            
        }
        .login-btn{
            color:#fff;
            background: #eb6100;
            border-color: #eb6100;
        }
        
        img{
            margin: 0 25% 25px;
            width: 50%;
        }
    }
    .el-checkbox__input.is-checked .el-checkbox__inner{
        background-color: #FB841C;
        border-color: #FB841C ;
    }
    .content-box{
        height: calc(100% - 80px);
        width: 100%;
        position: absolute;
        background: url('../assets/img/bg_login.jpg') no-repeat;
            background-size: cover;
        background-position: center;
    }
    .logo-header{
        height: 80px;
        width: 100%;
        .logo{
            height: 40px;
            margin-top: 20px;
            float: left;
            margin-left: 5%;
        }

        .contact{
            height:30px;
            margin:25px 5% 0 0
        }
    }
    
</style>
