<!-- 设备监控 -->
<template>
	<Layout>
		<Content>
			<Breadcrumb>
				<BreadcrumbItem>{{ title }}</BreadcrumbItem>
			</Breadcrumb>
			<Card>
				<Form ref="formValidate" inline :label-width="110" :model="serverData">
					<FormItem label="机房编号：">
             <Input type="text" v-model="serverData.roomNumber" style="width:140px"></Input>
					</FormItem>
          <FormItem label="管理员：">
             <Input type="text" v-model="serverData.roomAdmin" style="width:140px"></Input>
					</FormItem>
          <FormItem label="联系电话：">
             <Input type="text" v-model="serverData.contactNumber" style="width:140px"></Input>
					</FormItem>
					<FormItem label="主机数量：">
						<Input type="text" v-model="serverData.hostNum" style="width:140px"></Input>
					</FormItem>
					<FormItem :label-width="20">
						<Button type="primary" @click="search">查询</Button>
						<Button type="primary" v-if="checkButton('hardware_equipment_monitoring_xz')" @click="addNew">新增</Button>
					</FormItem>
				</Form>
				<hy-table ref="selection" :data="data" :columns="columns" :total="pageOption.total" :page-size="pageOption.pageSize" @on-change="pageChange" @on-page-size-change="changePageSize" show-sizer show-elevator/>
			</Card>
      <Modal v-model="createModal" title="机房信息" width="40%" :mask-closable="false">
        <Form ref="createForm" :model="createForm" :rules="createRule" :label-width="90">
          <FormItem label="机房编号" prop="roomNumber">
            <Input type="text" v-model="createForm.roomNumber"/>
          </FormItem>
          <FormItem label="管理员" prop="roomAdmin">
            <Input type="text" v-model="createForm.roomAdmin"/>
          </FormItem>
          <FormItem label="联系电话" :required="true" prop="contactNumber">
            <Input type="text" v-model="createForm.contactNumber"/>
          </FormItem>
          <FormItem label="主机数量" :required="true" prop="hostNum">
            <Input type="text" v-model="createForm.hostNum"/>
          </FormItem>
          <FormItem label="路由器数量" :required="true" prop="routerNum">
            <Input type="text" v-model="createForm.routerNum"/>
          </FormItem>
          <FormItem label="空调数量" :required="true" prop="airConNum">
            <Input type="text" v-model="createForm.airConNum"/>
          </FormItem>
          <FormItem label="机柜数量" :required="true" prop="cabinetNum">
            <Input type="text" v-model="createForm.cabinetNum"/>
          </FormItem>
        </Form>
        <div slot="footer">
          <Button class="modalBtn" type="primary" @click="saveHostRoom(modelType)" size="large">确定</Button>
          <Button class="modalBtn" type="default" @click="createModal = false" size="large">取消</Button>
        </div>
      </Modal>
		</Content>
	</Layout>
</template>

<script>
	import api from '@/api/axiosApi'
	import softhardApiList from '@/api/softhardApiList'
	import { mapState } from 'vuex'
	function getCreateForm() {
		return {
			roomNumber:'',
			roomAdmin:'',
			contactNumber:'',
			hostNum:'0',
			routerNum:'0',
			airConNum:'0',
			cabinetNum:'0',
		}
	}
	const validateNumber = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('不能为空！'))
        return false
      }
      const reg = /^\d+(\.\d+)?$/
      if (!reg.test(value)) {
        callback(new Error('请输入大于等于0的数字！'))
        return false
      }
    }
	export default {
		data() {
			return {
        title:this.$store.state.title,
        serverData: { //查询参数
          roomNumber:"",
          contactNumber:"",
					roomAdmin: "",
					hostNum:""
				},
				modelType:'0',//0位新增，1位修改
        createForm:{
          roomNumber:'',
          roomAdmin:'',
          contactNumber:'',
          hostNum:'0',
          routerNum:'0',
          airConNum:'0',
          cabinetNum:'0',
        },
        createModal:false,
        createRule: {
          roomNumber: [{required: true, message: '不可为空', trigger: 'blur'}],
          roomAdmin: [{required: true, message: '不可为空', trigger: 'blur'}],
          contactNumber: [{
          required: true,
          validator: (rule, value, cb) => {
	          if (!this.createForm.contactNumber) {
	            cb(new Error('不能为空'))
	          }
	          if (value && !/^1[345789]\d{9}$/.test(value)) {
	            cb(new Error('手机号格式不正确'))
	          }
	          cb()
	        }}],
          hostNum: [{ required: true, message: '请输入主机数量', trigger: 'blur' },
											{ type: 'number', message: '请输入大于等于0的数字！', trigger: 'blur',validator: validateNumber}],
          routerNum: [{ required: true, message: '请输入路由器数量', trigger: 'blur' },
											{ type: 'number', message: '请输入大于等于0的数字！', trigger: 'blur', validator: validateNumber}],
          airConNum: [{ required: true, message: '请输入空调数量', trigger: 'blur' },
											{ type: 'number', message: '请输入大于等于0的数字！', trigger: 'blur', validator: validateNumber}],
          cabinetNum: [{ required: true, message: '请输入机柜数量', trigger: 'blur' },
											{ type: 'number', message: '请输入大于等于0的数字！', trigger: 'blur', validator: validateNumber}],
        },
				columns: [{
						type: 'index',
						title: '序号',
						width: 60,
						align: 'center'
					},
					{
						title: '机房编号',
						key: 'roomNumber'
          },
					{
						title: '管理员',
						key: 'roomAdmin'
          },
          {
						title: '联系电话',
						key: 'contactNumber'
					},
					{
						title: '主机数量',
						key: 'hostNum'
          },
          {
						title: '路由器数量',
						key: 'routerNum'
					},
					{
						title: '空调数量',
						key: 'airConNum'
					},
					{
						title: '机柜数量',
						key: 'cabinetNum'
					},
					{
						title: '创建时间',
						width: 220,
						align: 'center',
						key: 'createTime'
					},
					{
						title: '操作',
						key: 'act',
						width: 300,
						render: (h, params) => {
							const look = h('Button', {
								props: {
									type: 'primary',
									size: 'small'
								},
								style: {
									display:this.checkButton('hardware_equipment_monitoring_ckbg')?'inline-block':'none'
								},
								on: {
									click: () => {//跳转到报告查看
                    this.$router.push({
                      name: 'reportView',
                      params: {params: params.row}
                    })
									}
								}
							}, '查看报告');
							const edit = h('Button', {
								props: {
									type: 'primary',
									size: 'small'
								},
								style: {
									display:this.checkButton('hardware_equipment_monitoring_xg')?'inline-block':'none'
								},
								on: {
									click: () => {
										this.gotoCtrl(1, params.index)
									}
								}
              },"修改");
              const del = h('Button', {
								props: {
									type: 'primary',
									size: 'small'
								},
								style: {
									display:this.checkButton('hardware_equipment_monitoring_sc')?'inline-block':'none'
								},
								on: {
									click: () => {
                    this.itemDelete(params.row.id,params.index);
									}
								}
							},"删除");
							return h('div', [look, edit, del]);
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
			this.emPageList(); //查询服务分配管理分页
		},
		methods: {
			handleChange(value, selecteddata) {
				if(selecteddata.length>0){
					let val = selecteddata.map(o => o.value).reverse();
					this.serverData.system = val[0];
				}else{
					this.serverData.system = "";
				}
			},
			pageChange(num) { //页码改变的回调
				this.pageOption.pageNo = num;
				this.emPageList();
			},
			changePageSize(num) { //切换每页条数时的回调
				this.pageOption.pageSize = num;
				this.emPageList();
			},
			emPageList() { // 查询服务分配管理分页
				this.data = [];
				var formData = {
					"data": { ...this.serverData
					}, //Object.assign({}, this.searchCondition),
					"pageNo": this.pageOption.pageNo,
					"pageSize": this.pageOption.pageSize
				};
				api(softhardApiList.emPageList, formData).then((res) => {
					if(res.status == 200 && res.data.data) {
						this.data = res.data.data.list;
						if(this.data.length>0){
							this.pageOption.pageSize = res.data.data.pageSize;
							this.pageOption.total = res.data.data.total;
							this.pageOption.pageNo = res.data.data.pageNum;
							this.data.forEach(item=>{//时间格式化
								item.createTime = item.createTime.replace("-","年");
								item.createTime = item.createTime.replace("-","月");
								item.createTime = item.createTime.replace(" ","日 ");
							})
						};
					}
				}, (err) => {
					//dong something...
				})
			},
			search() {
				this.emPageList();
      },
      addNew(){
				this.createModal = true;
				this.modelType = '0';
				this.$refs.createForm.resetFields();
      },
      saveHostRoom(){
				let url = '';
				(this.modelType=='0') ? url = softhardApiList.saveEquipmentMonitor : url = softhardApiList.updateEquipmentMonitor;
				this.createForm.createTime = null;
				console.log(this.createForm);
				this.$refs['createForm'].validate((valid) => {
					if(valid) {
						api(url, this.createForm).then((res) => {
							if(res.status == 200 && res.data.data) {
								if(res.data.data && res.data.errmsg == 'ok'){
									let msg = (this.modelType=='0') ? '保存成功！' : '编辑成功！';
									this.$Message.success(msg);
									this.emPageList();
								}else{
									this.$Message.error(res.data.errmsg);
								}
							}else{
								this.$Message.error(res.data.errmsg);
							}
						});
						this.createModal = false;
					}
				})
			},
      itemDelete(id,idx) {//删除
        const vm = this
        vm.$Modal.confirm({
          title: '删除确认',
          content: '确认删除吗？',
          onOk: () => {
            api(softhardApiList.deleteEquipmentMonitor, {id: id}).then(res => {
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
				this.idx = idx;
				// 删除
				if (type === 0) {
					this.tableList.data.splice(idx, 1)
				} else {
				// 修改
					const data = JSON.parse(JSON.stringify(this.data[idx]))
					this.createForm = {...getCreateForm(), ...data};
					this.createForm.airConNum = this.createForm.airConNum.toString();
					this.createForm.cabinetNum = this.createForm.cabinetNum.toString();
					this.createForm.hostNum = this.createForm.hostNum.toString();
					this.createForm.routerNum = this.createForm.routerNum.toString();
					console.log(this.createForm);
					this.createModal = true;
					this.modelType = '1';
				}
			},
			checkButton(code){
				if(this.authButton.indexOf(code) >= 0){
					return true;
				}else{
					return false;
				}
			}
		},
		computed: {
			...mapState([
				'authButton'
			])
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
