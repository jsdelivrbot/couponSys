<template>
  <section>
    <!--工具条-->
    <el-col :span="24" class="toolbar">
      <el-row >
        <el-col class="search-item">
          <div class="search-box">
            <span class="search-title">优惠券类型</span>
            <el-select v-model="search.type"  placeholder="请选择" style="width:172px">
              <el-option label="优惠券" :value="1"></el-option>
              <el-option label="乐橙券" :value="2"></el-option>
              <el-option label="外平台券" :value="3"></el-option>
            </el-select>
          </div>
          <div class="search-box">
            <span class="search-title">优惠券编码</span>
            <el-input v-model="search.code" placeholder="请输入优惠券编码" style="width:200px" :clearable="true"></el-input>
          </div>
          <div class="search-box">
            <span class="search-title">优惠券名称</span>
            <el-input v-model="search.name" placeholder="请输入优惠券名称" style="width:200px" :clearable="true"></el-input>
          </div>
          <div class="search-box">
            <span class="search-title">发放时间</span>
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
            <span class="search-title">会员编码</span>
            <el-input v-model="search.memberCode" placeholder="请输入会员编码" style="width:200px" :clearable="true"></el-input>
          </div>
          <div class="search-box">
            <span class="search-title">会员名</span>
            <el-input v-model="search.memberName" placeholder="请输入会员名" style="width:200px" :clearable="true"></el-input>
          </div>
          <div class="search-box">
            <span class="search-title">会员手机号</span>
            <el-input v-model="search.mobilephone" placeholder="请输入会员手机号" style="width:200px" :clearable="true" :minlength="5" :maxlength="11"></el-input>
          </div>
          
        </el-col>
        <el-col class="search-item" >
          <div class="search-box" v-if="search.type==3">
              <span class="search-span">券状态：</span>
              <el-radio v-model="search.vstatus1" label="">全部</el-radio>
              <el-radio v-model="search.vstatus1" label="1">已发放</el-radio>
              <el-radio v-model="search.vstatus1" label="3">已过期</el-radio>
              <el-radio v-model="search.vstatus1" label="4">未发放</el-radio>
          </div>
          <div class="search-box" v-else>
              <span class="search-span">券状态：</span>
              <el-radio v-model="search.vstatus" label="">全部</el-radio>
              <el-radio v-model="search.vstatus" label="1">未使用</el-radio>
              <el-radio v-model="search.vstatus" label="2">已使用</el-radio>
              <el-radio v-model="search.vstatus" label="3">已过期</el-radio>
              <el-radio v-model="search.vstatus" label="4">未领取</el-radio>
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

      <el-table-column prop="couponSchemecode" label="优惠券编码"  ></el-table-column>
      <el-table-column prop="couponSchemeName" label="优惠券名称"  ></el-table-column>
      <el-table-column prop="verificationCode" label="优惠券劵码" v-if="selType!=1"></el-table-column>
      <el-table-column  label="券类型" >
        <template scope="scope">
          {{selType==1 ? '优惠劵' : selType==2 ? '乐橙劵' : '外平台劵'}}
        </template>
      </el-table-column>
      <el-table-column prop="sendSchemeName" label="营销活动名称"  ></el-table-column>
      <el-table-column prop="memberName" label="会员名" ></el-table-column>
      <el-table-column prop="mobilephone" label="手机号" ></el-table-column>
      <el-table-column label="到期时间" >
        <template scope="scope">
            {{$format(scope.row.dueTime)}}
        </template>
      </el-table-column>
      <el-table-column  label="发放时间" >
        <template scope="scope">
            {{$format(scope.row.receiveTime)}}
        </template>
      </el-table-column>
      
      <el-table-column  label="劵状态" >
        <template scope="scope" >
          <template v-if="selType==3">
            {{scope.row.vstatus==1 ? '已发放' : scope.row.vstatus==3 ? '已过期': scope.row.vstatus==4 ? '未发放':''}}
          </template>
          <template v-else>
            {{scope.row.vstatus==1 ? '未使用' : scope.row.vstatus==2 ? '已使用': scope.row.vstatus==3 ? '已过期':'未领取'}}
          </template>
          
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
    name:'in-grantlist',
    data(){
      return{
        showAdd:false,
        pageNo:1,
        totalNum:0,
        pageSize:20,
        listItems:[],
        search:{
          name:'',
          type:1,
          memberName:'',
          memberCode:'',
          mobilephone:'',
          vstatus:'',
          vstatus1:'',
          code:'',
          time:[],
        },
        listLoading: false,
        selType:1
      } 
    },
    components:{
      
    },
    mounted(){
      this.getData()
    },
    activated(){
      this.getData()
    },
    methods:{
      
      async getData(){
        this.listLoading=true
        this.selType=this.search.type
        let d={
            page:this.pageNo,
            pageSize:this.pageSize,
            sortType:'receiveTime',
            search_LIKE_couponSchemeName: this.search.name,
            search_LIKE_couponSchemecode: this.search.code,
            search_EQ_vstatus:this.selType==3?this.search.vstatus1:this.search.vstatus,
            search_EQ_memberName:this.search.memberName,
            search_EQ_memberCode:this.search.memberCode,
            search_EQ_mobilephone:this.search.mobilephone,
            search_GTE_receiveTime:this.search.time?this.$format(this.search.time[0]):'',
            search_LTE_receiveTime:this.search.time?this.$format(this.search.time[1]):'',
        };
        let res;
        if(this.search.type==1){
          res = await this.$get('/couponSys/couponOnline/page.json',d);
        }else if(this.search.type==2){
          res = await this.$get('/couponSys/couponOffline/page.json',d);
        }else{
          res = await this.$get('/couponSys/couponOutplatform/page.json',d);
        }
        if(res){
          this.listLoading=false
          this.listItems=res.content;
          this.totalNum=res.totalNum;
        }else{
          this.listLoading=false
          this.listItems=[];
          this.totalNum=0;
        }
        
        
      },
      //点击搜索
      async searchItem(){
        if (this.search.mobilephone.length<6&&this.search.mobilephone.length>0) {
          this.$message.error('至少输入6位手机号才能搜索');
          return
        }
        this.pageNo=1;
        this.getData()
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
      
      
      
    }

  }
</script>
<style lang="scss" >
  
</style>
