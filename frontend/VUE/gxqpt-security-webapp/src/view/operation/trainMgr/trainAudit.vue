<template>
	<Layout>
		<Content>
			<Breadcrumb>
				<BreadcrumbItem>培训管理</BreadcrumbItem>
				<BreadcrumbItem>培训审核</BreadcrumbItem>
			</Breadcrumb>
			<Card>
				<Form :model="searchCondition" ref="searchformData" inline :label-width="80">
                    <FormItem label="培训时间">
                        <DatePicker type="daterange" :editable="false" v-model="trainTime" placeholder="请选择服务时间" style="width: 200px"></DatePicker>
                    </FormItem>
                    <FormItem label="培训类别">
                      <Select v-model="currentTypeIdx" style="width:200px">
                        <Option :value="0">全部</Option>
                        <Option :value="i + 1" v-for="(item, i) in typeList" :key="i">{{item.type}}</Option>
                      </Select>
                    </FormItem>
                    <FormItem label="培训名称">
                        <Input type="text" v-model="searchCondition.name"></Input>
                    </FormItem>
                    <!--<FormItem label="培训内容">
                        <Input v-model="searchCondition.context"></Input>
                    </FormItem>-->
                    <FormItem label="提交单位">
                        <!-- <Select v-model="searchCondition.company" style="width: 140px;">
                            <Option :value="item.name" v-for="item in companyList" :key="item.id">{{item.name}}</Option>
                        </Select> -->
                        <Input type="text" v-model="searchCondition.company"></Input>
					</FormItem>
                    <FormItem label="提交人">
                        <!-- <div class="ivu-form-item-content" @click="openTreeModal(1)">
                            <div class="ivu-input-wrapper ivu-input-type">
                                <i class="ivu-icon ivu-icon-load-c ivu-load-loop ivu-input-icon ivu-input-icon-validate"></i>
                                <div class="ivu-input" style="width: 140px;">{{searchCondition.person}}</div>
                            </div>
                        </div> -->
                        <Input type="text" v-model="searchCondition.person"></Input>
					</FormItem>
					<FormItem label="状态" prop="status">
						<Select v-model="searchCondition.status" style="width: 140px;">
                            <Option value="">全部</Option>
                            <Option :value="1">申请</Option>
                            <!--<Option :value="2">驳回</Option>-->
                            <Option :value="3">处理中</Option>
                            <Option :value="4">驳回后申请</Option>
                            <Option :value="5">完结</Option>
                        </Select>
					</FormItem>
                    <FormItem label="任务编号" style="width: 200px">
                        <Input v-model="searchCondition.applyCode"></Input>
                    </FormItem>
					<FormItem :label-width="20">
						<Button type="primary" @click="preSearch" v-if="authButton.includes('ops_train_audit_query')">查询</Button>
					</FormItem>
				</Form>
                <hy-table
                    highlight-row
                    border
                    :current="pageInfo.pageNo"
                    :columns="tableList.columns"
                    :data="tableList.data"
                    :total="Number(pageInfo.total)"
                    :pageSize="Number(pageInfo.pageSize)"
                    pageType="small"
                    show-elevator
                    show-sizer
                    ref="thisTable"
                    @on-page-change="pageChange" />
                <Modal v-model="dealDetailModal" title="审批处理" width="40%" :mask-closable="false">
                    <Form ref="modalForm" :rules="validform" :model="checkInfo" :label-width="90" class="clearfix">
                        <FormItem label="处理方案" prop="status" :required="true">
                            <RadioGroup v-model="checkInfo.status">
                                <Radio :label="0">同意</Radio>
                                <Radio :label="1">驳回</Radio>
                            </RadioGroup>
                        </FormItem>
                        <template v-if="checkInfo.status === 0">
                            <FormItem label="处理意见" prop="advice">
                                <Input
                                    v-model="checkInfo.advice" 
                                    type="textarea"
                                    :autosize="{minRows: 5,maxRows: 8}"
                                    placeholder="请输入处理意见..."
                                    :maxlength="200"></Input>
                            </FormItem>
                            <FormItem label="培训人" :required="true" prop="trainerId">
                                <div class="ivu-form-item-content" @click="openTreeModal(2)">
                                    <div class="ivu-input-wrapper ivu-input-type">
                                        <i class="ivu-icon ivu-icon-load-c ivu-load-loop ivu-input-icon ivu-input-icon-validate"></i>
                                        <div class="ivu-input" style="width: 100%;">{{trainerInfo.title}}</div>
                                    </div>
                                </div>
                            </FormItem>
                            <FormItem label="协助人">
                                <div class="ivu-form-item-content" @click="openTreeModal(3)">
                                    <div class="ivu-input-wrapper ivu-input-type">
                                        <i class="ivu-icon ivu-icon-load-c ivu-load-loop ivu-input-icon ivu-input-icon-validate"></i>
                                        <div class="ivu-input" style="width: 100%;">{{assistantNames.join(',')}}</div>
                                    </div>
                                </div>
                            </FormItem>
                        </template>
                        <template v-else-if="checkInfo.status === 1">
                            <FormItem label="驳回理由" prop="advice" :required="true">
                                <Input
                                    v-model="checkInfo.advice"
                                    type="textarea"
                                    :autosize="{minRows: 5, maxRows: 8}"
                                    :maxlength="200"></Input>
                            </FormItem>
                        </template>
                    </Form>
                    <div slot="footer">
                        <Button type="default" @click="dealDetailModal = false" size="large">取消</Button>
                        <Button type="primary" @click="doCheck" size="large">确定</Button>
                    </div>
                </Modal>
                <!-- 选择培训人的树形弹窗 -->
                <personTreeModal ref="getTrainer" :multiple="personStatus === 3" @on-ok="selectPerson" />
			</Card>
		</Content>
	</Layout>
</template>
<script>
import {mapState} from 'vuex'
import api from '@/api/axiosApi'
import operationApiList from '@/api/operationApiList'
import apiList from '@/api/comApiList'

import personTreeModal from './trainModal/personTreeModal.vue'

function getDateRange(time) {
  if (!time) {
    return ''
  }
  // 结束日期
  const endDate = new Date(time)
  // 当前日期往前推30天
  const startDate = new Date(time - 30 * 24 * 60 * 60 * 1000)
  return {
    start: `${startDate.getFullYear()}-${startDate.getMonth() + 1}-${startDate.getDate()}`,
    end: `${endDate.getFullYear()}-${endDate.getMonth() + 1}-${endDate.getDate()}`
  }
}

export default {
    components: {
        personTreeModal
    },
    data() {
        const vm = this
        return {
            personStatus: 1,
            currentTypeIdx: 0,
            typeList: [],
            companyList: [],
            // 当前操作的培训行信息
            currentInfo: {},
            dealDetailModal: false,
            searchCondition: {
                type: '',
                name: '',
                context: '',
                company: '',
                person: '',
                status: '',
                applyCode: this.$route.query.code || ''
            },
            checkInfo: {
                // 培训意见
                advice: '',
                // 是否同意1驳回0同意
                status: ''
            },
            // 培训人的信息
            trainerInfo: {
                title: '',
                id: ''
            },
            // 协助人名单
            assistantList: [],
            // 培训时间的数组
            trainTime: [],
            tableList: {
                columns: [{
                    title: '序号',
                    type: 'index',
                    width: 80,
                    align: 'center'
                }, {
                    title: '任务编号',
                    key: 'applyCode'
                }, {
                    title: '培训名称',
                    key: 'name'
                }, {
                    title: '培训类别',
                    key: 'type'
                }, {
                    title: '培训时间',
                    key: 'trainTime',
                    width: 160
                }, {
                    title: '提交单位',
                    key: 'company'
                }, {
                    title: '提交人',
                    key: 'person'
                }, {
                    title: '状态',
                    key: 'status',
                    render: (h, params) => {
                        const status = ['--', '申请', '驳回', '处理中', '驳回后申请', '完结']
                        return h('span', status[params.row.status])
                    }
                }, {
                    title: '培训评分',
                    key: 'trainSorce'
                }, {
                    title: '培训反馈文件',
                    key: 'fileUrl',
                    align: 'center',
                    render: (h, params) => {
                        const {files, id} = params.row
                        const {userId, token} = vm.$store.state.userInfo
                        if (files && files.length > 0 && vm.authButton.includes('ops_train_audit_download') && params.row.status === 5) {
                            const ids = []
                            files.map(file => {
                                ids.push(file.id)
                            })
                            return h('a', {
                                attrs: {
                                    href: `/api/file/p/file/downloadpackage?ids[]=${ids.join(',')}&userId=${userId}&token=${token}`,
                                    target: '_blank'
                                }
                            }, [
                                h('Button', {
                                    props: {
                                        type: 'primary',
                                        size: 'small'
                                    },
                                    style: {
                                        marginRight: '5px'
                                    }
                                }, '下载'),
                            ])
                        } else {
                            return h('span', '--')
                        }
                    }
                }, {
                    title: '操作',
                    type: 'act',
                    width: 200,
                    align: 'center',
                    render: (h, params) => {
                        const deal = h('Button', {
                            props: {
                                type: 'primary',
                                size: 'small'
                            },
                            style: {
                                marginRight: '5px'
                            },
                            on: {
                                click: () => {
                                    vm.dealDetailModal = true
                                    vm.currentInfo = params.row
                                }
                            }
                        }, '处理')
                        const detail = h('Button', {
                            props: {
                                type: 'primary',
                                size: 'small'
                            },
                            style: {
                                marginRight: '5px'
                            },
                            on: {
                                click: () => {
                                    vm.$router.push({
                                        name: 'trainDetail',
                                        params: {
                                            id: params.row.id
                                        },
                                        query: {
                                            status: params.row.status
                                        }
                                    })
                                }
                            }
                        }, '详情')
                        const btns = []
                        if (vm.authButton.includes('ops_train_audit_handle') && (params.row.status == 1|| params.row.status == 4)) {
                            btns.push(deal)
                        }
                        if (vm.authButton.includes('ops_train_audit_detail')) {
                            btns.push(detail)
                        }
                        return h('div', btns)
                    }
                }],
                data: []
            },
            pageInfo: {
                pageNo: 1,
                pageSize: 10,
                total: 0
            },
            validform: {
                status: [{
                    required: true,
                    message: '必须选择处理方案',
                    trigger: 'change',
                    validator: (rule, value, cb) => {
                        if (vm.checkInfo.status === '') {
                            cb(new Error('请选择是否同意'))
                        } else {
                            cb()
                        }
                    }
                }],
                trainerId: [{
                    trigger: 'change',
                    validator: (rule, value, cb) => {
                        if (!vm.trainerInfo.id) {
                            cb(new Error('请选择培训人'))
                        } else {
                            cb()
                        }
                    }
                }],
                advice: [{
                    trigger: 'blur',
                    validator: (rule, value, cb) => {
                        if (vm.checkInfo.advice === '' && vm.checkInfo.status === 1) {
                            cb(new Error('请填写驳回理由'))
                        } else {
                            cb()
                        }
                    }
                }]
            }
        }
    },
    computed: {
        assistantNames () {
            const names = []
            this.assistantList.map(item => {
                names.push(item.title)
            })
            return names
        },
        ...mapState(['authButton'])
    },
    mounted () {
        this.search()
        this.getTrainTypeList()
        // this.getMyOrgList()
    },
    methods: {
        preSearch() {
            this.pageInfo.pageNo = 1
            this.search()
        },
        pageChange(pageNo, pageSize) {
            this.pageInfo.pageNo = pageNo
            this.pageInfo.pageSize = pageSize
            this.search()
        },
        // 获取所有单位
        getMyOrgList() {
          const vm = this
          api(apiList.getMyOrgList)
          .then(res => {
            if (res.data.errcode === 0) {
              vm.companyList = res.data.data.orgList
            }
          }, error => {console.log(error)})
        },
        // 查询所有的类型和名称的关系列表
        getTrainTypeList() {
            const vm = this
            api(operationApiList.trainTypeList)
            .then(res => {
                if (res.data.errcode === 0) {
                    const typeList = res.data.data
                    typeList.push({
                        type: '自定义培训'
                    })
                    vm.typeList = res.data.data
                }
            }, error => {console.log(error)})
        },
        // 搜索列表
        doSearch(pageNo, pageSize) {
            const vm = this
            const endTime = vm.trainTime.length > 0 ? getDateRange(vm.trainTime[1]).end : ''
            const startTime = vm.trainTime.length > 0 ? getDateRange(vm.trainTime[0]).end : ''
            api(operationApiList.trainAuditorPage, {
                pageNo: pageNo || vm.pageInfo.pageNo,
                pageSize: pageSize || vm.pageInfo.pageSize,
                data: {
                    ...vm.searchCondition,
                    handle: 0,
                    endTime: endTime ? `${endTime} 23:59:59` : '',
                    startTime: startTime ? `${startTime} 00:00:00` : ''
                }
            }).then(res => {
                if (res.data.errcode === 0) {
                    const result = res.data.data
                    vm.pageInfo.pageNo = result.pageNum
                    vm.pageInfo.total = result.total
                    vm.tableList.data = result.list
                }
            }, error => {console.log(error)})
        },
        search() {
            this.doSearch()
        },
        // 审批
        doCheck() {
            const vm = this
            vm.$refs.modalForm.validate(valid => {
                if (valid) {
                    let data = {}
                    // 同意
                    if (vm.checkInfo.status === 0) {
                        const assistant = []
                        vm.assistantList.map(item => {
                            assistant.push({
                                id: item.id,
                                name: item.title
                            })
                        })
                        data = {
                            // 培训人id
                            trainerId: vm.trainerInfo.id,
                            // 培训人姓名
                            trainer: vm.trainerInfo.title,
                            // 协助人
                            assistant: assistant,
                            ...vm.checkInfo
                        }
                    } else {
                        data = vm.checkInfo
                    }
                    api(operationApiList.trainHandleApply, {
                        // 培训id
                        id: vm.currentInfo.id,
                        ...data
                    }).then(res => {
                        if (res.data.errcode === 0) {
                            vm.$Message.success('处理成功！！！')
                            vm.dealDetailModal=  false
                            vm.search()
                        }
                    }, error => {console.log(error)})
                } else {
                    vm.$Message.info('表单验证失败！')
                }
            })
        },
        // 打开树形结构，选择人员
        openTreeModal(status) {
            this.personStatus = status
            this.$nextTick(() => {
                this.$refs.getTrainer.open()
            })
        },
        // 选择人员之后的回调
        selectPerson(list) {
            // 选择人的status,1为提交人（查询）2为培训人3为协助人
            if (this.personStatus == 3) {
                this.assistantList = list
            } else if (this.personStatus == 2) {
                this.trainerInfo = list.length > 0 ? list[0] : {}
            } else {
                this.searchCondition.person = list.length > 0 ? list[0].title : ''
            }
        }
    },
    watch: {
        'checkInfo.status' (val, oldVal) {
            this.checkInfo.advice = ''
            if (val === 0) {
                this.$nextTick(() => {
                    this.$refs.modalForm.validateField('advice', res => {})
                })
            }
        },
        currentTypeIdx () {
            if (this.currentTypeIdx === 0) {
                this.searchCondition.type = ''
            } else {
                this.searchCondition.type = this.typeList[this.currentTypeIdx - 1].type
            }
        }
    }
}
</script>
<style lang="less" scoped="scoped">
	.act-btn-group{
		button{
			margin: 0 12px;
		}
    }
</style>