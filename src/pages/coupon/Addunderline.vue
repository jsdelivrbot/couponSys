<template>
    <section>
        <el-row class="breadcrumb">发放券/{{id==null?'创建':queryStatus==0?'编辑':'查看'}}线下制券</el-row>
        <div class="create-info">
            <span>线下制券编码：{{code}}</span>
            <span class="fr">创建人：{{username}} | 创建时间：{{$format(createTime)}}</span>
        </div>
        <el-card v-loading="loading">
            <el-row >
                <el-col class="module-title" >基本设置</el-col>
            </el-row>
            <el-row >
                <el-col >
                    <span class="search-span">线下制券名称：</span>
                    <el-input v-model="name" style="width:50%" placeholder="线下制券名称不可以超过100个字符" @input="checkName(name,100,'name')" :disabled="queryStatus!=0&&id!=null?true:false"></el-input>
                </el-col>
            </el-row>
            <el-row >
                <el-col >
                    <span class="search-span">活动类型：</span>
                    <el-button type="primary">线下制券</el-button>  
                </el-col>
            </el-row>
            <el-row >
                <el-col >
                    <span class="search-span">推送渠道：</span>
                    <el-button type="primary">线下渠道</el-button>  
                </el-col>
            </el-row>
            <el-row >
                <el-col class="module-title" >奖品设置</el-col>
            </el-row>
            <el-row>
                <el-col v-for="(item,index) in selprize" :key="index">
                    <span class="search-span"></span>
                    <el-tag type="primary">{{index+1}}</el-tag>
                    <el-cascader
                        style="margin-right:10px"
                        :options="prizearr"
                        :disabled="queryStatus!=0&&id!=null?true:false"
                        :props="props"
                        v-model="selprize[index]">
                    </el-cascader>
                </el-col>
                
            </el-row>
            <el-row class="btn-group" >
                <el-button type="primary" size="large" @click="saveInfo" v-show="!id||queryStatus==0" >保存</el-button>
                <el-button type="primary" size="large" @click="goBack">返回</el-button> 
            </el-row>
        </el-card>
    </section>
</template>
<script>
    export default {
        name:'addunderline',
        data(){
            return{
                queryStatus:this.$route.query.status,
                id:this.$route.query.id||null,
                code:'',  //劵方案编码
                createTime:'',  //创建时间
                username:'',
                creater:'',  //创建人账号
                name:'',   //线下制券名称
                props: {
                    label:'COUPONSCHEMENAME',
                    value: 'FKCOUPONSCHEMEID',
                    children: 'children'
                },
                prizearr: [{  
                    FKCOUPONSCHEMEID: 2,
                    COUPONSCHEMENAME: '乐橙券',
                    children: []
                }],
                selprize:[   //奖品值
                   []
                ],  
                loading:false
            }
            
        },

        mounted(){
            this.getPrize()
            if(this.id==null){
                this.getCode()
            }else{
                this.initInfo()
            }
            let userinfo = unescape(sessionStorage.getItem('userinfo'));
            if (userinfo) {
                userinfo = JSON.parse(userinfo);
                this.fkCreatorUserId=userinfo.id;
                this.username=userinfo.name
            }
        },
        methods:{
            checkName(val,num,name){
                 if(val){
                    this.$nextTick(()=>{
                        var len = 0;  
                        var arr=[]
                        for (var i=0; i<val.length; i++) {  
                            if (val.charCodeAt(i)>127 || val.charCodeAt(i)==94) {  
                                len += 2;  
                            } else {  
                                len ++;  
                            }
                            if(len>num){
                                this[name]=val
                                break;
                            }
                            arr.push(val[i])
                        }  
                        this[name]=arr.join('')
                    })     
                }
            },
            async initInfo(){
                let d={
                    id:this.id
                }
                let res = await this.$get('/couponSys/couponOfflineMake/toUpdate.json',d);
                if(this.queryStatus!=0&&this.id!=null){
                    let list=[{
                        COUPONSCHEMENAME:res.couponSchemeName,
                        FKCOUPONSCHEMEID:res.fkCouponSchemeId
                    }]
                    this.prizearr[0].children = list;
                }
                this.code=res.code
                this.createTime=res.createTime
                this.name=res.name
                this.pushChannel=res.pushChannel

                this.selprize=[]
                let item=[2,res.fkCouponSchemeId]
                this.selprize.push(item)
                
            },
            async getCode(){
                
                let res = await this.$get('/couponSys/couponOfflineMake/create.json');
                this.code=res.code
                this.createTime=res.createTime
            
            },
            async getPrize(val,index){
                if(this.queryStatus!=0&&this.id!=null){

                    return
                }
                for(let i=0;i<this.prizearr.length;i++){
                    let d={
                        type:this.prizearr[i].FKCOUPONSCHEMEID
                    }
                    let res = await this.$get('/couponSys/couponOfflineMake/queryCouponScheme.json',d);
                    
                    if(res.errcode==0){
                        let list=res.couponScheme.list
                        this.prizearr[i].children = list;
                    }
                }
            },
            
            
            async saveInfo(){
                if(this.name==''||this.selprize[0].length==''){
                    this.$message.error('信息设置不完整');
                    return
                }
                let d={   
                    content:{       
                        id:this.id,     
                        fkCreatorUserId:this.fkCreatorUserId, 
                        pushChannel:1,
                        fkCouponSchemeId:this.selprize[0][1],
                        code:this.code,
                        createTime:this.$format(this.createTime),
                        name:this.name,
                        type:1,
                    }
                }
                this.loading=true
                let res;
                if(this.id==null){
                    res = await this.$post('/couponSys/couponOfflineMake/add.json',d);
                }else{
                    res = await this.$post('/couponSys/couponOfflineMake/update.json',d);
                }
                this.loading=false
                if(res.errcode==0){
                    this.$message.success(this.id==null?'添加成功':'修改成功')
                    // eventBus.$emit('resetUnderlinelist')
                    this.$router.go(-1)
                }else{
                    this.$message.error(res.msg);
                }
            },
            goBack(){
                this.$router.go(-1)
            },
        }
    }
</script>
<style lang="scss" scoped>
    .search-span{
        display: inline-block;
        width: 140px;
        text-align: right;
    }
    
    .el-col{
        margin-bottom: 20px
    }
    .notes{
        margin-left: 30px;
    }
    .el-textarea{
        vertical-align: top;
    }
    .btn-group{
        margin-top:40px;
        text-align:center
    }
</style>
