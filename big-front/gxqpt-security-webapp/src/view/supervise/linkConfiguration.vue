<!-- 环节配置 -->
<template>
	<Layout>
		<Content>
			<Breadcrumb>
				<BreadcrumbItem>权责运行监管配置中心</BreadcrumbItem>
				<BreadcrumbItem>权责名称：{{ title }}</BreadcrumbItem>
			</Breadcrumb>
			<Card style="min-height: 600px;">
				<Form ref="formValidate" inline :label-width="110" :model="serverData">
					<FormItem label="环节名称：">
						<Input type="text" v-model="serverData.name" style="width:140px" clearable></Input>
					</FormItem>
          <FormItem label="环节编码：">
             <Input type="text" v-model="serverData.code" style="width:140px" clearable></Input>
					</FormItem>
					<FormItem :label-width="20">
						<Button type="primary" @click="search">查询</Button>
						<Button type="primary" @click="addNew">新增</Button>
						<Button type="primary" @click="goback">返回</Button>
					</FormItem>
				</Form>
				<hy-table ref="dragable" :data="data" :columns="columns" :total="pageOption.total" :page-size="pageOption.pageSize" @on-change="pageChange" @on-page-size-change="changePageSize" show-sizer border dragable/>
			</Card>
      <Modal v-model="createModal" :title="modelTitle" width="40%" :mask-closable="false">
        <Form ref="createForm" :model="createForm" :rules="createRule" :label-width="90">
					<FormItem label="环节名称" prop="name">
						<Input type="text" v-model="createForm.name" :disabled="disa"/>
          </FormItem>
          <FormItem label="环节编码" prop="code">
            <Input type="text" v-model="createForm.code" :disabled="disa"/>
          </FormItem>
          <FormItem label="预警级别" required>
            <Select v-model="createForm.level"  filterable :disabled="disa">
                <Option :value="1" >一般</Option>
                <Option :value="2" >较重</Option>
                <Option :value="3" >严重</Option>
                <Option :value="4" >特别严重</Option>
            </Select>
          </FormItem>
          <FormItem label="责任人" prop="dutyUser">
						<Select v-model="createForm.dutyUser" ref="dutyUser" clearable filterable :disabled="disa">
                <Option v-for="item in dutyUsers" :value="item.gxqptEmpId" :key="item.gxqptEmpId">{{ item.name }}</Option>
            </Select>
          </FormItem>
          <FormItem label="确认人" prop="confirmUser">
						<Select v-model="createForm.confirmUser" filterable multiple :disabled="disa">
                <Option v-for="item in confirmUsers" :value="item.gxqptEmpId" :key="item.gxqptEmpId">{{ item.name }}</Option>
            </Select>
          </FormItem>
          <FormItem label="抄送人" prop="copyUser">
						<Select v-model="createForm.copyUser" filterable multiple :disabled="disa">
                <Option v-for="item in copyUsers" :value="item.gxqptEmpId" :key="item.gxqptEmpId">{{ item.name }}</Option>
            </Select>
          </FormItem>
					<FormItem label="时长阈值" prop="timeLimit">
						<Input type="text" v-model="createForm.timeLimit" :disabled="disa" @on-change="onChange" :maxlength="3" style="width: calc(100% - 130px); float: left; margin-right: 10px;">
							<span slot="append">h</span>
						</Input>
						<span class="fontTooltip">最大值120</span>
          </FormItem>
          <FormItem label="提醒方式" prop="remindType">
						<Select class="queryItem"  :clearable="!disa" multiple v-model="createForm.remindType" @on-change="selectChange" :disabled="disa">
							<Option value="1">消息</Option>
							<Option value="2">短信</Option>
						</Select>
          </FormItem>
					<FormItem label="环节备注" prop="desc">
						<Input type="textarea" v-model="createForm.desc" :disabled="disa"/>
          </FormItem>
          <!-- <FormItem label="附件报告" style="margin-top: 40px;" prop="file">
            <hy-upload ref="evalReport" multiple :on-success="setEvalReport" :on-remove="removeEvalReport" :before-upload="evalBeforeUpload" :defaultFileList="createForm.attachment"></hy-upload>
          </FormItem> -->
        </Form>
				<div slot="footer" v-if="modelType!='2'">
          <Button class="modalBtn" type="primary" @click="saveHostRoom" size="large">确定</Button>
          <Button class="modalBtn" type="default" @click="createModal = false" size="large">取消</Button>
        </div>
				<div slot="footer" v-else>
          <Button class="modalBtn" type="default" @click="createModal = false" size="large">关闭</Button>
        </div>
      </Modal>
		</Content>
	</Layout>
</template>

<script>
	import api from '@/api/axiosApi'
	import superviseApiList from '@/api/superviseApiList'
	import hyUpload from '@/components/hengyun/hyUpload'
	import { validNumber } from '@/api/formValidate'
	import { mapState } from 'vuex'
	function getCreateForm() {
		return {
			name:'',//环节名称
			code:'',//环节编码
			level: 1,//预警级别
			timeLimit:'',//时长阈值
			dutyUser:'',//责任人
			confirmUser:'',//确认人
			copyUser:'',//抄送人
			remindType:[],//提醒方式
			desc:'',//备注
			sortNum:'',//排序编号
			powerSuperviseId:''//上一级内容id
		}
	}
	const superviseTypeData = [{name:"实时",id:1},{name:"天",id:2},{name:"周",id:3},{name:"月",id:4}];
	export default {
		data() {
			const vm = this
			return {
        title:'',
				remindTypes: '',
        serverData: { //查询参数
					name:"",//环节名称
					code:"",//环节编码
				},
				modelTitle:'新建',
				modelType:'0',//0新增，1修改，2详情
				appArr:this.$store.state.myServerData,//应用列表
				superviseTypeData:superviseTypeData,
				personArr:[],//人员列表
        createForm:{
          name:'',//环节名称
					code:'',//环节编码
					level: 1,//预警级别
					timeLimit:'',//时长阈值
					dutyUser:'',//责任人
					confirmUser:'',//确认人
					copyUser:null,//抄送人
					remindType:[],//提醒方式
					desc:'',//备注
					sortNum:'',//排序编号
					powerSuperviseId:''//上一级内容id
        },
        createModal:false,
        createRule: {
					name: [{
            required: true,
            trigger: 'blur',
            validator: (rule, value, cb) => {
							console.log(value.length);
              if (value=='') {
                cb(new Error('请输入环节名称！'))
                return
              }else if(value.length <0 || value.length>64){
								cb(new Error('请输入1-64字符环节名称！'))
                return
							}
              cb()
            }
          }],
					code: [
						{required: true, trigger: 'blur', 
							validator: (rule, value, cb) => {
								if (value=='') {
									cb(new Error('环节编码不能为空！'))
									return
								}else if(value.length <0 || value.length>64){
									cb(new Error('请输入1-64字符的环节编码！'))
									return
								}
								cb()
							}
						}
					],
					interfaceUrl: [{required: true, message: '接口地址不能为空', trigger: 'blur'}],
					dutyUser: [{required: true, message: '责任人不能为空', trigger: 'change'}],
					confirmUser: [{
            required: true,
            trigger: 'change',
            validator: (rule, value, cb) => {
              if (this.createForm.confirmUser.length==0) {
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
          timeLimit: [
						{required: true, trigger: 'change',validator: (rule, value, callback) => {
							let reg = /^\d+(\.\d+)?$/;
							if(!reg.test(value)) {
								callback(new Error("时长阈值不能为空"));
							} else if(value<0 || value > 120){
								rule.message = "请输入0-120之间的整数";
								callback(new Error(rule.message));
							}else{
								callback();
							}
						}}
					],
					desc: [{required: false,validator: (rule, value, cb) => {
              if (value.length>500) {
                cb(new Error('请输入0-500字符的环节备注！'))
                return
              }
              cb()
            }}],
        },
				columns: [{
						type: 'index',
						title: '序号',
						width: 60,
						align: 'center'
					},
					{
						title: '环节编码',
						key: 'code'
          },
					{
						title: '环节名称',
						key: 'name'
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
						title: '预警阈值',
						key: 'timeLimit',
						render: (h, params) => {
              return h('span', params.row.timeLimit + "/h")
            }
					},
          {
						title: '提醒方式',
						key: 'runStatus',
						render: (h, params) => {
							let runStatus = ''
							if (params.row.remindType=='1') {
								runStatus = '消息'
							} else if (params.row.remindType=='2') {
								runStatus = '短信'
							} else if (params.row.remindType=='3') {
								runStatus = '消息、短信'
							}
              return h('span', runStatus)
            }
					},
					{
						title: '操作',
						key: 'act',
						width: 300,
						align: 'center',
						render: (h, params) => {
							const detail = h('Button', {
								props: {
									type: 'primary',
									size: 'small'
								},
								// style: {
								// 	display:this.checkButton('hardware_bmyh_fwqgl_xq')?'inline-block':'none'
								// },
								on: {
									click: () => {
										this.modelTitle = '详情';
										this.$refs.createForm.resetFields();
										this.gotoCtrl(2, params.index)
									}
								}
              },"详情");
							const edit = h('Button', {
								props: {
									type: 'primary',
									size: 'small'
								},
								on: {
									click: () => {
										this.modelTitle = '修改';
										this.$refs.createForm.resetFields();
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
								// 	display:this.checkButton('hardware_bmyh_fwqgl_xq')?'inline-block':'none'
								// },
								on: {
									click: () => {
										this.itemDelete(params.row.id,params.index);
									}
								}
							},"删除");
							const up = h('Button', {
								props: {
									type: 'primary',
									size: 'small',
									icon: 'md-arrow-dropup'
								},
								on: {
									click: () => {
										this.endTable(params.row, 0)
									}
								}
							},"上移");
							const down = h('Button', {
								props: {
									type: 'primary',
									size: 'small',
									icon: 'md-arrow-dropdown'
								},
								on: {
									click: () => {
										this.endTable(params.row, 1)
									}
								}
							},"下移");
							const btns = []
							btns.push(detail)
							btns.push(edit)
							btns.push(del)
							if (params.row.sortNum !== 1) {
								btns.push(up)
							}
							let totalNum = (vm.pageOption.pageNo -1) * vm.pageOption.pageSize + (params.index + 1)
							if (totalNum !== vm.pageOption.total) {
								btns.push(down)
							}
							console.log(totalNum)
							return h('div', btns);
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
					var num = this.createForm.timeLimit
					.replace(/[^\d]/g, '');
					if(num !== null && num !== ''){
						num =  parseInt(num);
					}
					this.createForm.timeLimit = num
				})
			},
			init(){//初始化
				this.title = this.$route.params.params.name
				this.getAccrualLinkPage(); //查询服务分配管理分页
				this.findOrgByPower();
			},
			pageChange(num) { //页码改变的回调
				this.pageOption.pageNo = num;
				this.getAccrualLinkPage();
			},
			changePageSize(num) { //切换每页条数时的回调
				this.pageOption.pageSize = num;
				this.getAccrualLinkPage();
			},
			getAccrualLinkPage() { // 查询分页
				this.data = [];
				var formData = {
					"data": { ...this.serverData,
						powerSuperviseId:this.$route.params.params.id
					},
					"pageNo": this.pageOption.pageNo,
					"pageSize": this.pageOption.pageSize
				};
				api(superviseApiList.getAccrualLinkPage, formData).then((res) => {
					if(res.status == 200 && res.data.data) {
						this.data = res.data.data.list;
						if(this.data.length>0){
							this.pageOption.pageSize = res.data.data.pageSize;
							this.pageOption.total = res.data.data.total;
							this.pageOption.pageNo = Number(res.data.data.pageNum);
						};
					}
				}, (err) => {
					//dong something...
				})
			},
			findOrgByPower() {
				api(superviseApiList.findOrgByPower).then((res) => {
					if (res.status === 200 && res.data.data) {
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
						console.log(this.personArr);
						// commit("GETORGBYPOWER", data);
					}
				}, (err) => {
					//dong something...
				})
			},
			search() {
				console.log(this.serverData);
				this.getAccrualLinkPage();
      },
      addNew(){
				this.modelType = '0';
				this.modelTitle = '新建';
				this.createModal = true;
				this.$refs.createForm.resetFields();
				this.$refs.dutyUser.clearSingleSelect();				
				this.createForm.sortNum = parseInt(this.pageOption.total+1);
			},
			goback(){
				this.$router.go(-1);
			},
			endTable (rowData, moveType) {//列表单元拖放结束后的回调函数
				// let data = { // 拖拽排序 配合on-end
				// 	firstId: String(JSON.parse(JSON.stringify(this.data[e.from])).id),
				// 	firstSortNum: e.to+1,
				// 	secondId: String(JSON.parse(JSON.stringify(this.data[e.to])).id),
				// 	secondSortNum: e.from+1
				// };
				let data = {}
				if (moveType === 0) {
					data = {
						firstId: String(JSON.parse(JSON.stringify(this.data[rowData.sortNum - 1])).id),
						firstSortNum: rowData.sortNum - 1,
						secondId: String(JSON.parse(JSON.stringify(this.data[rowData.sortNum - 2])).id),
						secondSortNum: rowData.sortNum
					};
				} else {
					data = {
						firstId: String(JSON.parse(JSON.stringify(this.data[rowData.sortNum - 1])).id),
						firstSortNum: rowData.sortNum + 1,
						secondId: String(JSON.parse(JSON.stringify(this.data[rowData.sortNum])).id),
						secondSortNum: rowData.sortNum
					};
				}
				this.updateSortAccrualLinkPage(data)
			},
			updateSortAccrualLinkPage(data){
				api(superviseApiList.updateSortAccrualLinkPage, data).then((res) => {
					if(res.status == 200 && res.data.data) {
						if(res.data.data && res.data.errmsg == 'ok'){
							let msg = '更新排序成功！';
							this.$Message.success(msg);
							this.getAccrualLinkPage();
						}else{
							this.$Message.error(res.data.errmsg);
						}
					}else{
						this.$Message.error(res.data.errmsg);
					}
				});
			},
			selectChange (val) {
				if (val.length == 1) {
					this.remindTypes = val.join(",")
				} else if (val.length == 2) {
					this.remindTypes = '3'
				}
			},
      saveHostRoom(){
        const vm = this
				if(vm.modelType=='2'){
					vm.createModal = false;
					return;
				}
				if(vm.modelType=='0'){
				  vm.createForm.id = ''
				}
				let url = '';
				(vm.modelType=='0') ? url = superviseApiList.saveAccrualLinkPage : url = superviseApiList.updateAccrualLinkPage;
				vm.$refs['createForm'].validate((valid) => {
					if(valid) {
					  let newdata = JSON.parse(JSON.stringify(vm.createForm))
					  console.log(vm.createForm);
					  newdata.confirmUser = newdata.confirmUser.join(",")
					  newdata.copyUser = newdata.copyUser == ''? '' : newdata.copyUser.join(',')
						newdata.powerSuperviseId = vm.$route.params.id;
						newdata.runStatus = (newdata.runStatusArr=="0") ? false : true;
						newdata.remindType = newdata.remindType.join('')
						if(newdata.remindType.length == 2){
						  newdata.remindType = 3
						}
						console.log(newdata)
						api(url, newdata).then((res) => {
							if(res.status == 200 && res.data.data) {
								if(res.data.data && res.data.errmsg == 'ok'){
									let msg = (vm.modelType=='0') ? '保存成功！' : '编辑成功！';
									vm.$Message.success(msg);
									vm.getAccrualLinkPage();
									vm.createModal = false;
									newdata = {}
								}else{
									vm.$Message.error(res.data.errmsg);
									vm.createModal = true
								}
							}else{
								vm.$Message.error(res.data.errmsg);
								vm.createModal = true
							}
						});
					}else{
						console.log(this.createForm);
					}
				})
      },
			itemDelete(id,idx) {//删除
        const vm = this
        vm.$Modal.confirm({
          title: '删除确认',
          content: '确认删除吗？',
          onOk: () => {
            api(superviseApiList.deleteAccrualLinkPage, {id: id}).then(res => {
              if (res.data.errcode === 0) {
                vm.$Message.success('删除成功！')
								vm.modal = false;
								vm.data.splice(idx, 1);
								this.getAccrualLinkPage();
              }else{
								vm.$Message.error(res.data.errmsg)
							}
            }, error => {console.log(error)})
          }
        })
			},
			// 操作
			gotoCtrl (type, idx, sortNum) {//type 0：删除，1:修改，2：详情，3：更改排序
			  //this.$refs.dutyUser.clearSingleSelect();
				this.idx = idx;
				if(type==0){// 删除
					this.tableList.data.splice(idx, 1)
				}else{// 修改和详情
					const data = JSON.parse(JSON.stringify(this.data[idx]))
					this.createForm = {...getCreateForm(), ...data};
					if (this.createForm.remindType == '3') {
						this.createForm.remindType = ["1", "2"]
					} else {
						this.createForm.remindType = this.createForm.remindType.toString();
					}
					console.log(this.createForm.remindType)
					this.createForm.confirmUser = this.createForm.confirmUser.split(",");
					if(this.createForm.copyUser==null)this.createForm.copyUser="";
					if(this.createForm.copyUser)this.createForm.copyUser = this.createForm.copyUser.split(",");
					this.createForm.runStatusArr = (this.createForm.runStatus) ?　'1' : '0';
					switch (type) {
						case 1:
							this.modelType='1';
							this.createModal = true;
							break;
						case 2:
							this.modelType='2';
							this.createModal = true;
							break;
						case 3:
							this.createForm.sortNum = sortNum;
							this.updateSortAccrualLinkPage();
							break;
						default:
							break;
					}
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
            return !((vm.createForm.confirmUser ? vm.createForm.confirmUser.includes(item.gxqptEmpId) : false) || (vm.createForm.copyUser ? vm.createForm.copyUser.includes(item.gxqptEmpId) : false))
          })
        },
        // 确认人列表
        confirmUsers () {
          const vm = this
          return vm.personArr.filter(item => {
            return !((vm.createForm.dutyUser ? vm.createForm.dutyUser.includes(item.gxqptEmpId) : false) || (vm.createForm.copyUser ? vm.createForm.copyUser.includes(item.gxqptEmpId) : false))
          })
        },
        // 抄送人列表
        copyUsers () {
          const vm = this
          return vm.personArr.filter(item => {
            return !((vm.createForm.dutyUser ? vm.createForm.dutyUser.includes(item.gxqptEmpId) : false) || (vm.createForm.confirmUser ? vm.createForm.confirmUser.includes(item.gxqptEmpId) : false))
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
	.hy-table {
    cursor: move;
  }
	.fontTooltip{
	  font-size: 12px;
	  color: #bbb;
	}
</style>
