<template>
    <el-row class="container" >
        <el-col :span="24" class="header" >
            <el-col :span="10" class="logo" :class="isCollapse?'logo-collapse-width':'logo-width'">
            </el-col>
            <el-col :span="10" class="title-box">
				<div class="tools" @click="isCollapse=!isCollapse">
					<i class="el-icon-minus"></i>
                    <i class="el-icon-minus"></i>
                    <i class="el-icon-minus"></i>
				</div>
                <div class="title">盈众乐橙营销管理系统</div>
			</el-col>
            <el-col :span="4" class="userinfo">
				<el-dropdown trigger="hover">
					<span class="userinfo-inner">{{sysUserName}}</span>
                    <i class="el-icon-arrow-down"></i>
					<el-dropdown-menu slot="dropdown">
						<el-dropdown-item @click.native="modify">修改密码</el-dropdown-item>
						<el-dropdown-item @click.native="logout">注销</el-dropdown-item>
					</el-dropdown-menu>
				</el-dropdown>
			</el-col>
        </el-col>
        <el-col :span="24" class="main">
            <aside class="menu-expanded">
                <!--导航菜单-->
                <el-menu  class="el-menu-vertical-demo" :unique-opened="true" background-color="#324157" text-color="#bfcbd9" :default-active="$route.path" @open="handleOpen" @close="handleClose" :collapse="isCollapse" router>
                    <template v-for="(item,index) in menuList" >
                        <el-submenu  :index="index+''" :key="index" >
                            <template slot="title">
                                <i v-for="(val,inx) in $router.options.routes" v-if="val.meta.name==item.name" :class="val.meta.icon"></i>
                                <span slot="title">{{item.name}}</span>
                            </template>
                            
                            <el-menu-item  v-for="(val,inx) in item.child" :index="val.url" :key="inx">
                                ●  {{val.name}}
                            </el-menu-item>
                            
                        </el-submenu>
                    </template>
                    
                </el-menu>
            </aside>
            <section class="content-container">
                
                <el-col :span="24" class="content-wrapper">
                    <transition name="fade" mode="out-in">
                       <!--  <keep-alive include="resourcelist,userlist,rolelist,verifylist,underlinelist,grantbox,couponbox,couponlist,lechenglist,platformlist,marketinglist,underlinelist,grantlist">
                            <router-view></router-view>
                        </keep-alive> -->
                        <keep-alive :include="/^in-/">
                            <router-view></router-view>
                        </keep-alive>
                    </transition>
                </el-col>
            </section>
        </el-col>
        <modifypassword ref="modify"></modifypassword>
  </el-row>
  
</template>
<script>
    import modifypassword from './Modifypassword.vue'
    
    export default {
        data(){
            return{
                isCollapse:false,
                sysUserName:'',
                menuList:[]
            }
        },
        mounted(){
            
            let userinfo = unescape(sessionStorage.getItem('userinfo'));
			if (userinfo) {
				userinfo = JSON.parse(userinfo);
				this.sysUserName = userinfo.name || '';
            }
            
            this.getMenu();
            eventBus.$on('resetMenu',(msg)=> {
                this.getMenu();
            })
        },
        components:{
            modifypassword
        },
        methods:{
            
            async getMenu(){
                let res = await this.$get('/couponSys/mgrFunction/menu.json');
                this.menuList=res
            },
            // 注销登录
            logout(){
                this.$confirm('确认退出吗?', '提示', {
                    //type: 'warning'
                }).then(async() => {
                    let res = await this.$get('/couponSys/logout');

                    sessionStorage.removeItem('user');
                    this.$router.push('/login');
                }).catch(() => {

                });
            },
            //修改密码
            modify(){
                this.$refs.modify.addFormVisible=true
            },
            handleOpen(){

            },
            handleClose(){

            }
        }
    }
</script>
<style lang="scss" scoped>
    @import '../assets/css/vars.scss';
    
    a{
        text-decoration: none;
        color: #48576a;
    }
    .container {
        position: absolute;
        top: 0px;
        bottom: 0px;
        width: 100%;
        min-width: 1100px;
        .header {
            height: 60px;
            line-height: 60px;
            background: $mainColor;
            color:#fff;
            .logo{
                height: 60px;
                font-size: 22px;
                padding-left: 20px;
                padding-right: 20px;
                border-right: 1px solid rgba(255,255,255,.5);
                // background: #a1887f;
                // background: #fff;  
                background-position: center;
                background-repeat: no-repeat;
                transition: width .3s ease-in-out;
                &.logo-collapse-width{
                    background-image: url('../assets/img/favicon.png');
                    width: 65px;
                }
                &.logo-width{
                    background-image: url('../assets/img/lecheng_logo1.png');
                    width: 230px;
                }
                img{
                    height: 100%;
                }
            }
            .title-box{
                .tools{
                    display: inline-block;
                    position: relative;
                    top: 5px;
                    padding: 0 20px;
                    cursor: pointer;
                    i{
                        display: block;
                        margin-top: -10px;
                    }
                }
                .title{
                    display: inline-block;
                }
            }
            .userinfo{
                text-align: right;
                padding-right: 35px;
                float: right;
                .el-dropdown{
                    color: #fff
                }
            }
        }
    }
    .main{
        margin-bottom: 0;
        position: absolute;
        top: 60px;
        bottom: 0;
        overflow: hidden;
        background: #f0f0f0;
        .menu-expanded{
            float: left;
            height: 100%;
            .el-menu::-webkit-scrollbar{
                width: 0px;
            }
            .el-menu{
                height: 100%;
                overflow: auto;
            }
            .el-menu--collapse{
                overflow: visible;
            }
            .el-menu-vertical-demo:not(.el-menu--collapse) {
                width: 230px;

            }
        }
        .content-container{
            box-sizing: border-box;
            height: 100%;
            overflow-y: auto;
            padding: 20px;
            .content-wrapper{
                box-sizing: border-box;
            }
        }
        
    }
</style>
