<template>
  <section>
    <el-row class="breadcrumb">用户管理</el-row>
    <!--工具条-->
    <el-row class="toolbar">
      <el-col class="search-box" v-if="showCreate">
        <el-button class="el-icon-plus button-primary-hover" @click="handleAdd"  > 创建新用户</el-button>
      </el-col>
      <el-col class="search-item">
        <div class="search-box">
          <span class="search-title">账号</span>
          <el-input v-model="search.account" placeholder="请输入账号" style="width:200px" :clearable="true"></el-input>
        </div>
        <div class="search-box">
          <span class="search-title">用户名</span>
          <el-input v-model="search.name" placeholder="请输入用户名" style="width:200px" :clearable="true"></el-input>
        </div>
        
        <div class="search-box">
            <el-button  type="primary" @click="searchItem" class="el-icon-search" > 搜索</el-button>
          </div>
      </el-col>
      
    </el-row>
      <!--列表-->
    <el-table :data="listItems" stripe  highlight-current-row v-loading="listLoading"  style="width: 100%">
      <!-- <el-table-column type="selection" ></el-table-column> -->
      <el-table-column prop="account" label="账号"  width="100"></el-table-column>
      <el-table-column prop="name" label="姓名"   width="100"></el-table-column>
      <el-table-column  label="注册时间" >
        <template scope="scope">
          {{$format(scope.row.createTime)}}
        </template>
      </el-table-column>
      <el-table-column prop="roleNames" label="角色"  ></el-table-column>
      <el-table-column prop="mobilePhone" label="电话"  ></el-table-column>
      <el-table-column label="状态"  >
        <template scope="scope">
          {{scope.row.vstatus==1 ? '未启用' : scope.row.vstatus==2 ? '已开通':'已停用'}}
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" width="210">
        <template scope="scope">
          <el-tooltip content="停用" placement="top" v-if="scope.row.vstatus==2">
            <el-button class="fa fa-stop-circle-o " aria-hidden="true" type="primary" size="mini"   @click="stopVstatus(scope.row.id,scope.$index)" :disabled="showStop ? false:true" ></el-button>
          </el-tooltip>
          <el-tooltip content="开启" placement="top" v-else>
            <el-button class="fa fa-play-circle-o" type="primary" size="mini"  @click="startVstatus(scope.row.id,scope.$index)" :disabled="showStart ? false:true" ></el-button>
          </el-tooltip>
          <el-tooltip content="重置密码" placement="top">
            <el-button type="primary" class="fa fa-key"  size="mini" @click="modifyPwd(scope.row.id)" :disabled="showUpdate ? false:true"></el-button>
          </el-tooltip>
          <el-tooltip content="编辑" placement="top">
            <el-button class="el-icon-edit" type="primary"  size="mini" @click="modifyItem(scope.row.id)" :disabled="showUpdate ? false:true"></el-button>
          </el-tooltip>
          <el-tooltip content="删除" placement="top">
            <el-button class="el-icon-delete" type="primary" size="mini" @click="removeItem(scope.row.id)" :disabled="showDelete ? false:true"></el-button>
          </el-tooltip>
        </template>
      </el-table-column>
    </el-table>
    <!--工具条-->
    <el-col :span="24" class="toolbar">
      <!-- <el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button> -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :page-sizes="[10, 20, 30, 40,50]"
        :page-size="pageSize"
        layout="sizes, prev, pager, next"
        :total="totalNum"
        style="float:right;">
      </el-pagination>
    </el-col>
    <!-- 强制修改密码页面 -->
    <resetuserpwd  ref="pwd" ></resetuserpwd>
    <!-- 新增弹窗页面 -->
    <adduser  ref="add"  :showType="showType" :showUpdate="showUpdate" :childValue="childValue" @childMethods="childMethods"></adduser>
  </section>
</template>
<script>
  import resetuserpwd from './Resetuserpwd.vue'
  import adduser from './Adduser.vue'
  export default {
    name:'in-userlist',
    data(){
      return{
        select:'2',
        showCreate:false,
        showDelete:false,
        showStart:false,
        showStop:false,
        showUpdate:false,
        pageNo:1,
        totalNum:0,
        pageSize:20,
        listItems:[],
        search:{
          account:'',
          name:''
        },
        listLoading: false,
        showType:1,  //显示弹窗类型  1：添加  2：修改
        childValue:'',
      } 
    },
    components:{
      adduser,
      resetuserpwd
    },
    mounted(){
      this.getData()
    },
    methods:{
      childMethods(msg){
        if(msg==1){
          this.search.name='',
          this.search.account=''
        }
        this.getData();
      },
      async getData(){
        this.listLoading=true
        let d={
            page:this.pageNo,
            pageSize:this.pageSize,
            search_LIKE_name: this.search.name,
            search_LIKE_account:this.search.account,
        };
        let res = await this.$get('/couponSys/mgrUser/page.json',d);
        this.listLoading=false
        this.listItems=res.content;
        this.totalNum=res.totalNum;
        if(res.functions&&res.functions.create){
          this.showCreate=true
        } 
        if(res.functions&&res.functions.delete){
          this.showDelete=true
        }
        
        if(res.functions&&res.functions.start){
          this.showStart=true
        }
        if(res.functions&&res.functions.stop){
          this.showStop=true
        }
        if(res.functions&&res.functions.update){
          this.showUpdate=true
        } 
      },
      //点击搜索
      async searchItem(){
        this.pageNo=1;
        this.getData()
      },
      //点击修改操作
      async modifyItem(id){
        let d={
            id:id
        };
        let res = await this.$get('/couponSys/mgrUser/toUpdate.json',d);
        if(res.errcode==0){
          this.$refs.add.addFormVisible=true;
          this.$refs.add.$nextTick(()=>{
            this.$refs.add.resetForm()
            this.showType=2;
            let roles=[];
            for(let i=0;i<res.value.roles.length;i++){
              roles.push(res.value.roles[i].id)
            }
            let item={
              name: res.value.name,
              account: res.value.account,
              mobilePhone:res.value.mobilePhone,
              station: res.value.station,
              fourvalue: res.value.foursShop?res.value.foursShop.split(','):[],
              rolevalue:roles,
              showUrl:res.value.showImgUrl,
              imgUrl:res.value.imgUrl
            }
            this.$refs.add.ruleForm1=item;
            this.childValue=res.value;
          })
          
        }
      },
      //点击删除
      removeItem(id){
        this.$confirm('确认删除吗?', '提示', {
            //type: 'warning'
        }).then(async () => {
            let d={
                id:id
            };
            let res = await this.$delete('/couponSys/mgrUser/delete.json',d);
            if(res.errcode==0){
              this.getData();
              this.$message.success('删除成功');
            }
        })
        
      },
      //点击停用
      async stopVstatus(id,index){
        let d={
            id:id
        };
        let res = await this.$get('/couponSys/mgrUser/stop.json',d);
        if(res.errcode==0){
          this.listItems[index].vstatus=3
        }
      },
      //点击开启
      async startVstatus(id,index){
        let d={
            id:id
        };
        let res = await this.$get('/couponSys/mgrUser/start.json',d);
        if(res.errcode==0){
          this.listItems[index].vstatus=2
        }
      },
      //点击重置密码
      modifyPwd(id){
        this.$refs.pwd.id=id
        this.$refs.pwd.addFormVisible=true
      },
      //pageSize 改变时会触发
      handleSizeChange(val){
        this.pageNo=1;
        this.pageSize=val;
        this.getData()
      },
      //currentPage 改变时会触发
      handleCurrentChange(val){
        this.pageNo=val;
        this.getData()
      },
      
      //点击新增
      handleAdd(){ 
        this.$refs.add.addFormVisible=true
        this.$refs.add.$nextTick(()=>{
          this.$refs.add.resetForm()
          this.showType=1;
          let item={
            name: '',
            account: '',
            mobilePhone:'',
            station: '',
            foursShop: '',
            rolevalue:[],
            showUrl:'',
            imgUrl:''
          }
          this.$refs.add.ruleForm=item;
        })
      },
    }

  }
</script>
<style lang="scss" >

  
  
</style>
