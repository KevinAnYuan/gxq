<template>
  <Layout>
    <Content>
      <Breadcrumb>
        <BreadcrumbItem>培训管理</BreadcrumbItem>
        <BreadcrumbItem>培训课程管理</BreadcrumbItem>
      </Breadcrumb>
      <Card>
        <Row>
          <Col span="24">
            <Form
              ref="formValidate"
              inline
              :label-width="100"
              :model="searchData.name">
              <FormItem label="课程名称：">
                <Input
                  type="text"
                  v-model="searchData.name">
                </Input>
              </FormItem>
              <FormItem :label-width="20">
                <Button type="primary" @click="getSearchTree">查询</Button>
              </FormItem>
            </Form>
          </Col>
        </Row>
        <Row>
          <Col span="6">
            <Tree :data="ztreeData" :render="renderContent" id="trainClassify"></Tree>
          </Col>
          <Col span="16" v-if="currentNodeInfo.level === 1">
            <h3>培训信息</h3>
            <Form ref="formValidate" :label-width="100">
              <FormItem label="培训类别：">
                <Row>
                  <Col span="24">
                    <Input :value="currentNodeInfo.type" disabled></Input>
                  </Col>
                </Row>
              </FormItem>
              <FormItem label="培训名称：">
                <Row>
                  <Col span="24">
                    <Input :value="currentNodeInfo.title" disabled></Input>
                  </Col>
                </Row>
              </FormItem>
              <FormItem label="培训内容：">
                <Row>
                  <Col span="24">
                    <Input :value="currentNodeInfo.serviceInfo" disabled type="textarea" :autosize="{minRows: 3,maxRows: 5}"></Input>
                  </Col>
                </Row>
              </FormItem>
            </Form>
          </Col>
        </Row>
      </Card>
      <Modal v-model="modal" :mask-closable="false">
        <Form ref="modalFormValidate" inline :label-width="90" :rules="modalRules" :model="modalFormData">
          <!-- 点击根节点‘培训课程管理’ -->
          <template v-if="currentNodeInfo.level === -1">
            <FormItem label="培训类型" style="width: 100%;" prop="type">
              <Input
                type="text"
                :maxlength="100"
                v-model="modalFormData.type">
              </Input>
              <p class="fontTooltip">内容限100字符以内</p>
            </FormItem>
          </template>
          <template v-else-if="currentNodeInfo.level === 0">
            <FormItem label="新增类型" v-if="currentAct === actTypes.create" prop="status">
              <Select v-model="modalFormData.status" style="width:150px;">
                <Option value='1'>--新增子类--</Option>
                <Option value='2'>--新增类--</Option>
              </Select>
            </FormItem>
            <template v-if="modalFormData.status === '1'">
              <FormItem label="培训名称" style="width: 100%;" prop="name">
                <Input
                  type="text"
                  :maxlength="100"
                  v-model="modalFormData.name">
                </Input>
                <p class="fontTooltip">内容限100字符以内</p>
              </FormItem>
              <FormItem label="培训内容" style="width: 100%;" prop="context">
                <Input
                  type="text"
                  :maxlength="1000"
                  v-model="modalFormData.context">
                </Input>
                <p class="fontTooltip">内容限1000字符以内</p>
              </FormItem>
            </template>
            <template v-else-if="modalFormData.status === '2'">
              <FormItem label="培训类型" style="width: 100%;" prop="type">
                <Input
                  type="text"
                  v-model="modalFormData.type">
                </Input>
              </FormItem>
            </template>
          </template>
          <template v-else>
            <FormItem label="培训名称" style="width: 100%;" prop="name">
              <Input
                type="text"
                v-model="modalFormData.name">
              </Input>
            </FormItem>
            <FormItem label="培训内容" style="width: 100%;" prop="context">
              <Input
                type="text"
                v-model="modalFormData.context">
              </Input>
            </FormItem>
          </template>
        </Form>
        <div slot="footer">
          <Button class="modalBtn" type="default" @click="modal = false" size="large">取消</Button>
          <Button class="modalBtn" type="primary" @click="saveOrUpdate" size="large">确定</Button>
        </div>
      </Modal>
    </Content>
  </Layout>
</template>

<script>
import {mapState} from 'vuex'
import api from '@/api/axiosApi'
import {changeTreeData} from '@/assets/js/utils'
import operationApiList from '@/api/operationApiList'
// 操作类型
const actTypes = {
  edit: 'edit',
  create: 'create'
}

function handleTreeData(data, isSearch, hasName) {
  const treeData = []
  const childrenData = []
  data.map(item => {
    const {level, children, type, id, parentId} = item
    if (level == 0) {
      let obj = {
        id,
        children,
        parentId,
        title: type,
        level: 0,
        expand: false
      }
      if(isSearch && hasName){
        obj.expand = true
      }
      childrenData.push(obj)
    }
  })
  treeData.push({
    id: '',
    children: childrenData,
    parentId: '',
    title: '培训课程管理',
    level: -1,
    expand: true
  })
  return treeData
}
export default {
  data() {
    return {
      searchData: {
        name: ''
      },
      // 树的所有节点数组
      treeRoot: [],
      // 当前操作的节点对象，包含父节点的nodekey
      currentNode: {},
      // 操作类型
      actTypes,
      // 当前操作的类型
      currentAct: 'edit',
      // 当前操作的节点的信息
      currentNodeInfo: {
        // 自生的id
        id: '',
        // 父节点的id，如果没有父节点就为-1
        parentId: ''
      },
      // 当前操作节点的详细信息
      currentNodeDetail: {
        id: '',
        parentId: ''
      },
      // 新增或者编辑时的表单
      modalFormData: {
        // 子类或者同级类
        status: '1',
        type: '',
        name: '',
        context: ''
      },
      ztreeData: [],
      buttonProps: {
        type: 'ghost',
        size: 'small',
      },
      modal: false,
      modalRules: {
        name: [{required: true, message: '不可为空', trigger: 'blur'}],
        status: [{required: true, message: '不可为空', trigger: 'change'}],
        context: [{required: true, message: '不可为空', trigger: 'blur'}],
        type: [{required: true, message: '不可为空', trigger: 'blur'}]
      }
    }
  },
  computed: {
    saveOrUpdateUrl() {
      return this.currentAct === actTypes.create ? 'courseSave' : 'courseUpdate'
    },
    ...mapState(['authButton'])
  },
  methods: {
    // 获取树结构,isAct代表是不是操作之后的刷新
    getTreeData(isAct, node) {
      const vm = this
      api(operationApiList.trainCourseTree,{serviceName: vm.searchData.name})
      .then(res => {
        if (res.data.errcode === 0) {
          const treeData = handleTreeData(res.data.data, '', '')
          // 操作--新增
          if (isAct) {
            const opts = {
              root: vm.treeRoot,
              node: vm.currentNode,
              data: vm.currentNodeInfo,
              nodeInfo: node,
              name: 'serviceName',
              type: 'createChild'
            }
            // vm.currentNodeInfo.serviceName = res.data.data.serviceName
            changeTreeData(opts)
          } else {
            vm.ztreeData = JSON.parse(JSON.stringify(treeData).replace(/serviceName/g, 'title'))
          }
        }
      }, error => {console.log(error)})
    },
    // 关键字查询
    getSearchTree() {
      const vm = this
      api(operationApiList.trainCourseTree,{serviceName: vm.searchData.name})
      .then(res => {
        if (res.data.errcode === 0) {
          const treeData = handleTreeData(res.data.data, 'search', vm.searchData.name)
          vm.ztreeData = JSON.parse(JSON.stringify(treeData).replace(/serviceName/g, 'title'))
        }
      }, error => {console.log(error)})
    },
    renderContent(h, { root, node, data }) {
      const vm = this
      // 添加按钮
      const createBtn = h('Button', {
        props: {
          icon: 'plus',
          ...this.buttonProps
        },
        style: {
          marginRight: '8px',
          color: 'green'
        },
        on: {
          click: () => {
            this.modalFormData.type = ''
            this.modalFormData.status = '1'
            this.modalFormData.name = ''
            this.modalFormData.context = ''
            this.currentNodeInfo = data
            this.currentAct = actTypes.create
            this.modal = true
          }
        }
      })
      // 编辑按钮
      const editBtn = h('Button', {
        props: {
          icon: 'edit',
          ...this.buttonProps
        },
        style: {
          marginRight: '8px',
          color: 'blue'
        },
        on: {
          click: () => {
            this.currentNodeInfo = data
            this.currentAct = actTypes.edit
            this.modal = true
            this.modalFormData.type = data.level == 0 ? data.title : data.type
            this.modalFormData.status = data.level == 0 ? '2' : '1'
            this.modalFormData.context = data.serviceInfo
            this.modalFormData.name = data.level == 0 ? '' : data.title
            this.modalFormData.id = data.id
          }
        }
      })
      // 删除按钮
      const deleteBtn = h('Button', {
        props: {
          icon: 'close',
          ...vm.buttonProps
        },
        style: {
          marginRight: '8px',
          color: 'red'
        },
        on: {
          click: () => {
            vm.treeRoot = root
            vm.currentNode = node
            vm.currentNodeInfo = data
            vm.$Modal.confirm({
              title: '删除确认',
              content: '确定删除此节点吗？',
              onOk: () => {
                vm.doRemove()
              }
            })
          }
        }
      })
      let btns = []
      // 根节点层，不可删除不可修改
      if (data.level == -1) {
        if (vm.authButton.includes('ops_train_curriculum_manage_add')) {
          btns.push(createBtn)
        }
      // 第一层（培训类别）
      } else if (data.level == 0) {
        if (vm.authButton.includes('ops_train_curriculum_manage_add')) {
          btns.push(createBtn)
        }
        if (vm.authButton.includes('ops_train_curriculum_manage_update')) {
          btns.push(editBtn)
        }
        if (vm.authButton.includes('ops_train_curriculum_manage_delete') && node.children.length === 0) {
          btns.push(deleteBtn)
        }
      // 第二层（培训名称）
      } else if (data.level == 1) {
        if (vm.authButton.includes('ops_train_curriculum_manage_update')) {
          btns.push(editBtn)
        }
        if (vm.authButton.includes('ops_train_curriculum_manage_delete')) {
          btns.push(deleteBtn)
        }
      } else {
        if (vm.authButton.includes('ops_train_curriculum_manage_add')) {
          btns.push(createBtn)
        }
        if (vm.authButton.includes('ops_train_curriculum_manage_update')) {
          btns.push(editBtn)
        }
        if (vm.authButton.includes('ops_train_curriculum_manage_delete')) {
          btns.push(deleteBtn)
        }
      }
      return h('span', {
        attrs: {
          class: 'doc-tree-title'
        }
      }, [
        h('span', {
          on: {
            click: () => {
              // 点击文字也可以展开树结构
              Vue.set(data, 'expand', !data.expand)
              this.currentNodeInfo = data
              // this.getDetail()
            }
          },
          attrs: {
            class: 'title'
          }
        }, data.title),
        h('span', {
          attrs: {
            class: 'act-btns-container'
          },
          style: {
            marginLeft: '10px'
          }
        }, btns)
      ])
    },
    doRemove() {
      const vm = this
      api(operationApiList.courseDelete, {
        id: vm.currentNodeInfo.id
      }).then(res => {
        if (res.data.errcode === 0) {
          vm.$Message.success('删除成功！')
          changeTreeData({
            root: vm.treeRoot,
            node: vm.currentNode,
            data: vm.currentNodeInfo,
            type: 'delete'
          })
        }
      }, error => {console.log(error)})
    },
    // 保存或者更新节点信息
    saveOrUpdate() {
      const vm = this
      vm.$refs.modalFormValidate.validate(valid => {
        if (valid) {
          const {parentId, id, level} = {...vm.currentNodeInfo}
          // 点击课程培训管理新增类别
          if (level === -1) {
            vm.modalFormData.status = '2'
          }
          let data = {
            ...vm.modalFormData
          }
          // 编辑
          if (vm.currentAct === actTypes.edit) {
            data.id = id
          } else {
            // 新增子类
            if (vm.modalFormData.status == '1' && level == 0) {
              data.pid = id
              data.level = level + 1
              data.type = vm.currentNodeInfo.title
            } else {
              data.pid = parentId || -1
              data.level = level === -1 ? 0 : level
            }
          }
          api(operationApiList[vm.saveOrUpdateUrl], data)
          .then(res => {
            if (res.data.errcode === 0) {
              // 编辑
              if (vm.currentAct === actTypes.edit) {
                const opts = {
                  root: vm.treeRoot,
                  node: vm.currentNode,
                  data: vm.currentNodeInfo,
                  nodeInfo: vm.modalFormData,
                  type: 'edit'
                }
                if (level == 0) {
                  opts.name = 'type'
                }
                changeTreeData(opts)
              } else {
                if (!res.data.data.serviceName) {
                  res.data.data.serviceName = res.data.data.type
                }
                vm.getTreeData(true, res.data.data)
              }
              vm.modal = false
              vm.$refs.modalFormValidate.resetFields()
            }
          }, error => {console.log(error)})
        }
      })
    }
  },
  mounted() {
    this.getTreeData()
  }
}
</script>

<style lang="less" scoped>
.w168 {
  width: 168px;
}
.fontTooltip{
  font-size: 12px;
  color: #bbb;
}
</style>

<style lang="less">
#trainClassify{
  .doc-tree-title{
    display: inline-block;
    vertical-align: middle;
    .title{
      display: inline-block;
      height: 24px;
      float: left;
      line-height: 24px;
      cursor: pointer;
    }
    .act-btns-container{
      display: none;
      margin-left: 10px;
    }
    &:hover{
      .act-btns-container{
        display: inline-block;
      }
    }
  }
}
</style>