<template>
		<Layout>
			<Content>
				<Breadcrumb>
					<BreadcrumbItem>统一监管平台</BreadcrumbItem>
					<BreadcrumbItem>预警处理结果台账</BreadcrumbItem>
				</Breadcrumb>
				<Card style="min-height: 600px;">
					<Form ref="formValidate" inline :label-width="110">
						<FormItem label="预警级别：" :label-width="72">
							<Select class="queryItem" v-model="formQueryData.level">
								<Option value="5">全部</Option>
								<Option value="4">特别严重</Option>
								<Option value="3">严重</Option>
								<Option value="2">较重</Option>
								<Option value="1">一般</Option>
							</Select>
						</FormItem>
						<FormItem label="预警类型：">
							<Input class="queryItem" type="text" v-model="warnTypeName" @on-focus="showWarnTypeModal"></Input>
							<Modal v-model="warnTypeModal" title="预警类型选择" width="60%" :mask-closable="false">
								<Tree class="iviewTree" :data="warnType" @on-select-change="getWarnTypeId"></Tree>
								<div slot="footer"></div>
							</Modal>
						</FormItem>
						<FormItem label="预警标题：">
							<Input class="queryItem" type="text" v-model="formQueryData.title"> </Input>
						</FormItem>
						<FormItem label="应用程序名称：">
							<Input class="queryItem" type="text" v-model="formQueryData.appName"> </Input>
						</FormItem>
						<FormItem label="预警时间：" :label-width="72">
							<DatePicker class="queryItem" @on-change="changeTime" format="yyyy/MM/dd" type="daterange" placeholder="请选择日期"></DatePicker>
						</FormItem>
						<Button v-if="checkButton('supervise_hangle_result_ledger_select')" type="primary" @click="searchfun">查询</Button>
					</Form>
					<hy-table ref="selection" :data="data" :columns="columns" :total="pageOption.total" :page-size="pageOption.pageSize" :current="pageOption.current" @on-change="pageChange" @on-page-size-change="changePageSize" @on-row-click="rowClick" show-total show-sizer border/>
					<Steps v-if="showFlow" class="flow" :current="currentFlow">
						<Step v-for="flowRow in flowRowsData" :key="flowRow.title" :title="flowRow.title" :content="flowRow.content"></Step>
					</Steps>
				</Card>
			</Content>
			<Modal v-model="modal" title="详情" width="60%" :mask-closable="false">
				<ewView :dataView="dataView"></ewView>
				<div slot="footer">
					<Button type="text" @click="closeModal">关闭</Button>
				</div>
			</Modal>
		</Layout>
</template>

<script>
	import ewView from './earlyWarningView'
	import api from '@/api/axiosApi'
	import superviseApiList from '@/api/superviseApiList'
	import hyTable from '@/components/hengyun/table'
	import { mapState } from 'vuex'
	export default {
		data() {
			return {
				modal: false,
				showFlow: false,
				warnTypeModal: false,
				warnTypeName: "",
				currentFlow: 1, //当前流程环节
				flowRowsData: [], //流程数据
				formQueryData: { //查询参数
					level: "",
					typeId: null,
					title: "",
					appName: "",
					warnStartTime: '',
					warnEndTime: '',
				},
				pageOption: { //分页参数
					current: 1,
					total: 0,
					pageSize: 10
				},
				warnType: [], //预警类型数据
				columns: [{
						title: '序号',
						width: 60,
						align: 'center',
						render: (h, params) => {
							return h('span', params.index + (this.pageOption.current - 1) * this.pageOption.pageSize + 1);
						}
					},
					{
						title: '预警级别',
						key: 'level',
						render: (h, params) => {
							var levelText;
							switch(params.row.level) {
								case 1:
									levelText = "一般";
									break;
								case 2:
									levelText = "较重";
									break;
								case 3:
									levelText = "严重";
									break;
								case 4:
									levelText = "特别严重";
									break;
								default:
									levelText = "";
									break;
							};
							return h('span', levelText);
						}
					},
					{
						title: '预警类型',
						key: 'type',
					},
					{
						title: '预警标题',
						key: 'title'
					},
					{
						title: '应用程序名称',
						key: 'appName',
					},
					{
						title: '责任人',
						key: 'recName',
						render: (h, params) => {
							if(params.row.recType == "3") {
								return h('span', params.row.recName);
							} else {
								return h('span', "待定");
							};
						}
					},
					{
						title: '责任人处理信息',
						key: 'remarks',
						width: 110
					},
					{
						title: '预警确认时长(h)',
						key: 'confirmCostTime',
						width: 120,
					},
					{
						title: '预警处理时长(h)',
						key: 'handleCostTime',
						width: 120,
						render: (h, params) => {
							if(params.row.recType != 3) {
								return h('span', "待定");
							} else {
								return h('span', params.row.handleCostTime);
							};
						}
					},
					{
						title: '预警时间',
						key: 'warntime',
						width: 130,
					},
					{
						title: '操作',
						key: 'act',
						width: 90,
						align: 'center',
						render: (h, params) => {
							return h('div', [
								h('Button', {
									props: {
										type: 'primary',
										size: 'small'
									},
									style: {
										display: this.checkButton('supervise_hangle_result_detail') ? 'inline-block' : 'none'
									},
									on: {
										click: () => {
											var warnId = params.row.id;
											this.getByWarnId(warnId);
										}
									}
								}, '详情')
							]);
						}
					},
				],
				data: [], //列表数据
				dataView: { //详情数据
					dutyList: [{
						app_name: "",
						biz_type: "",
						name_: "",
					}],
					warn: [{
						title: "",
						type: "",
						warntime: "",
						status: "",
						content: "",
						level: "",
						hanTime: "",
					}],
					sendeeList: [{
						recName: "",
						recType: "",
						recTime: "",
						remarks: "",
					}]
				},
			}
		},
		components: {
			ewView,
			hyTable,
		},
		mounted() {
			this.getWarnType(); //获取预警类型信息
			this.getWarnHandleResultLedgerList(); //获取预警处理信息列表
		},
		methods: {
			showWarnTypeModal() {
				this.warnTypeModal = true;
			},
			changeTime(val) { //获取选中时间段
				if(val[0] && val[1]) {
					var startTime = val[0].replace(/\//g, "-");
					var endTime = val[1].replace(/\//g, "-");
					this.formQueryData.warnStartTime = startTime + " 00:00:00";
					this.formQueryData.warnEndTime = endTime + " 23:59:59";
				} else {
					this.formQueryData.warnStartTime = "";
					this.formQueryData.warnEndTime = "";
				}
			},
			getWarnType() { //获取预警类型信息
				api(superviseApiList.findWarnTypeTree, {
					id: "2012"
				}).then((res) => {
					if(res.status == 200 && res.data.data) {
						let warnTypeObj = res.data.data;
						let warnArray = [{
							id:null,
							title:"全部",
							expand: true,
							children:this.builderTree(warnTypeObj)
						}];
						this.warnType = warnArray;
					}
				}, (err) => {
					//dong something...
				})
			},
			getWarnTypeId(data){//获取选中的预警类型id
				this.formQueryData.typeId = data[0].id;
				this.warnTypeName = data[0].title;
				this.warnTypeModal = false;
			},
			builderTree(r) {
				let _this = this;
				if(!r || r.length == 0) {
					return;
				}
				let arrayFirst = [];
				r.forEach(function(value, index) {
					let obj = {},isChildOrg = true;
					obj.id = value.id;
					obj.title = value.typeName;
					if(!value.children || value.children.length == 0) {
						obj.children = [];
						isChildOrg = false;
					};

					if(isChildOrg) {
						obj.children = _this.builderTree(value.children);
					};
					arrayFirst.push(obj);
					return;
				});
				return arrayFirst;
			},
			searchfun(){
				this.pageOption.current=1;
				this.getWarnHandleResultLedgerList()
			},
			getWarnHandleResultLedgerList() { //获取预警处理信息列表
				this.data = [];
				let searchString = (JSON.stringify(this.formQueryData)).replace(/\ +|\\t/gm,'');
				let searchJson = JSON.parse(searchString);
				var formData = {
					"data": searchJson, //Object.assign({}, this.formQueryData),
					"pageNo": this.pageOption.current,
					"pageSize": this.pageOption.pageSize
				};
				api(superviseApiList.getWarnHandleResultLedgerList, formData).then((res) => {
					if(res.status == 200 && res.data.data) {
						this.data = res.data.data.list;
						if(this.data.length > 0) {
							this.pageOption.pageSize = res.data.data.pageSize;
							this.pageOption.total = res.data.data.total;
							this.pageOption.current = res.data.data.pageNum;
						};
					}
				}, (err) => {
					//dong something...
				})
			},
			getByWarnId(warnId) { //根据Id查询预警信息详情
				event.stopPropagation();
				api(superviseApiList.getByWarnId, {
					"warnId": warnId
				}).then((res) => {
					if(res.status == 200 && res.data.data) {
						this.dataView = res.data.data;
						this.modal = true;
					}
				}, (err) => {
					//dong something...
				})
			},
			pageChange(num) { //页码改变的回调
				this.pageOption.current = num;
				this.getWarnHandleLedgerList();
			},
			changePageSize(num) { //切换每页条数时的回调
				this.pageOption.pageSize = num;
				this.getWarnHandleLedgerList();
			},
			closeModal() { //关闭modal页
				this.modal = false;
			},
			rowClick(params, index) { //点击单元行时触发
				var warnId = params.id;
				var warnTime = params.warntime;
				this.getProcessResultMsg(warnId, warnTime);
			},
			getProcessResultMsg(warnId, warnTime) { //根据ID查询预警台账流程信息
				this.flowRowsData = [];
				this.currentFlow = 1;
				api(superviseApiList.getProcessResultMsg, {
					"warnId": warnId
				}).then((res) => {
					if(res.status == 200 && res.data.data.list) {
						var sendeeList = res.data.data.list;
						var zrrName = "待定";
						for(var i in sendeeList) {
							var titleName = sendeeList[i].recName;
							var statusText;
							switch(sendeeList[i].status) {
								case '1':
									statusText = "未确认";
									break;
								case '2':
									statusText = "已确认";
									break;
								case '3':
									statusText = "已处理";
									break;
								default:
									statusText = sendeeList[i].status || "";
									break;
							};
							var contentText;
							if(sendeeList[i].status == "1") {
								contentText = "";
							} else if(sendeeList[i].status == "2" || sendeeList[i].status == "3") {
								this.currentFlow += 1;
								contentText = sendeeList[i].updateTime + " 确认备注：" + (sendeeList[i].remarks ? sendeeList[i].remarks : "无");
							};
							if(sendeeList[i].recType == "3") {
								zrrName = sendeeList[i].recName;
								if(sendeeList[i].status != "3") {
									statusText = "未处理";
									contentText = "";
								};
							};
							var resultText = "";
							if(sendeeList[i].result) {
								resultText = "属实";
							} else {
								resultText = "不属实";
							};
							if(sendeeList[i].status == "2") {
								statusText += resultText;
							}
							var flowItem = {
								title: titleName + statusText,
								content: contentText
							};
							this.flowRowsData.push(flowItem);
							if(sendeeList[i].status == 2 && !sendeeList[i].result) {
								return false;
							};
						};
						var flowItemFirst = {
							title: '预警产生',
							content: warnTime + " 责任人：" + zrrName
						};
						this.flowRowsData.unshift(flowItemFirst);
						this.showFlow = true;
					} else {
						this.showFlow = false;
					}
				}, (err) => {
					//dong something...
				})
			},
			checkButton(code) {
				if(this.authButton.indexOf(code) >= 0) {
					return true;
				} else {
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

<style type="text/css" scoped>
	.wAuto {
		width: auto;
	}
	
	.queryItem {
		width: 180px;
	}
	
	.flow {
		margin-top: 20px;
	}
	.iviewTree{
		max-height: 450px;
	    overflow: auto;
	}
</style>