<template>
  <section>
    <el-row class="breadcrumb">后台参数配置</el-row>
    <!--工具条-->
    <el-row class="toolbar">
      <el-col class="search-box" v-if="showCreate">
        <el-button class="el-icon-plus button-primary-hover" @click="handleAdd"  > 创建参数配置</el-button>
      </el-col>
      <el-col class="search-item">
        <div class="search-box">
          <span class="search-title">键</span>
          <el-input v-model="search.key" placeholder="请输入键" style="width:200px" :clearable="true"></el-input>
        </div>
        <div class="search-box">
          <span class="search-title">键名称</span>
          <el-input v-model="search.name" placeholder="请输入键名称" style="width:200px" :clearable="true"></el-input>
        </div>
        <div class="search-box">
            <span class="search-title">类型</span>
            <el-select v-model="search.type"  placeholder="请选择" style="width:200px" :clearable="true">
              <el-option label="回调地址" :value="1"></el-option>
              <el-option label="接口开关" :value="2"></el-option>
            </el-select>
        </div>
        <div class="search-box">
            <el-button  type="primary" @click="searchItem" class="el-icon-search" > 搜索</el-button>
          </div>
      </el-col>
      
    </el-row>
      <!--列表-->
    <el-table :data="listItems" stripe  highlight-current-row v-loading="listLoading"  style="width: 100%">
      <!-- <el-table-column type="selection" ></el-table-column> -->

      <el-table-column label="键"  >
        <template scope="scope">
          <span class="see-details" @click="modifyItem(scope.row.id,-1)" v-if="showRead">{{scope.row.key}}</span>
          <span v-else>{{scope.row.key}}</span>
        </template>
      </el-table-column>
      <el-table-column prop="name" label="键名称" ></el-table-column>
      <el-table-column prop="value" label="值"  ></el-table-column>
      <el-table-column  label="类型"  >
        <template scope="scope">
            {{scope.row.type==1? '回调地址':'接口开关'}}
        </template>
      </el-table-column>
      <el-table-column label="状态"  >
        <template scope="scope">
          {{scope.row.vstatus==0 ? '未开通' : scope.row.vstatus==1 ? '已开通':'已停用'}}
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" width="210">
        <template scope="scope">
          <el-tooltip content="停用" placement="top" v-if="scope.row.vstatus==1">
            <el-button class="fa fa-stop-circle-o " aria-hidden="true" type="primary" size="mini"   @click="stopVstatus(scope.row.id,scope.$index)" :disabled="showStop ? false:true" ></el-button>
          </el-tooltip>
          <el-tooltip content="开启" placement="top" v-else>
            <el-button class="fa fa-play-circle-o" type="primary" size="mini"  @click="startVstatus(scope.row.id,scope.$index)" :disabled="showStart ? false:true" ></el-button>
          </el-tooltip>
          <el-tooltip content="编辑" placement="top">
            <el-button class="el-icon-edit" type="primary"  size="mini" @click="modifyItem(scope.row.id,2)" :disabled="scope.row.vstatus!=1&&showUpdate ? false:true"></el-button>
          </el-tooltip>
          <el-tooltip content="删除" placement="top">
            <el-button class="el-icon-delete" type="primary" size="mini" @click="removeItem(scope.row.id)" :disabled="scope.row.vstatus!=1&&showDelete ? false:true"></el-button>
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
    <addconfiginfo  ref="add"  :showType="showType" :showUpdate="showUpdate" :childValue="childValue" @childMethods="childMethods"></addconfiginfo>
  </section>
</template>
<script>
  import addconfiginfo from './Addconfiginfo.vue'
  export default {
    name:'in-configinfolist',
    data(){
      return{
        select:'2',
        showCreate:false,
        showDelete:false,
        showStart:false,
        showStop:false,
        showUpdate:false,
        showRead:false,
        pageNo:1,
        totalNum:0,
        pageSize:20,
        listItems:[],
        search:{
          key:'',
          name:'',
          type:''
        },
        listLoading: false,
        showType:1,  //显示弹窗类型  1：添加  2：修改
        childValue:'',
      } 
    },
    components:{
      addconfiginfo
    },
    mounted(){
      this.getData()
    },
    methods:{
      childMethods(msg){
        if(msg==1){
          this.search.name=''
          this.search.key=''
          this.search.type=''
        }
        this.getData();
      },
      async getData(){
        this.listLoading=true
        let d={
            page:this.pageNo,
            pageSize:this.pageSize,
            search_LIKE_name: this.search.name,
            search_LIKE_key:this.search.key,
            search_LIKE_type:this.search.type,
        };
        let res = await this.$get('/couponSys/bConfigInfo/page.json',d);
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
        if(res.functions&&res.functions.read){
          this.showRead=true
        } 
      },
      //点击搜索
      async searchItem(){
        this.pageNo=1;
        this.getData()
      },
      //点击修改操作
      async modifyItem(id,status){
        let d={
            id:id
        };
        let res = await this.$get('/couponSys/bConfigInfo/toUpdate.json',d);
        if(res.errcode==0){
          this.$refs.add.addFormVisible=true;
          this.$refs.add.$nextTick(()=>{
            this.$refs.add.resetForm()
            this.showType=status;
            let item={
                name: res.entity.name,
                key: res.entity.key,
                value:res.entity.value,
                vstatus: res.entity.vstatus,
                type: res.entity.type,
                description:res.entity.description
            }
            this.$refs.add.ruleForm=item;
            this.childValue=res.entity;
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
            let res = await this.$get('/couponSys/bConfigInfo/delete.json',d);
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
        let res = await this.$get('/couponSys/bConfigInfo/stop.json',d);
        if(res.errcode==0){
          this.listItems[index].vstatus=2
        }
      },
      //点击开启
      async startVstatus(id,index){
        let d={
            id:id
        };
        let res = await this.$get('/couponSys/bConfigInfo/start.json',d);
        if(res.errcode==0){
          this.listItems[index].vstatus=1
        }
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
            key: '',
            value:'',
            vstatus: '0',
            type: '',
            description:''
          }
          this.$refs.add.ruleForm=item;
        })

      },
    }

  }
</script>
<style lang="scss" >

  
  
</style>
