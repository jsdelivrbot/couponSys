<template>
  <el-dialog :title="showType==1 ? '新增':'编辑'"  :visible.sync="addFormVisible" :close-on-click-modal="false">
    <el-form :model="ruleForm1" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
      <el-form-item label="上传图片" prop="showUrl">
        <el-upload class="avatar-uploader" 
          :action="postUrl" 
          name="files"
          :show-file-list="false" 
          :on-success="handleAvatarSuccess"
          :disabled="showType==2&&!showUpdate?true:false">
          <img v-if="ruleForm1.showUrl" :src="ruleForm1.showUrl" class="avatar">
          <i v-else class="el-icon-plus avatar-uploader-icon"></i>
        </el-upload>
      </el-form-item>
      <el-form-item class="item-two">
        <el-col :span="12">
          <el-form-item label="账号" prop="account">
            <el-input v-model="ruleForm1.account" :disabled="showType==2?true:false"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="姓名" prop="name">
            <el-input v-model="ruleForm1.name" :disabled="showType==2&&!showUpdate?true:false"></el-input>
          </el-form-item>
        </el-col>
      </el-form-item>

      <el-form-item class="item-two">
        <el-col :span="12">
          <el-form-item label="电话" prop="mobilePhone">
            <el-input v-model="ruleForm1.mobilePhone" :disabled="showType==2&&!showUpdate?true:false"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="岗位" prop="station">
            <el-input v-model="ruleForm1.station" :disabled="showType==2&&!showUpdate?true:false"></el-input>
          </el-form-item>
        </el-col>
        
      </el-form-item>
      <el-form-item class="item-two">
        <el-col >
          <el-form-item label="4s店" prop="fourvalue">
            <el-select style="width:100%" v-model="ruleForm1.fourvalue" multiple placeholder="请选择" :disabled="showType==2&&!showUpdate?true:false">
              <el-option v-for="(item,index) in fourshops"  :label="item.NAME" :value="item.PK_FOURSPOINTMG+''">
              </el-option>
            </el-select>
          </el-form-item>
        </el-col>
        
      </el-form-item>
      <el-form-item class="item-two">
        
        <el-col>
          <el-form-item label="角色明细" prop="rolevalue">
            <el-select style="width:100%" v-model="ruleForm1.rolevalue" multiple placeholder="请选择" :disabled="showType==2&&!showUpdate?true:false">
              <el-option v-for="item in roleList" :key="item.id" :label="item.name" :value="item.id">
              </el-option>
            </el-select>
          </el-form-item>
        </el-col>
      </el-form-item>

    </el-form>


    <div slot="footer" class="dialog-footer">
      <el-button @click="addFormVisible = false">取消</el-button>
      <el-button type="primary" @click="submitForm('ruleForm')">保存</el-button>
    </div>
  </el-dialog>
</template>
<script>
  export default {
    props:['showType','showUpdate','childValue'],
    data() {
      return {
        addFormVisible: false,
        postUrl:host+'/couponSys/fileUtil/uploadFile.json',
        ruleForm1: {
          name: '',
          account: '',
          mobilePhone: '',
          station:'',
          rolevalue:'',
          fourvalue:'',
          imgUrl:'',
          showUrl:''
        },
        rules: {
          account: [
            {required: true,message: '请输入账号',trigger: 'blur'},
          ],
          name: [
            {required: true,message: '请输入姓名',trigger: 'blur'},
          ],
          mobilePhone: [
            {required: true,message: '请输入电话',trigger: 'blur'},
          ],
          // fourvalue: [
          //   {required: true,message: '请选择4s店',trigger: 'change'},
          // ],
        },
        limit: [{
          id: 1,
          label: '一级 2',
          children: [{
            id: 3,
            label: '二级 2-1',
          }, {
            id: 2,
            label: '二级 2-2',
          }]
        }],
        roleList:[], //角色列表
        
        fourshops:[]
      }

    },
    activated(){
      this.getroleList()
      this.getFours()
    },
    methods: {
      async getFours() {
        let res = await this.$get('/couponSys/common/fourshopsList.json');
        if (res.errcode == 0) {
          this.fourshops = res.fourshops
        }
      },
      async getroleList(){
        let res = await this.$get('/couponSys/mgrRole/roleList.json');
        this.roleList=res;
      },
      handleAvatarSuccess(res, file) {
        
        if(res.errcode==0){
          this.ruleForm1.showUrl=res.showUrl
          this.ruleForm1.imgUrl = res.resultUrl;
        }
      },
      
      submitForm(formName) {
        this.$refs[formName].validate(async (valid) => {
          if (valid) {
            
            if(this.showType==1){
              this.addForm()
            }else{
              this.modifyForm()
            }
          } 
        });
      },
      async addForm(){
        let roles=[];
        for(let i=0;i<this.ruleForm1.rolevalue.length;i++){
          let item={
            id:this.ruleForm1.rolevalue[i]
          }
          roles.push(item)
        }

        let d={
            id:'',
            account:this.ruleForm1.account,
            name:this.ruleForm1.name,
            foursShop:this.ruleForm1.fourvalue.join(','),
            station:this.ruleForm1.station,
            mobilePhone:this.ruleForm1.mobilePhone,
            imgUrl:this.ruleForm1.imgUrl,
            roles:roles,
            
          };
          let res = await this.$post('/couponSys/mgrUser/add.json',d);
          if(res.errcode==0){
            this.$emit('childMethods',1)
            this.$message.success('添加成功');
            this.addFormVisible=false;
          }else{
            this.$message.error(res.msg);
          }
      },
      async modifyForm(){
        let roles=[];
        for(let i=0;i<this.ruleForm1.rolevalue.length;i++){
          let item={
            id:this.ruleForm1.rolevalue[i]
          }
          roles.push(item)
        }

        let d={
            id:this.childValue.id,
            account:this.ruleForm1.account,
            name:this.ruleForm1.name,
            foursShop:this.ruleForm1.fourvalue.join(','),
            station:this.ruleForm1.station,
            mobilePhone:this.ruleForm1.mobilePhone,
            imgUrl:this.ruleForm1.imgUrl,
            roles:roles,
          };
          let res = await this.$post('/couponSys/mgrUser/update.json',d);
          if(res.errcode==0){
            this.$emit('childMethods',2)
            this.$message.success('修改成功');
            this.addFormVisible=false;
          }else{
            this.$message.error(res.msg);
          }
      },
      resetForm() {
        this.$refs.ruleForm.resetFields();
      },
    }
  }

</script>

<style lang="scss" scoped>
  

</style>
