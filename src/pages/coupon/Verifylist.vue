<template>
  <section>
    <!--工具条-->
    <el-col :span="24" class="toolbar" >
      <el-row >
        <el-col class="search-item">
          <div class="search-box">
            <span class="search-title">优惠券类型</span>
            <el-select v-model="search.type"  placeholder="请选择" style="width:122px">
              <el-option label="乐橙券" :value="2"></el-option>
              <!-- <el-option label="外平台券" :value="3"></el-option> -->
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
            <span class="search-title">核销日期</span>
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
            <span class="search-title">核销门店</span>
            <!-- <el-cascader
                :options="fourshops"
                @active-item-change="getSection"
                :props="props"
                :clearable="true"
                style="width:150px"
                v-model="search.store">
            </el-cascader> -->
            <el-select v-model="search.store" placeholder="请选择门店" style="width:135px" :clearable="true">
              <el-option v-for="(item,index) in fourshops" :label="item.name" :value="item.id+''" ></el-option>
            </el-select>
            
          </div>
          <div class="search-box">
            <span class="search-title">会员编码</span>
            <el-input v-model="search.memberCode" placeholder="请输入会员编码" style="width:200px" :clearable="true"></el-input>
          </div>
          <div class="search-box">
            <span class="search-title">会员名</span>
            <el-input v-model="search.memberName" placeholder="请输入会员名" style="width:200px" :clearable="true"></el-input>
          </div>
          <div class="search-box">
            <span class="search-title">手机号</span>
            <el-input v-model="search.mobilephone" placeholder="请输入手机号" style="width:200px" :clearable="true" :minlength="5" :maxlength="11"></el-input>
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
      <el-table-column prop="couponSchemeCode" label="优惠券编码"></el-table-column>
      <el-table-column prop="couponSchemeName" label="优惠券名称"></el-table-column>
      <el-table-column prop="verificationCode" label="优惠券劵码"></el-table-column>
      <el-table-column  label="券类型">
        <template scope="scope">
          {{scope.row.couponSchemeType==1?'优惠券':scope.row.couponSchemeType==2?'乐橙券':'外平台券'}}
        </template>
      </el-table-column>
      <el-table-column prop="memberName" label="会员名"></el-table-column>
      <el-table-column prop="mobilephone" label="手机号"></el-table-column>
      <el-table-column  label="到期时间" >
        <!-- {{$format(scope.row.dueTime)}} -->
      </el-table-column>
      <el-table-column  label="领券时间" >
        <template scope="scope">
            {{$format(scope.row.receiveTime)}}
        </template>
      </el-table-column>
      <el-table-column  label="核销日期" >
        <template scope="scope">
            {{$format(scope.row.verificationTime)}}
        </template>
      </el-table-column>
      <el-table-column prop="fkFourshopName" label="核销门店" ></el-table-column>
      <el-table-column prop="operater" label="核销人" ></el-table-column>
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
    name:'in-verifylist',
    // props:['activeName'],
    data(){
      return{
        pageNo:1,
        totalNum:0,
        pageSize:20,
        listItems:[],
        search:{
          type:'', 
          code:'',
          name:'',
          time:[],
          store:'',
          memberName:'',
          memberCode:'',
          mobilephone:''
        },
        listLoading: false,
        fourshops:[],
        props:{
            label:'name',
            value: 'id',
            children: 'children'
        },

      } 
    },
    components:{
      
    },
    mounted(){
      this.getData()
      this.getStore()
      eventBus.$on('resetVerifylist',(msg)=> {
          this.getData();
      })
    },
    methods:{
      async getSection(val){      
          let index=0
          for(var i=0;i<this.fourshops.length;i++){
              if(this.fourshops[i].id==val[0]){
                  index=i
                  break;
              }
              
          }
          if(this.fourshops[index].children==0){
              let d={
                  fourshop:val[0]
              }
              let res = await this.$get('/couponSys/common/getDeptByFourshop.json',d);
              if(Object.keys(res).length!=0){
                  this.fourshops[index].children = res
              }
              
          }
      },
      async getStore(){
        let res = await this.$get('/couponSys/couponVerification/create.json');
        if (res.errcode == 0) {
          this.fourshops = res.fourslist
          // for(var i in this.fourshops){
          //     this.$set(this.fourshops[i],'children',[])
          // }
        }
      },
      async getData(){
        this.listLoading=true
        let d={
            page:this.pageNo,
            pageSize:this.pageSize,
            sortType:'verificationTime',
            search_EQ_couponSchemeType: this.search.type,
            search_LIKE_couponSchemeCode: this.search.code,
            search_LIKE_couponSchemeName: this.search.name,
            search_GTE_verificationTime:this.search.time?this.$format(this.search.time[0]):'',
            search_LTE_verificationTime:this.search.time?this.$format(this.search.time[1]):'',
            search_EQ_fkFourshopId:this.search.store,
            search_EQ_memberCode:this.search.memberCode,
            search_EQ_memberName:this.search.memberName,
            search_EQ_mobilephone:this.search.mobilephone,
            
        };
        let res = await this.$get('/couponSys/couponVerification/page.json',d);
        this.listLoading=false
        this.listItems=res.content;
        this.totalNum=res.totalNum;

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
      
      //点击删除
      removeItem(id){
        this.$confirm('确认删除吗?', '提示', {
            //type: 'warning'
        }).then(async () => {
            let d={
                id:id
            };
            let res = await this.$delete('/couponSys/mgrFunction/delete.json',d);
            if(res.errcode==0){
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
      
      
      
    }

  }
</script>
<style lang="scss" >
  
</style>
