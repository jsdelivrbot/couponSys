<template>
    <el-dialog title="修改密码" :visible.sync="addFormVisible" :close-on-click-modal="false" width="500px">
        <el-form   :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
            <el-form-item label="旧密码" prop="oldpassword">
                <el-input type="password" v-model="ruleForm.oldpassword"></el-input>
            </el-form-item>   
            <el-form-item label="新密码" prop="newpassword1">
                <el-input type="password" v-model="ruleForm.newpassword1"></el-input>
            </el-form-item>
            <el-form-item label="确认密码" prop="newpassword2">
                <el-input type="password" v-model="ruleForm.newpassword2"></el-input>
            </el-form-item>
        </el-form> 
        <div slot="footer" class="dialog-footer">
            <el-button @click="addFormVisible = false">取消</el-button>
            <el-button type="primary" @click="handleSubmit" >保存</el-button>
        </div>
  </el-dialog>
</template>

<script>
    export default {
        data(){
            return{
                addFormVisible:false,
                ruleForm: {
                    oldpassword: '',
                    newpassword1: '',
                    newpassword2: ''
                },
                rules: {
                    oldpassword: [
                        { required: true, message: '请输入旧密码', trigger: 'blur' },
                    ],
                    newpassword1: [
                        { required: true, message: '请输入新密码', trigger: 'blur' },
                    ],
                    newpassword2: [
                        { required: true, message: '请输入确认新密码', trigger: 'blur' },
                    ]
                },
            }
        },
        methods:{
            handleSubmit() { 
                this.$refs.ruleForm.validate(async (valid) => {
                    if (valid) {
                        if(this.ruleForm.newpassword1!=this.ruleForm.newpassword2){
                            this.$message.error('两次输入密码不一致');
                            return
                        }
                        let d={
                            oldPwd:this.ruleForm.oldpassword,
                            newPwd:this.ruleForm.newpassword1     
                        };
                        let res = await this.$post('/couponSys/mgrUser/changePassword.json',d);
                        if(res.errcode==0){
                            this.ruleForm.oldpassword='';
                            this.ruleForm.newpassword1='';
                            this.ruleForm.newpassword2=''
                            this.$message.success('修改成功');
                            this.addFormVisible=false
                        }else{
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

<style>

</style>
