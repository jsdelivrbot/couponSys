<template>
  <section>
    <!--工具条-->
    <el-col :span="24" class="toolbar" >
      <el-row >
        <el-col class="search-box" v-if="marketinglistadd">
          <el-button class="el-icon-plus button-primary-hover" @click="handleAdd"  > 新增推送活动</el-button>
        </el-col>
        <el-col class="search-item">
          <div class="search-box">
            <span class="search-title">营销活动编码</span>
            <el-input v-model="search.code" placeholder="请输入活动编码" style="width:200px" :clearable="true"></el-input>
          </div>
          <div class="search-box">
            <span class="search-title">营销活动名称</span>
            <el-input v-model="search.name" placeholder="请输入活动名称" style="width:200px" :clearable="true"></el-input>
          </div>
          <div class="search-box">
            <span class="search-title">创建时间</span>
            <el-date-picker
                :editable="false"
                v-model="search.time"
                type="daterange"
                :default-time="['00:00:00', '23:59:59']"
                placeholder="选择日期时间"
                style="width:240px">
              </el-date-picker>
          </div>
          
        </el-col> 
        <el-col class="search-item">
          <div class="search-box">
            <span class="search-span">营销活动状态：</span>
            <el-radio v-model="search.status" label="">全部</el-radio>
            <el-radio v-model="search.status" label="1">已执行</el-radio>
            <el-radio v-model="search.status" label="2">未执行</el-radio>
          </div>
          <div class="search-box">
            <el-button  type="primary" @click="searchItem" class="el-icon-search" > 搜索</el-button>
          </div>
        </el-col>
      </el-row>
    </el-col>
    
    <!--列表-->
    <el-table :data="listItems" stripe   highlight-current-row v-loading="listLoading"  style="width: 100%">
      <!-- <el-table-column type="selection" ></el-table-column> -->
      <el-table-column  label="营销活动编号"  >
        <template scope="scope">
          <span class="see-details" @click="modifyItem(scope.row.id,-1)">{{scope.row.code}}</span>
        </template>
      </el-table-column>
      <el-table-column prop="name" label="营销活动名称"  ></el-table-column>
      <el-table-column  label="活动类型" >
        <template scope="scope">
          {{scope.row.type=='1'?'推送活动':''}}
        </template>
      </el-table-column>
      <el-table-column prop="messageConfigName" label="消息通知" ></el-table-column>
      <el-table-column prop="pushPrize" label="推送奖品"  :show-overflow-tooltip='true'></el-table-column>
      <el-table-column prop="createTime" label="创建日期" ></el-table-column>
      <el-table-column prop="fkCreatorUserName" label="创建人" ></el-table-column>
      <el-table-column prop="memberNumber" label="参与用户数" ></el-table-column>
      <el-table-column label="操作" width="160" align="center">
        <template scope="scope">
          <el-tooltip content="推送" placement="top">
            <el-button class="fa fa-paper-plane-o" type="primary" size="mini" @click="pushItem(scope.row.id)" :disabled="marketinglistpush&&scope.row.status==2 ? false:true"></el-button>
          </el-tooltip>
          <el-tooltip :content="scope.row.status==2?'编辑':'查看'" placement="top">
            <el-button :class="scope.row.status==2?'el-icon-edit':'el-icon-view'" type="primary"  size="mini" @click="modifyItem(scope.row.id,scope.row.status)" :disabled="marketinglistupdate ? false:true"></el-button>
          </el-tooltip>
          <el-tooltip content="删除" placement="top">
            <el-button class="el-icon-delete" type="primary" size="mini" @click="removeItem(scope.row.id)" :disabled="marketinglistdelete&&scope.row.status==2 ? false:true"></el-button>
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
    
  </section>
</template>
<script>
  export default {
    name:'in-marketinglist',
    props:['marketinglistadd','marketinglistupdate','marketinglistdelete','marketinglistpush'],
    data(){
      return{
        showAdd:false,
        pageNo:1,
        totalNum:0,
        pageSize:20,
        listItems:[],
        search:{
          code:'',
          name:'',
          time:[],
          status:'',
        },
        listLoading: false,
        showType:1,  //显示弹窗类型  1：添加  2：修改
        childValue:'',
      } 
    },
    components:{
      
    },
    mounted(){
      this.getData()
      // eventBus.$on('resetMarketinglist',(msg)=> {
      //     this.getData();
      // })
    },
    activated(){
      this.getData()
    },
    methods:{
      
      async getData(){
        this.listLoading=true
        let d={
            page:this.pageNo,
            pageSize:this.pageSize,
            search_LIKE_code: this.search.code,
            search_LIKE_name: this.search.name, 
            search_EQ_status: this.search.status, 
            search_GTE_createTime:this.search.time?this.$format(this.search.time[0]):'',
            search_LTE_createTime:this.search.time?this.$format(this.search.time[1]):'',

        };
        let res = await this.$get('/couponSys/sendCouponScheme/page.json',d);
        this.listLoading=false
        this.listItems=res.content;
        this.totalNum=res.totalNum;
        if(res.functions&&res.functions.add){
          this.showAdd=true
        }
      },
      //点击搜索
      async searchItem(){
        this.pageNo=1;
        this.getData()
      },
      //点击推送
      pushItem(id){
        this.$confirm('确认推送吗?', '提示', {
            //type: 'warning'
        }).then(async () => {
            let d={
                id:id
            };
            this.listLoading=true
            let res = await this.$get('/couponSys/sendCouponScheme/issue.json',d);
            this.listLoading=false
            if(res.errcode==0){
              this.getData();
              this.$message.success('推送成功');
            }else{
              this.$message.error(res.msg);
            }
        })
        
      },
      //点击修改操作
      async modifyItem(id,status){
        this.$router.push({path:'/home/coupon/addmarketing',query:{id:id,status:status}})
      },
      //点击删除
      removeItem(id){
        this.$confirm('确认删除吗?', '提示', {
            //type: 'warning'
        }).then(async () => {
            let d={
                id:id
            };
            let res = await this.$get('/couponSys/sendCouponScheme/delete.json',d);
            if(res.errcode==0){
              this.getData();
              this.$message.success('删除成功');
            }else{
              this.$message.error(res.msg);
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
        this.$router.push('/home/coupon/addmarketing')
        
      },
      //点击核销优惠券
      showCancellation(){
        this.$emit('cancellationMatheds',1)
      }
    }

  }
</script> 
<style lang="scss" scoped>
  .el-table__row .fa {
    padding: 6px 8px;
}
</style>
