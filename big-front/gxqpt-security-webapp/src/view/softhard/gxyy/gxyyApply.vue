<template>
	<Layout>
		<Content>
			<Breadcrumb>
				<!-- <BreadcrumbItem>高新翼云</BreadcrumbItem> -->
				<BreadcrumbItem>{{title}}</BreadcrumbItem>
			</Breadcrumb>
			<Card>
				<Form ref="formValidate" inline :label-width="100" :model="searchCondition">
					<FormItem label="申请单位：">
						<Input class="queryItem" v-model="searchCondition.applyOrgname" type="text">
						</Input>
					</FormItem>
					<FormItem label="申请人：">
						<Input class="queryItem" v-model="searchCondition.applyUname" type="text">
						</Input>
					</FormItem>
					<FormItem label="申请类别：">
						<Select class="queryItem" clearable v-model="searchCondition.applyType" >
							<Option v-for="item in categoryList" :value="item.value" :key="item.value">{{item.label}}</Option>
						</Select>
					</FormItem>
					<FormItem label="申请时间：">
						<DatePicker class="queryItem" format="yyyy-MM-dd" type="daterange" style="width: 200px" @on-change="changeTime">
						</DatePicker>
					</FormItem>
					<FormItem :label-width="20">
						<Button type="primary" @click="handleSubmit">查询</Button>
						<!--<Button type="primary" @click="handleSubmit">刷新</Button>-->
					</FormItem>
				</Form>
				<hy-table ref="selection" :data="data" :columns="columns" :current="pageOption.current" :total="pageOption.total" :page-size="pageOption.pageSize" @on-change="pageChange" @on-page-size-change="changePageSize" show-sizer show-elevator/>
			</Card>
		</Content>
	</Layout>
</template>

<script>
	import api from '@/api/axiosApi'
	import softhardApiList from '@/api/softhardApiList'
	import { mapState } from 'vuex'
	export default {
		data() {
			return {
				title:this.$store.state.title,
				searchCondition: {
					applyOrgname: '',
					applyUname: '',
					applyType: '',
					applyStartTime: '',
					applyEneTime: '',
					scode: 'YY',
				},
				columns: [{
						type: 'index',
						title: '序号',
						width: 60,
						align: 'center'
					},
					{
						title: '申请单号',
						key: 'applyNo'
					},
					{
						title: '申请单位',
						key: 'applyOrgname'
					},
					{
						title: '申请人',
						key: 'applyUname'
					},
					{
						title: '申请类别',
						key: 'applyType',
						render: (h, params) => {
							switch(params.row.applyType) {
								case '1':
									return h('div', "新增");
								case '2':
									return h('div', "变更");
								default:
									return h('span', '--');
							}
						}
					},
					{
						title: '申请时间',
						key: 'applyTime'
					},
					{
						title: '申请状态',
						key: 'sname'
					},
					{
						title: '处理环节',
						key: 'stepName'
					},
					{
						title: '操作',
						key: 'act',
						width: 200,
						render: (h, params) => {
							const vm = this;
							const handleType = params.row.handleType;
							const detail = h('Button', {
								props: {
									type: 'primary',
									size: 'small'
								},
								style: {
									display:this.checkButton('hardware_gxyy_sqgl_xq')?'inline-block':'none'
								},
								on: {
									click: () => {
										let keyid = params.row.id;
										vm.$router.push({
											path: '/engineView/' + keyid
										});
									}
								}
							}, '详情');
							const check = h('Button', {
								props: {
									type: 'primary',
									size: 'small'
								},
								style: {
									display:this.checkButton('hardware_gxyy_sqgl_cl')?'inline-block':'none'
								},
								on: {
									click: () => {
										let keyid = params.row.id;
										vm.$router.push({
											path: '/engineDeal/' + keyid
										});
									}
								}
							}, '处理');
							const create = h('Button', {
								props: {
									type: 'primary',
									size: 'small'
								},
								style: {
									display:this.checkButton('hardware_gxyy_sqgl_zjzj')?'inline-block':'none'
								},
								on: {
									click: () => {
										let keyid = params.row.id;
										vm.$router.push({
											path: '/createEngine/' + keyid
										});
									}
								}
							}, '新增主机');
							switch(handleType) {
								case '1':
									return h('div', [detail]);
								case '2':
									if(params.row.stepCode=="YIYUN_ADD_HOST"){
										return h('div', [create,detail]);
									}else{
										return h('div', [check,detail])
									};
							}
						}
					}
				],
				data: [{
					id:"10001",
					applyNo:"1",
				}],
				pageOption: { //分页参数
					current: 1,
					total: 0,
					pageSize: 10
				},
				categoryList: [{
					value: 1,
					label: '新增'
				}, {
					value: 2,
					label: '变更'
				}],
				typeList: [{
					value: 2,
					label: '类型1'
				}],
				providerList: [{
					value: 3,
					label: '提供方1'
				}]
			}
		},
		mounted() {
			this.getApprApplyList(); //审批列表数据查询
		},
		methods: {
			changeTime(val) { //申请时间选择
				if(val[0] && val[1]) {
					var startTime = val[0].replace(/\//g, "-");
					var endTime = val[1].replace(/\//g, "-");
					this.searchCondition.applyStartTime = startTime;
					this.searchCondition.applyEneTime = endTime;
				} else {
					this.searchCondition.applyStartTime = "";
					this.searchCondition.applyEneTime = "";
				}
			},
			pageChange(num) { //页码改变的回调
				this.pageOption.current = num;
				this.getApprApplyList();
			},
			changePageSize(num) { //切换每页条数时的回调
				this.pageOption.pageSize = num;
				this.getApprApplyList();
			},
			getApprApplyList() { //审批列表数据查询
				this.data = [];
				this.showFlow = false;
				let searchString = (JSON.stringify(this.searchCondition)).replace(/\ +/g, "");
				let searchJson = JSON.parse(searchString);
				var formData = {
					"data": searchJson, //Object.assign({}, this.formQueryData),
					"pageNo": this.pageOption.current,
					"pageSize": this.pageOption.pageSize
				};
				api(softhardApiList.getApprApplyList, formData).then((res) => {
					if(res.status == 200 && res.data.data) {
						this.data = res.data.data.list;
						console.log(this.data);
						if(this.data.length > 0) {
							this.pageOption.pageSize = res.data.data.pageSize;
							this.pageOption.total = res.data.data.total;
							this.pageOption.current = res.data.data.pageNum;
							this.data.forEach(item=>{
								item.handleName = (item.handleType==1) ? '已处理' : '未处理';//修改申请状态的名称标志
								item.applyTime = item.applyTime.replace("-","年");
								item.applyTime = item.applyTime.replace("-","月");
								item.applyTime = item.applyTime.replace(" ","日 ");
							});
						};
					}
				}, (err) => {
					//dong something...
				})
			},
			// 点击查询按钮
			handleSubmit() {
				this.pageOption.current=1;
				this.getApprApplyList();
			},
			// 从服务端获取列表数据
			getList(pageNo, pageSize) {
				console.log(pageNo)
				console.log(pageSize)
			},
			// 操作栏点击
			gotoCtrl(idx) {
				alert(['详情', '审核'][idx])
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

<style lang='less'>
	.queryItem {
		width: 200px;
	}
	.department-apply {
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