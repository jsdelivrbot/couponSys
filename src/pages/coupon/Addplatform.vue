<template>
    <section>
        <el-row class="breadcrumb">券方案/{{id==null?'创建':queryStatus==2?'编辑':'查看'}}外平台券方案</el-row>
        <div class="create-info">
            <span>券方案编码：{{code}}</span>
            <span class="fr">创建人：{{username}} | 创建时间：{{$format(createTime)}}</span>
        </div>
        <el-card v-loading="loading">
            
            <el-row >
                <el-col class="module-title" >基本设置</el-col>
            </el-row>
            <el-row >
                <el-col :span="24">
                    <span class="search-span">劵方案名称：</span>
                    <el-input v-model="name" style="width:40%" placeholder="优惠券名称不可以超过25个字符" @input="checkName(name,25,'name')" :disabled="queryStatus!=2&&id!=null?true:false"></el-input>
                </el-col>
            </el-row>
            <el-row >
                <el-col :span="24">
                    <span class="search-span">发券平台：</span>
                    <el-select v-model="platformCode"  placeholder="选择发券平台" :disabled="queryStatus!=2&&id!=null?true:false">
                        <el-option v-for="(item,index) in platformlist" :label="item.name" :value="item.code" ></el-option>
                    </el-select>
                </el-col>
            </el-row>
            <el-row >
                <el-col>
                    <span class="search-span fl">上传劵码文件：</span>
                    <el-upload
                        :disabled="queryStatus!=2&&id!=null?true:false"
                        style="margin-left: 5px;"
                        class="fl"
                        ref="upload"
                        :action="postUrl" 
                        name="files"
                        accept=".csv"
                        drag
                        :on-success="handleAvatarSuccess"
                        :on-progress="handleProgress"
                        :file-list="fileList"
                        >
                        <i class="el-icon-upload"></i>
                        <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
                        <div class="el-upload__tip" slot="tip">只能上传.csv格式文件，且不超过1M</div>
                    </el-upload>
                </el-col>
            </el-row>
            <el-row >
                <el-col >
                    <span class="search-span">优惠券类型：</span>
                    <el-button :type="thresholdType==1?'primary':''" @click="thresholdType=1">满减券</el-button>
                    <el-button :type="thresholdType==2?'primary':''" @click="thresholdType=2">无门槛代金券</el-button>
                </el-col>
                
            </el-row>
            
            <el-row >
                <el-col  v-if="thresholdType==1">
                    <span class="search-span">优惠券面值：</span>
                    <span>订单满</span><el-input style="width:80px;margin:0 10px" v-model="thresholdAmt" @input="keyupMoney('thresholdAmt',thresholdAmt)" :disabled="queryStatus!=2&&id!=null?true:false"></el-input><span>减</span><el-input style="width:80px;margin:0 10px" v-model="couponAmt" @input="keyupMoney('couponAmt',couponAmt)" :disabled="queryStatus!=2&&id!=null?true:false"></el-input><span>元</span>
                </el-col>
                <el-col  v-if="thresholdType==2">
                    <span class="search-span">优惠券面值：</span>
                    <el-input style="width:80px;margin-right:10px"  v-model="couponAmt" @input="keyupMoney('couponAmt',couponAmt)" :disabled="queryStatus!=2&&id!=null?true:false"></el-input><span>元</span>
                </el-col>
                
            </el-row>
            <el-row >
                <el-col :md="8">
                    <span class="search-span">总发行量：</span>
                    <span>{{issuanceQty}} 张</span>
                </el-col>
            </el-row>
            <el-row >
                <el-col >
                    <span class="search-span">优惠券有效期：</span>
                    <el-select v-model="validityType" placeholder="请选择" :disabled="queryStatus!=2&&id!=null?true:false">
                        <el-option label="截止日期至" :value="1"></el-option>
                        <el-option label="自定义时间段" :value="2"></el-option>
                        <el-option label="领取后" :value="3"></el-option>
                    </el-select>
                    <span v-show="validityType==1">
                        <el-date-picker
                            :disabled="queryStatus!=2&&id!=null?true:false"
                            v-model="deadTime"
                            :editable="false"
                            type="datetime"
                            placeholder="截止时间"
                            :default-value="new Date(new Date().getFullYear(),new Date().getMonth(),new Date().getDate(),23,59,59)"
                            :picker-options="pickerBeginDateBefore">
                        </el-date-picker>
                    </span>
                    <span v-show="validityType==2">
                        <el-date-picker
                            v-model="validityTime"
                            :disabled="queryStatus!=2&&id!=null?true:false"
                            :editable="false"
                            @change="initTimeArr('availableTime',availableTime)"
                            type="datetimerange"
                            :default-time="['00:00:00', '23:59:59']"
                            placeholder="选择时间范围"
                            :picker-options="pickerBeginDateBefore">
                        </el-date-picker>
                    </span>
                    <span v-show="validityType==3">
                        <span>至领券之日起</span><el-input style="width:80px;margin:0 10px" v-model="validityDay"  @input="keyupNum('validityDay',validityDay)" :disabled="queryStatus!=2&&id!=null?true:false"></el-input><span>天</span>
                    </span>
                </el-col>
            </el-row>
            
            <el-row >
                <el-col >
                    <span class="search-span">使用须知：</span>
                    <el-input type="textarea" :rows="4" placeholder="请详细描述该优惠券使用的说明内容，不超过500个字符" style="width:50%" v-model="notice" @input="checkName(notice,500,'notice')" :disabled="queryStatus!=2&&id!=null?true:false"></el-input>
                </el-col>
            </el-row>
            <el-row >
                <el-col class="module-title" >推广设置</el-col>
            </el-row>
            <el-row >
                <el-col >
                    <span class="search-span">可领取时间：</span>
                    <el-date-picker
                        :disabled="queryStatus!=2&&id!=null?true:false"
                        v-model="availableTime"
                        :editable="false"
                        @change="initTimeArr('availableTime',availableTime)"
                        type="datetimerange"
                        :default-time="['00:00:00', '23:59:59']"
                        placeholder="选择时间范围"
                        :picker-options="pickerBeginDateBefore">
                    </el-date-picker>
                </el-col>
            </el-row>
            <el-row >
                <el-col >
                    <span class="search-span">限领设置：</span>
                    <el-select v-model="limitType" placeholder="请选择" :disabled="queryStatus!=2&&id!=null?true:false">

                        <el-option label="活动期间每人限领" :value="1"></el-option>
                        <el-option label="活动期间每人每日限领" :value="2"></el-option>
                    </el-select>
                    <el-input style="width:80px;margin:0 10px" v-model="limitQty"  @input="keyupNum('limitQty',limitQty)" :disabled="queryStatus!=2&&id!=null?true:false"></el-input><span>张</span>
                </el-col>
            </el-row>
            <el-row >
                <el-col >
                    <span class="search-span">会员等级：</span>
                    <el-select v-model="mbeGrade" multiple placeholder="选择会员等级" :disabled="queryStatus!=2&&id!=null?true:false">
                        <el-option label="普卡" value="251"></el-option>
                        <el-option label="银卡" value="252"></el-option>
                        <el-option label="金卡" value="253"></el-option>
                        <el-option label="钻石卡" value="254"></el-option>
                    </el-select>
                </el-col>
            </el-row>
            
            <el-row >
                <el-col >
                    <span class="search-span" ></span>
                    <el-checkbox v-model="isPublic" :disabled="queryStatus!=2&&id!=null?true:false">领券中心页面显示该券方案</el-checkbox>
                </el-col>
            </el-row>
            <el-row >
                <el-col >
                    <span class="search-span" ></span>
                    <el-checkbox v-model="isShared" :disabled="queryStatus!=2&&id!=null?true:false">领券链接可被分享</el-checkbox>
                </el-col>
            </el-row>
            <el-row class="btn-group" >
                <el-button type="primary" size="large" @click="saveInfo" v-show="!id||queryStatus==2" >保存</el-button>
                <el-button type="primary" size="large" @click="goBack">返回</el-button> 
            </el-row>
        </el-card>
    </section>
</template>
<script>
    export default {
        name:'addplatform',
        data(){
            return{
                postUrl:host+"/couponSys/fileUtil/uploadCodeFile.json?fileName=''",
                queryStatus:this.$route.query.status||null,
                id:this.$route.query.id||null,
                code:'',  //劵方案编码
                createTime:'',  //创建时间
                username:'',
                creater:'',  //创建人账号
                name:'',   //优惠券名称
                platformCode:'',  //发券平台
                platformName:'',
                platformlist:[],
                type:3,  //劵类型  1.优惠券  2.乐橙劵   3.外平台劵
                couponAmt:'',  //优惠券面值
                thresholdType:1, //1.有门槛  2.无门槛
                thresholdAmt:'',  //满减金额(有门槛)
                issuanceQty:0, //发行劵总数量
                remainQty:'',  //发行劵剩余数量
                validityType:1,  //1.绝对时间  2.相对时间
                validityDay:'',   //相对时间  多少天
                deadTime:null,  //截止之间
                validityTime:[], //绝对时间有效期时间范围
                notice:'',  //须知
                availableTime:[],  //可领取时间范围
                mbeGrade:'',  //会员等级
                limitType:1,  //限额设置 1.没人限额 2.期间每人限额  3.期间每人每天限额
                limitQty:'',  //限额数量
                isPublic:false,  //领劵中心是否可见  1.是  2.否
                isShared:false,  //是否分享  1.是  2.否
                vstatus:1,
                fileList:[],
                codeFile:'',
                codeFileOriginal:'',
                loading:false,
                pickerBeginDateBefore:{
                    disabledDate(time) {
                        return time.getTime() < Date.now() - 8.64e7;
                    }
                
                }
            }
            
        },

        mounted(){
            if(this.id==null){
                this.getCode()
            }else{
                this.initInfo()
            }
            
        },
        methods:{
            initTimeArr(name,time){
                if(!time){
                    this[name]=[]
                }
            },
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
                let res = await this.$get('/couponSys/couponSchemeOutplatform/toUpdate.json',d);
                this.code=res.entity.code
                this.name=res.entity.name
                this.username=res.entity.createrName
                this.creater=res.entity.creater
                this.createTime=res.entity.createTime
                this.thresholdType=res.entity.thresholdType
                this.couponAmt=res.entity.couponAmt
                this.thresholdAmt=res.entity.thresholdAmt
                this.issuanceQty=res.entity.issuanceQty
                this.validityType=res.entity.validityType
                this.validityDay=res.entity.validityDay
                this.deadTime=res.entity.deadTime
                this.validityTime=[res.entity.startTime,res.entity.endTime]
                this.notice=res.entity.notice
                this.availableTime=[res.entity.availableStartTime,res.entity.availableEndTime]
                this.mbeGrade=res.entity.mbeGrade.split(',')
                this.limitType=res.entity.limitType
                this.limitQty=res.entity.limitQty
                this.isPublic=res.entity.isPublic==1?true:false
                this.isShared=res.entity.isShared==1?true:false
                this.vstatus=res.entity.vstatus
                this.codeFile=res.entity.codeFile
                this.platformlist=res.list;
                this.platformCode=res.entity.platformCode
                this.postUrl=host+'/couponSys/fileUtil/uploadCodeFile.json?fileName='+res.entity.codeFile
                let file={
                    name:res.entity.codeFileOriginal
                }
                this.fileList.push(file)
            },
            async getCode(){
                
                let res = await this.$get('/couponSys/couponSchemeOutplatform/create.json');
                this.code=res.entity.code;
                this.platformlist=res.list
                this.createTime=this.$format(res.entity.createTime)
                let userinfo = unescape(sessionStorage.getItem('userinfo'));
                if (userinfo) {
                    userinfo = JSON.parse(userinfo);
                    this.creater=userinfo.id;
                    this.username=userinfo.name
                }

            },
            checkValue(){

                if(this.name==''||this.couponAmt==''||(this.thresholdType==1&&this.thresholdAmt=='')||this.availableTime.length==0||this.limitQty==''||this.mbeGrade==''||this.codeFile==''||this.platformCode==''){
                    this.$message.error('信息设置不完整');
                    return false
                }
                if((this.validityType==1&&!this.deadTime)||(this.validityType==2&&this.validityTime.length==0)||(this.validityType==3&&this.validityDay=='')){
                    this.$message.error('信息设置不完整');
                    return false
                }

                if(this.thresholdType==1&&(parseFloat(this.thresholdAmt)<parseFloat(this.couponAmt))){
                    this.$message.error('订单面值必须大于满减券面值');
                    return false
                }

                if((this.validityType==1&&new Date(this.deadTime).getTime()<new Date(this.availableTime[1]).getTime())||(this.validityType==2&&new Date(this.validityTime[1]).getTime()<new Date(this.availableTime[1]).getTime())){
                    this.$message.error('可领取时间必须在优惠券有效期截止时间之前');
                    return false
                }
                return true
            },
            async saveInfo(){
                let check=this.checkValue()
                if(!check){
                    return
                }
                for(let i=0;i<this.platformlist.length;i++){
                    if(this.platformlist[i].code==this.platformCode){
                        this.platformName=this.platformlist[i].name
                    }
                }

                let d={
                    id:this.id,
                    code:this.code,
                    createTime:this.$format(this.createTime),
                    creater:this.creater,
                    name:this.name,
                    type:this.type,
                    couponAmt:this.couponAmt,
                    thresholdType:this.thresholdType,
                    thresholdAmt:this.thresholdType==1?this.thresholdAmt:'',
                    issuanceQty:this.issuanceQty,
                    remainQty:this.remainQty,
                    validityType:this.validityType,
                    validityDay:this.validityDay,
                    deadTime:this.$format(this.deadTime),
                    startTime:this.$format(this.validityTime[0]),
                    endTime:this.$format(this.validityTime[1]),
                    notice:this.notice,
                    availableStartTime:this.$format(this.availableTime[0]),
                    availableEndTime:this.$format(this.availableTime[1]),
                    mbeGrade:this.mbeGrade.join(','),
                    limitType:this.limitType,
                    limitQty:this.limitQty,
                    isPublic:this.isPublic?1:2,
                    isShared:this.isShared?1:2,
                    vstatus:this.vstatus,
                    codeFile:this.codeFile,
                    codeFileOriginal:this.codeFileOriginal,
                    platformCode:this.platformCode,
                    platformName:this.platformName


                }
                this.loading=true
                let res;
                if(this.id==null){
                    res = await this.$post('/couponSys/couponSchemeOutplatform/add.json',d);
                }else{
                    res = await this.$post('/couponSys/couponSchemeOutplatform/update.json',d);
                }
                this.loading=false
                if(res.errcode==0){
                    this.$message.success(this.id==null?'添加成功':'修改成功');
                    // eventBus.$emit('resetPlatformlist')
                    this.$router.go(-1)
                }else{
                    this.$message.error(res.msg);
                }
            },
            goBack(){
                this.$router.go(-1)
            },

            handleProgress(event, file, fileList){
                if(!file.name.endsWith('.csv')){
                    this.$refs.upload.abort(file);
                    this.fileList.push(file)
                    this.fileList.pop()
                    this.$message.error('只能上传.csv格式文件');
                }
            },
            handleAvatarSuccess(res, file) {
                if(res.errcode==0){
                    this.fileList=[];
                    this.fileList.push(file)
                    this.codeFile=res.codeFile
                    this.codeFileOriginal=res.codeFileOriginal
                    this.issuanceQty=res.codeNumber
                    this.postUrl=host+'/couponSys/fileUtil/uploadCodeFile.json?fileName='+this.codeFile
                }else{
                    this.fileList.pop()
                    this.$message.error(res.msg);
                }
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
