<template>
  <section>
    <el-row class="breadcrumb">角色管理</el-row>
    <!--工具条-->
    <el-row  class="toolbar" >
      <el-col class="search-box" v-if="showAdd">
        <el-button class="el-icon-plus button-primary-hover" @click="handleAdd"  > 创建新角色</el-button>
      </el-col>
      <el-col class="search-item">
        <div class="search-box">
          <span class="search-title">角色名称</span>
          <el-input v-model="search.name" placeholder="请输入角色名称" style="width:200px" :clearable="true"></el-input>
        </div>
        <div class="search-box">
            <el-button  type="primary" @click="searchItem" class="el-icon-search" > 搜索</el-button>
          </div>
      </el-col>
        
    </el-row>
      <!--列表-->
    <el-table :data="listItems" stripe  highlight-current-row v-loading="listLoading"  style="width: 100%">
      <!-- <el-table-column type="selection" ></el-table-column> -->
      <el-table-column prop="name" label="角色名称"  ></el-table-column>
      <el-table-column prop="code" label="角色编码"   ></el-table-column>
      <el-table-column  label="角色状态"  >
        <template scope="scope">
          {{scope.row.vstatus==1 ? '未启用' : scope.row.vstatus==2 ? '已启用':'已停用'}}
        </template>
      </el-table-column>

      <el-table-column label="操作" width="120" align="center">
        <template scope="scope">
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
    <!-- 新增弹窗页面 -->
    <addrole ref="add" :showType="showType" :childValue="childValue" @childMethods="childMethods"></addrole>
  </section>
</template>
<script>
  import addrole from './Addrole.vue'
  export default {
    name:'in-rolelist',
    data(){
      return{
        showAdd:false,
        showDelete:false,
        showUpdate:false,
        pageNo:1,
        totalNum:0,
        pageSize:20,
        listItems:[],
        search:{
          name:''
        },
        listLoading: false,
        showType:1,  //显示弹窗类型  1：添加  2：修改
        childValue:''
      } 
    },
    components:{
      addrole
    },
    mounted(){
      this.getData()
      
    },
    methods:{
      childMethods(msg){
        if(msg==1){
          this.search.name=''
        }
        this.getData();
      },
      async getData(){
        this.listLoading=true
        let d={
            page:this.pageNo,
            pageSize:this.pageSize,
            search_LIKE_name: this.search.name
        };
        let res = await this.$get('/couponSys/mgrRole/page.json',d);
        this.listLoading=false
        this.listItems=res.content;
        this.totalNum=res.totalNum;
        if(res.functions&&res.functions.add){
          this.showAdd=true
        }
        if(res.functions&&res.functions.delete){
          this.showDelete=true
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
        let res = await this.$get('/couponSys/mgrRole/toUpdate.json',d);
        if(res.errcode==0){
          this.$refs.add.addFormVisible=true;
          this.$refs.add.$nextTick(()=>{
            this.$refs.add.resetForm()
            this.showType=2;
            let item={
              name: res.value.name,
              code: res.value.code,
              vstatus: res.value.vstatus.toString(),
            }
            this.$refs.add.ruleForm=item;

            let arr=[];
            for(let i=0;i<res.value.functions.length;i++){
              arr.push(res.value.functions[i].id)
            }
            this.$refs.add.$refs.tree.setCheckedKeys(arr);
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
            let res = await this.$delete('/couponSys/mgrRole/delete.json',d);
            if(res){
              this.getData();
              this.$message.success('删除成功');
            }
        })
        
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
            code: '',
            vstatus:''
          }
          this.$refs.add.ruleForm=item;
          this.$refs.add.$refs.tree.setCheckedKeys([]);
        })
      }
    }

  }
</script>
<style lang="scss">

</style>
