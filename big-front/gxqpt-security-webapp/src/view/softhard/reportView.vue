<!-- 报告查看 -->
<template>
	<Layout>
		<Content>
			<Breadcrumb>
				<BreadcrumbItem>{{ title }}</BreadcrumbItem>
			</Breadcrumb>
			<Card>
				<Form ref="formValidate" inline :label-width="110" :model="serverData">
					<FormItem label="报告年度：">
						 <DatePicker v-model="serverData.date" type="year" placeholder="请选择年份" style="width: 200px"></DatePicker>
					</FormItem>
          <FormItem label="报告月份：">
						<Select v-model="serverData.month" style="width:200px">
								<Option v-for="item in monthList" :value="item" :key="item">{{ item }}月</Option>
						</Select>
					</FormItem>
					<FormItem :label-width="20">
						<Button type="primary" @click="search">查询</Button>
						<Button type="primary" @click="addNew">新增</Button>
					</FormItem>
				</Form>
				<hy-table ref="selection" :data="data" :columns="columns" :current="pageOption.pageNo" :total="pageOption.total" :page-size="pageOption.pageSize" @on-change="pageChange" @on-page-size-change="changePageSize" show-sizer show-elevator/>
			</Card>
      <Modal v-model="createModal" :title="Modletitle" width="40%" :mask-closable="false">
        <Form ref="createForm" :model="createForm" :rules="createRule" :label-width="90">
          <FormItem label="机房编号" prop="roomNumber">
            <Input type="text" v-model="createForm.roomNumber" disabled />
          </FormItem>
          <FormItem label="管理员" prop="roomAdmin">
            <Input type="text" v-model="createForm.roomAdmin" disabled />
          </FormItem>
          <FormItem label="联系电话" prop="contactNumber">
            <Input type="number" v-model="createForm.contactNumber" disabled />
          </FormItem>
          <FormItem label="报告年度" :required="true" prop="date">
						<DatePicker v-model="createForm.date" type="year" placeholder="请选择年份" style="width: 200px"></DatePicker>
          </FormItem>
          <FormItem label="报告年月" :required="true" prop="month">
						<Select v-model="createForm.month" style="width:200px">
								<Option v-for="item in monthList" :value="item" :key="item">{{ item }}月</Option>
						</Select>
          </FormItem>
          <FormItem label="附件报告" style="margin-top: 40px;" :required="true" prop="file">
            <hy-upload ref="evalReport" multiple :on-success="setEvalReport" :format="['xls','xlsx','doc', 'docx','pdf']" :on-remove="removeEvalReport" :before-upload="evalBeforeUpload" :defaultFileList="createForm.attachment"></hy-upload> <!-- :defaultFileList="createForm.attachment" -->
          </FormItem>
        </Form>
        <div slot="footer">
          <Button class="modalBtn" type="primary" @click="saveHostRoom" size="large">确定</Button>
          <Button class="modalBtn" type="default" @click="createModal = false" size="large">取消</Button>
        </div>
      </Modal>
		</Content>
	</Layout>
</template>

<script>
	import api from '@/api/axiosApi'
	import softhardApiList from '@/api/softhardApiList'
	import hyUpload from '@/components/hengyun/hyUpload'
	import { mapState } from 'vuex'
	function getCreateForm() {
		return {
			year:'',
			month:'',
			attachment:[],
		}
	}
	export default {
		data() {
			return {
		Modletitle: '新增',
        title:this.$store.state.title,
        serverData: { //查询参数
					year:"",
					date:"",
          month:"",
				},
				modelType:'0',
				monthList:[1,2,3,4,5,6,7,8,9,10,11,12],
        createForm:{
          roomNumber:this.$route.params.params.roomNumber,
          roomAdmin:this.$route.params.params.roomAdmin,
          contactNumber:this.$route.params.params.contactNumber,
          year:'',
					month:'',
					date:'',
					attachment:[],
        },
        createModal:false,
        createRule: {
          date: [{
            required: true,
            trigger: 'blur',
            validator: (rule, value, cb) => {
							console.log(this.createForm.date);
              if (!this.createForm.date) {
                cb(new Error('年度选择不能为空'))
                return
              }
              cb()
            }
          }],
					month: [{type:'number',required: true, message: '月度不能为空', trigger: 'blur'}],
          file: [{
            required: true,
            trigger: 'blur',
            validator: (rule, value, cb) => {
							console.log(this.createForm.attachment);
              if (this.createForm.attachment.length==0) {
                cb(new Error('附件报告不能为空'))
                return
              }
              cb()
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
						title: '报告年度',
						key: 'year'
          },
					{
						title: '报告月份',
						key: 'month'
          },
          {
						title: '报告文件',
						width: 300,
						key: 'fileName',
						render: (h, params) => {
							let fileArr = [];
							params.row.fileUrlArr.forEach((item,idx)=>{
								let mod = h('p', {
									style: {
										margin:'5px 0',
										color:'#2d8cf0',
										cursor:'pointer'
									},
									on: {
										click: () => {
											this.downloadFile(item,params.row.fileNameArr[idx]);
										}
									}
								},params.row.fileNameArr[idx]);
								fileArr.push(mod)
							})
							return h('div', fileArr);
						}
					},
					{
						title: '报告大小',
						key: 'fileSize'
          },
          {
						title: '上传用户',
						key: 'createUserName'
					},
					{
						title: '上传时间',
						width: 220,
						align: 'center',
						key: 'createTime'
					},
					{
						title: '操作',
						key: 'act',
						width: 160,
						render: (h, params) => {
							const edit = h('Button', {
								props: {
									type: 'primary',
									size: 'small'
								},
								// style: {
								// 	display:this.checkButton('hardware_bmyh_fwqgl_xq')?'inline-block':'none'
								// },
								on: {
									click: () => {
										this.Modletitle = '修改'
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
							return h('div', [edit, del]);
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
			this.emrPageList(); //查询服务分配管理分页
			this.createForm.roomNumber = this.$route.params.params.roomNumber;
			this.createForm.roomAdmin = this.$route.params.params.roomAdmin;
			this.createForm.contactNumber = this.$route.params.params.contactNumber;
		},
		methods: {
			pageChange(num) { //页码改变的回调
				this.pageOption.pageNo = num;
				this.emrPageList();
			},
			changePageSize(num) { //切换每页条数时的回调
				this.pageOption.pageSize = num;
				this.emrPageList();
			},
			emrPageList() { // 查询分页
				this.data = [];
				var formData = {
					"data": { ...this.serverData,
            hostMgId:this.$route.params.params.id
					},
					"pageNo": this.pageOption.pageNo,
					"pageSize": this.pageOption.pageSize
				};
				api(softhardApiList.emrPageList, formData).then((res) => {
					if(res.status == 200 && res.data.data) {
						this.data = res.data.data.list;
						if(this.data.length>0){
							this.pageOption.pageSize = res.data.data.pageSize;
							this.pageOption.total = res.data.data.total;
							this.pageOption.pageNo = res.data.data.pageNum;
							this.data.forEach(item=>{
								item.createTime = item.createTime.replace("-","年");
								item.createTime = item.createTime.replace("-","月");
								item.createTime = item.createTime.replace(" ","日 ");
								item.fileSize = (parseInt(item.fileSize/1024)) > 1024 ? (parseInt(item.fileSize/1048576)) + 'M' : (parseInt(item.fileSize/1024)) + 'KB'

								item.fileNameArr = item.fileName.split(",");
								item.fileUrlArr = item.fileUrl.split(",");
							})
							console.log(this.data);
						};
					}
				}, (err) => {
					//dong something...
				})
			},
			search() {
				this.pageOption.pageNo=1;
				(this.serverData.date) ? this.serverData.year = this.serverData.date.getFullYear() : this.serverData.year='';
				console.log(this.serverData);
				this.emrPageList();
      },
      addNew(){
			this.Modletitle = '新增'
			this.modelType = '0';
        	this.createModal = true;
      },
      saveHostRoom(){
				let url = '';
				(this.modelType=='0') ? url = softhardApiList.saveEquipmentMonitorReport : url = softhardApiList.updateEquipmentMonitorReport;
				this.createForm.createTime = null;
				this.$refs['createForm'].validate((valid) => {
					if(valid) {
						this.createForm.year = this.createForm.date.getFullYear();
						this.createForm.hostMgId = this.$route.params.params.id;
						api(url, this.createForm).then((res) => {
							if(res.status == 200 && res.data.data) {
								if(res.data.data && res.data.errmsg == 'ok'){
									let msg = (this.modelType=='0') ? '保存成功！' : '编辑成功！';
									this.$Message.success(msg);
									this.createForm.date = ''
                  this.createForm.month = ''
                  this.createForm.attachment = []
									this.emrPageList();
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
      evalBeforeUpload() { //文件上传前清空
      // this.$refs.evalReport.$children[0].clearFiles();
    	},
			setEvalReport(response, file, fileList) {//获取附件
				console.log(fileList);
				fileList.forEach(item => {
					let obj = {};
					if(item.response){
						obj.busType = 'hd_applymanage_upload';
						obj.fileName = item.response.data.list[0].submittedFileName;
						obj.fileSize = item.response.data.list[0].size;
						obj.fileType = item.response.data.list[0].mime;
						obj.fileUrl = item.response.data.list[0].url;
						obj.name = item.response.data.list[0].submittedFileName;
						obj.url = item.response.data.list[0].url;
						this.createForm.attachment = this.arrConcat(this.createForm.attachment,[obj]);
					}
				});
				console.log(this.createForm);
			},
			removeEvalReport(file, fileList) {
				this.createForm.attachment = [];
				this.createForm.attachment = fileList;
			},
			itemDelete(id,idx) {//删除
        const vm = this
        vm.$Modal.confirm({
          title: '删除确认',
          content: '确认删除吗？',
          onOk: () => {
            api(softhardApiList.deleteEquipmentMonitorReport, {id: id}).then(res => {
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
				this.createForm.attachment = [];
				this.idx = idx;
				// 删除
				if (type === 0) {
					this.tableList.data.splice(idx, 1)
				} else {
				// 修改
					const data = JSON.parse(JSON.stringify(this.data[idx]))
					console.log(data);
					// this.createForm = {...getCreateForm(), ...data};
					this.createForm.date = new Date(data.year.toString());
					this.createForm.month = data.month;
					data.fileUrlArr.forEach((item,idx)=>{
						let obj = {};
						obj.fileUrl = item;
						obj.fileName = data.fileNameArr[idx];
						obj.name = data.fileNameArr[idx];
						obj.url = item;
						this.createForm.attachment.push(obj);
					});
					this.createForm.id = data.id;
					console.log(this.createForm);
					this.createModal = true;
					this.modelType = '1';
				}
			},
			downloadFile(url,filename){//下载文件
				let urlDownload = "/api/file/file/download?url=" + url + "&filename=" + filename;
				window.open(urlDownload);
			},
			arrConcat(arr1,arr2){//数组去重
          let arr = {};
          for(var i=0;i<arr1.length;i++){
              arr[arr1[i].name]=true;
          }
          for (var i = 0; i < arr2.length; i++) {
              if(!arr[arr2[i].name]){
                  arr1.push(arr2[i]);
              }
          };
          return arr1;
      },
			checkButton(code){//按钮权限
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
