<!-- 服务运行监管配置中心 -->
<template>
  <Layout>
    <Content>
      <Breadcrumb>
        <BreadcrumbItem>统一监管平台</BreadcrumbItem>
        <BreadcrumbItem>{{ title }}</BreadcrumbItem>
      </Breadcrumb>
      <Card style="min-height: 600px;">
        <Form ref="formValidate" inline :label-width="110" :model="serverData">
          <FormItem label="服务名称：">
            <Select class="queryItem" v-model="serverData.appIds" clearable>
              <Option v-for="item in appArr" :key="item.appId" :value="item.appId">{{item.name}}</Option>
            </Select>
          </FormItem>
          <FormItem label="运行状态：">
            <Select class="queryItem" v-model="serverData.runStatus" clearable>
              <Option :value="true">开启</Option>
              <Option :value="false">禁用</Option>
            </Select>
          </FormItem>
          <FormItem :label-width="20">
            <Button type="primary" @click="search">查询</Button>
            <Button type="primary" @click="addNew">新增</Button>
          </FormItem>
        </Form>
        <hy-table ref="selection" :data="data" :columns="columns" :total="pageOption.total" :page-size="pageOption.pageSize" @on-change="pageChange" @on-page-size-change="changePageSize" show-sizer border/>
      </Card>
      <Modal v-model="createModal" :title="modelTitle" width="40%" :mask-closable="false" @on-cancel="cancelFun">
        <Form ref="createForm" :model="createForm" :rules="createRule" :label-width="90">
          <!-- <FormItem label="服务名称" prop="arrIdx" v-if="createForm.appName!=''">
            <Select class="queryItem" :clearable="!disa" v-model="createForm.arrIdx" :disabled="disa">
              <Option v-for="(item, idx) in appArr" :key="idx" :value="item.id">{{item.name}}</Option>
            </Select>
          </FormItem> -->
          <FormItem label="服务名称" prop="arrIdx">
            <template v-if="!disa">
              <Select class="queryItem" :clearable="!disa" v-model="createForm.arrIdx">
                <Option v-for="(item, idx) in appArr" :key="idx" :value="idx">{{item.name}}</Option>
              </Select>
            </template>
            <template v-else>
              <Input type="text" v-model="createForm.appName" disabled/>
            </template>
          </FormItem>
          <FormItem label="预警级别" required>
            <Select v-model="createForm.level"  filterable :disabled="disa">
                <Option :value="1" >一般</Option>
                <Option :value="2" >较重</Option>
                <Option :value="3" >严重</Option>
                <Option :value="4" >特别严重</Option>
            </Select>
          </FormItem>
          <FormItem label="预警阈值" prop="limit">
            <Input type="text" v-model="createForm.limit" :disabled="disa" @on-change="onChange">
              <span slot="append">次/每</span>
              <Select v-model="createForm.limitCycle" slot="append" style="width: 80px" :disabled="disa" @on-change="limitCycleChange">
                <Option v-for="item in superviseTypeData" :value="item.id" :key="item.id">{{ item.name }}</Option>
              </Select>
            </Input>
          </FormItem>
          <FormItem label="责任人" prop="dutyUser">
            <template v-if="!disa">
              <Select v-model="createForm.dutyUser" ref="dutyUserReset" :clearable="true" filterable :disabled="disa">
                <Option v-for="item in dutyUsers" :value="item.gxqptEmpId" :key="item.gxqptEmpId">{{ item.name }}</Option>
              </Select>
            </template>
            <template v-else>
              <Input type="text" v-model="createForm.dutyUserName" disabled/>
            </template>
          </FormItem>
          <FormItem label="确认人" prop="confirmUser">
            <template v-if="!disa">
              <Select v-model="createForm.confirmUser" filterable multiple :disabled="disa">
                <Option v-for="item in confirmUsers" :value="item.gxqptEmpId" :key="item.gxqptEmpId">{{ item.name }}</Option>
              </Select>
            </template>
            <template v-else>
              <Input type="text" v-model="createForm.confirmUserName" disabled/>
            </template>
          </FormItem>
          <FormItem label="抄送人" prop="copyUser">
            <template v-if="!disa">
              <Select v-model="createForm.copyUser" filterable multiple :disabled="disa">
                <Option v-for="item in copyUsers" :value="item.gxqptEmpId" :key="item.gxqptEmpId">{{ item.name }}</Option>
              </Select>
            </template>
            <template v-else>
              <Input type="text" v-model="createForm.copyUserName" disabled/>
            </template>
          </FormItem>
          <FormItem label="提醒方式" prop="remindType">
            <Select class="queryItem"  :clearable="!disa" multiple v-model="createForm.remindType" :disabled="disa" @on-change="selectChange">
              <Option value="1">消息</Option>
              <Option value="2">短信</Option>
            </Select>
          </FormItem>
          <FormItem label="运行状态" prop="runStatusArr">
            <Select class="queryItem" v-model="createForm.runStatusArr" :disabled="disa">
              <Option value="1">开启</Option>
              <Option value="0">禁用</Option>
            </Select>
          </FormItem>
          <FormItem label="备注" prop="desc">
            <Input type="textarea" v-model="createForm.desc" :disabled="disa"/>
          </FormItem>
          <!-- <FormItem label="附件报告" style="margin-top: 40px;" prop="file">
            <hy-upload ref="evalReport" multiple :on-success="setEvalReport" :on-remove="removeEvalReport" :before-upload="evalBeforeUpload" :defaultFileList="createForm.attachment"></hy-upload>
          </FormItem> -->
        </Form>
        <div slot="footer" v-if="modelType!='2'">
          <Button type="primary" @click="saveHostRoom" size="large">确定</Button>
          <Button class="modalBtn" type="default" @click="cancelFun" size="large">取消</Button>
        </div>
        <div slot="footer" v-else>
          <Button class="modalBtn" type="default" @click="cancelFun" size="large">关闭</Button>
        </div>
      </Modal>
    </Content>
  </Layout>
</template>

<script>
  import api from '@/api/axiosApi'
  import superviseApiList from '@/api/superviseApiList'
  import comApiList from '@/api/comApiList'
  import hyUpload from '@/components/hengyun/hyUpload'
  import { mapState } from 'vuex'
  function getCreateForm() {
    return {
      appId:'',//服务名称
      arrIdx:'',//编号
      level: 1,//预警级别
      limit:'',//上限阀值
      limitCycle:'',//阀值周期
      dutyUser:'',//责任人
      confirmUser:[],//确认人
      copyUser:[],//抄送人
      remindType:[],//提醒方式
      runStatusArr:'',
      runStatus:"",//运行状态
      desc:'',//备注
    }
  }
  const superviseTypeData = [{name:"实时",id:1},{name:"天",id:2},{name:"周",id:3},{name:"月",id:4}];
  export default {
    data() {
      const vm =this
      return {
        title:this.$store.state.title,
        remindTypes: '',
        serverData: { //查询参数
          appId:[],//服务名称集合
          appIds:"",//服务名称
          runStatus:"",//运行状态
        },
        appIds:[],//全部服务名称集合
        modelType:'0',//0新增，1修改，2详情
        appArr:[],//应用列表
        superviseTypeData:superviseTypeData,
        modelTitle:'新建',
        personArr:[],//人员列表
        createForm:{
          appId:'',//服务名称
          arrIdx:'',//编号
          level: 1,//预警级别
          limit:'',//上限阀值
          limitCycle:'',//阀值周期
          dutyUser:'',//责任人
          confirmUser:[],//确认人
          copyUser:[],//抄送人
          remindType:[],//提醒方式
          runStatusArr:'1',
          runStatus:'',//运行状态
          desc:'',//备注
        },
        createModal:false,
        createRule: {
          arrIdx: [{
            required: true,
            trigger: 'change',
            validator: (rule, value, cb) => {
              if (!this.createForm.arrIdx && typeof this.createForm.arrIdx!='number') {
                cb(new Error('请选择服务名称！'))
                return
              }
              cb()
            }
          }],
          interfaceName: [{required: true, message: '接口名称不能为空', trigger: 'blur'}],
          dutyUser: [{
            required: true,
            trigger: 'change',
            validator: (rule, value, cb) => {
              if (!this.createForm.dutyUser) {
                cb(new Error('责任人不能为空'))
                return
              }
              cb()
            }
          }],
          confirmUser: [{
            required: true,
            trigger: 'change',
            validator: (rule, value, cb) => {
              if (this.createForm.confirmUser.length == 0) {
                cb(new Error('确认人不能为空'))
                return
              }
              cb()
            }
          }],
          superviseType: [{required: true, message: '监管方式不能为空', trigger: 'change'}],
          // remindType: [{required: true, message: '提醒方式不能为空', trigger: 'change'}],
          remindType: [{
            required: true,
            trigger: 'change',
            validator: (rule, value, cb) => {
              if (this.createForm.remindType.length==0) {
                cb(new Error('请选择提醒方式！'))
                return
              }
              cb()
            }
          }],
          desc: [{required: false, message: '请输入500字以内的字数', trigger: 'change',validator: function(rule, value, cb){
            if(value!=null && value !='' && value.length>500){
               cb(new Error(rule.message))
            }
            cb()
          }}],
          runStatusArr: [{
            required: true,
            trigger: 'change',
            validator: (rule, value, cb) => {
              if (this.createForm.runStatusArr=='') {
                cb(new Error('请选择运行状态！'))
                return
              }
              cb()
            }
          }],
          limit: [{
            required: true,
            trigger: 'blur',
            validator: (rule, value, cb) => {
                if ((value === null || value === '') || !vm.createForm.limitCycle) {
                  cb(new Error('预警阈值/频率不能为空'))
                  return
                }else{
                  let reg = /^\d*$/;
                  if(!reg.test(value)) {
                    cb(new Error('请填写正确的预警阈值'));
                  } else if(value< 0 || value > 999999999){
                    cb(new Error('请输入0-999999999之间的整数'));
                  }else{
                    cb();
                  }
                }
            }
          }],
        },
        columns: [{
            type: 'index',
            title: '序号',
            width: 60,
            align: 'center'
          },
          {
            title: '服务名称',
            key: 'appName'
          },
          {
            title: '预警阈值',
            key: 'limit',
            render: (h, params) => {
              return h('span', params.row.limit + "次/" + superviseTypeData[params.row.limitCycle - 1].name)
            }
          },
          {
            title: '预警级别',
            key: 'level',
            render: (h, params) => {
              let level = ['一般', '较重', '严重', '特别严重']
              if (params.row.level) {
                let num = params.row.level -1
                return h('span', level[num])
              } else {
                return h('span', '-')
              }
            }
          },
          {
            title: '责任人',
            key: 'dutyUserName'
          },
          {
            title: '确认人',
            key: 'confirmUserName',
            render: (h, params) => {
              return h('div', {
                style: {
                  maxHeight: '15px',
                  overflow: 'hidden',
                  textOverflow: 'ellipsis',
                  whiteSpace: 'nowrap'
                },
                attrs: {
                  title: params.row.confirmUserName
                }
              }, params.row.confirmUserName)
            }
          },
          {
            title: '运行状态',
            key: 'runStatus',
            render: (h, params) => {
              return h('span', (params.row.runStatus) ? "开启" : "禁用")
            }
          },
          {
            title: '操作',
            key: 'act',
            width: 180,
            render: (h, params) => {
              const detail = h('Button', {
                props: {
                  type: 'primary',
                  size: 'small'
                },
                // style: {
                //  display:this.checkButton('hardware_bmyh_fwqgl_xq')?'inline-block':'none'
                // },
                on: {
                  click: () => {
                    this.modelTitle = '详情';
                    this.gotoCtrl(2, params.index)
                  }
                }
              },"详情");
              const edit = h('Button', {
                props: {
                  type: 'primary',
                  size: 'small'
                },
                // style: {
                //  display:this.checkButton('hardware_bmyh_fwqgl_xq')?'inline-block':'none'
                // },
                on: {
                  click: () => {
                    this.modelTitle = '修改';
                    this.gotoCtrl(1, params.index)
                  }
                }
              },"修改");
              const del = h('Button', {
                props: {
                  type: 'primary',
                  size: 'small'
                },
                // style: {
                //  display:this.checkButton('hardware_bmyh_fwqgl_xq')?'inline-block':'none'
                // },
                on: {
                  click: () => {
                    this.itemDelete(params.row.id,params.index);
                  }
                }
              },"删除");
              return h('div', [detail, edit, del]);
            }
          }
        ],
        data: [],
        pageOption: { //分页参数
          pageNo: 1,
          total: 0,
          pageSize: 10
        },
        sysVal: "",
      }
    },
    mounted() {
      this.init();
    },
    methods: {
      onChange (e) {
        this.$nextTick(() => {
          var num = this.createForm.limit.replace(/[^\d]/g, ''); 
          if(num !== null && num !== ''){
            num =  parseInt(num);
          }
          this.createForm.limit = num
        })
      },
      // 选择预警阈值的单位
      limitCycleChange () {
        this.$refs.createForm.validateField('limit', (err => {
          console.log(err)
        }))
      },
      init(){//初始化
        this.fwlbByAdmin()
          this.findOrgByPower();
        },
        pageChange(num) { //页码改变的回调
          if (!num) return;
          this.pageOption.pageNo = num;
          this.getServerPage();
        },
        changePageSize(num) { //切换每页条数时的回调
          if (!num) return;
          this.pageOption.pageSize = num;
          this.getServerPage();
      },
      fwlbByAdmin() {//查询我所管理的所有启用的应用或服务(1：应用；2：服务)
        api(comApiList.findAppIdListByPt, {
          type: 2
        }).then((res) => {
          if (res.status == 200 && res.data.data) {
            this.appArr = res.data.data
            this.appArr.forEach(item=>{
              this.appIds.push(item.appId)
            })
            this.serverData.appId = this.appIds
            this.getServerPage(); //查询服务分配管理分页
          }
        }, (err) => {
          //dong something...
        })
      },
        getServerPage() { // 查询分页
          this.data = [];
          var formData = {
            "data": { ...this.serverData},
            "pageNo": this.pageOption.pageNo,
            "pageSize": this.pageOption.pageSize
          };
          api(superviseApiList.getServerPage, formData).then((res) => {
            if(res.status == 200 && res.data.data) {
              this.data = res.data.data.list;
              this.pageOption.pageSize = res.data.data.pageSize;
              this.pageOption.total = res.data.data.total;
              this.pageOption.pageNo = res.data.data.pageNum;
            }
          }, (err) => {
            //dong something...
          })
        },
        findOrgByPower() {
          api(superviseApiList.findOrgByPower).then((res) => {
            if (res.status == 200 && res.data.data) {
              let ids = [];
              res.data.data.forEach(item=>{
                ids.push(String(item.id));
              })
              this.findEmpByOrgId(ids);
            }
          }, (err) => {
            //dong something...
          })
        },
        findEmpByOrgId(ids){
          api(superviseApiList.findEmpByOrgId, {orgIds: ids}).then((resp) => {
            if (resp.status == 200 && resp.data.data) {
              let data = [];
              resp.data.data.forEach(item => {
                data.push({ name: item.name, gxqptEmpId: item.gxqptEmpId });
              })
              this.personArr = data;
              // commit("GETORGBYPOWER", data);
            }
          }, (err) => {
            //dong something...
          })
        },
      search() {
        this.serverData.appId = [];
        this.pageOption.pageNo=1;
        (!this.serverData.appIds) ? this.serverData.appId = this.appIds : this.serverData.appId.push(this.serverData.appIds)
          this.getServerPage();
      },
      cancelFun () {
        // this.createForm.limit=''
        // this.createForm.limitCycle=''
        this.$refs.createForm.resetFields();
        this.createModal = false
      },
      addNew(){
          this.createModal = true;
          this.$refs.createForm.resetFields();
          this.modelTitle = '新建';
          this.modelType = '0';
        },
      saveHostRoom(){
          if(this.modelType=='2'){
            this.createModal = false;
            return;
          }
          let url = '';
          (this.modelType=='0') ? url = superviseApiList.saveServerPage : url = superviseApiList.updateServerPage;
          this.$refs['createForm'].validate((valid) => {
            if(valid) {
              this.createForm.remindType = this.remindTypes
              this.createForm.appId = this.appArr[this.createForm.arrIdx].appId;
              this.createForm.appCode = this.appArr[this.createForm.arrIdx].code;
              this.createForm.confirmUser = this.createForm.confirmUser.join(",");
              if(this.createForm.copyUser.length!=0){
                this.createForm.copyUser = this.createForm.copyUser.join(",");
              } else {
                this.createForm.copyUser = ''
              }
              this.createForm.runStatus = (this.createForm.runStatusArr=="0") ? false : true;
              api(url, this.createForm).then((res) => {
                if(res.status == 200 && res.data.data) {
                  if(res.data.data && res.data.errmsg == 'ok'){
                    console.log(this.createForm)
                    let msg = (this.modelType=='0') ? '保存成功！' : '编辑成功！';
                    this.$Message.success(msg);
                    this.getServerPage();
                    this.createModal = false;
                    // this.$refs.createForm.resetFields();
                  }else{
                    this.$Message.error(res.data.errmsg);
                    this.createModal = true;
                  }
                }else{
                  this.$Message.error(res.data.errmsg);
                  this.createModal = true;
                }
              });
            }
          })
      },
      selectChange (val) {
        console.log(val)
        if (val.length == 1) {
          this.remindTypes = val.join(",")
        } else if (val.length == 2) {
          this.remindTypes = '3'
        }
      },
        itemDelete(id,idx) {//删除
        const vm = this
        vm.$Modal.confirm({
          title: '删除确认',
          content: '确认删除吗？',
          onOk: () => {
            api(superviseApiList.deleteServerPage, {id: id}).then(res => {
              if (res.data.errcode === 0) {
                vm.$Message.success('删除成功！')
                vm.modal = false;
                vm.data.splice(idx, 1)
              }else{
                vm.$Message.error(res.data.errmsg)
              }
            }, error => {console.log(error)})
          }
        })
        },
        // 操作
        gotoCtrl (type, idx) {
          const vm = this
          vm.idx = idx;
          if(type==0){// 删除
            vm.tableList.data.splice(idx, 1)
          }else{// 修改和详情
            vm.$refs.createForm.resetFields();
            // vm.$refs.dutyUserReset.clearSingleSelect();
            vm.createForm.dutyUser = ''
            const data = JSON.parse(JSON.stringify(vm.data[idx]))
            const createForm = {...getCreateForm(), ...data};
            if (createForm.remindType == '3') {
              createForm.remindType = ["1", "2"]
            } else {
              createForm.remindType = createForm.remindType.toString();
            }
            createForm.confirmUser = createForm.confirmUser.split(",");
            if(createForm.copyUser==null) createForm.copyUser="";
            if(createForm.copyUser) createForm.copyUser = createForm.copyUser.split(",");
            createForm.runStatusArr = (createForm.runStatus) ?　'1' : '0';
            vm.appArr.forEach((item, index)=>{//整合idx
              if(item.appId==createForm.appId){
                createForm.arrIdx = index;
              }
            })
            vm.createModal = true;
            vm.createForm = createForm;
            vm.modelType = (type == 1) ? '1' : '2';
          }
        },
        checkButton(code){//按钮权限
          if(this.authButton.indexOf(code) >= 0){
            return true;
          }else{
            return false;
          }
        },
      },
      computed: {
        // 责任人列表
        dutyUsers () {
          const vm = this
          return vm.personArr.filter(item => {
            return !(vm.createForm.confirmUser.includes(item.gxqptEmpId) || vm.createForm.copyUser.includes(item.gxqptEmpId))
          })
        },
        // 确认人列表
        confirmUsers () {
          const vm = this
          return vm.personArr.filter(item => {
            return !((vm.createForm.dutyUser ? vm.createForm.dutyUser.includes(item.gxqptEmpId) : false) || vm.createForm.copyUser.includes(item.gxqptEmpId))
          })
        },
        // 抄送人列表
        copyUsers () {
          const vm = this
          return vm.personArr.filter(item => {
            return !((vm.createForm.dutyUser ? vm.createForm.dutyUser.includes(item.gxqptEmpId) : false) || vm.createForm.confirmUser.includes(item.gxqptEmpId))
          })
        },
        ...mapState([
          'authButton'
        ]),
        disa(){
          return(this.modelType=="2") ?  true : false;
        }
      },
      components:{
        hyUpload
      },
  }
</script>

<style lang='less' scoped>
  .ivu-btn-small {
    margin: 0 3px;
  }
  .handle{
    color:red;
    cursor: pointer;
  }
  .queryItem {
    width: 200px;
  }
  .access-list {
    span.handle {
      margin: 0 5px;
      display: inline-block;
      cursor: pointer;
      &:hover {
        color: #57a3f3;
      }
    }
  }
</style>
