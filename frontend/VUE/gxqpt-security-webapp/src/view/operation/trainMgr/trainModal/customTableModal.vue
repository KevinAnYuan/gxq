<template>
    <Modal @on-visible-change="openPoint" v-model="evalModal" title="选择导出列" width="20%" :closable='false' :mask-closable='false'>
        <div style="text-align:center">
            <Form ref="evalData" :model="evalData" :rules="ruleValidate" :label-width="70" class="clearfix">
                <Row :gutter="40">
                    <Col span="24">
                        <FormItem>
                            <CheckboxGroup v-model='checkList'>
                                <Checkbox v-for="(item,index) in listScreen" :key="index" :label="item.title" checked>{{item.title}}</Checkbox>
                            </CheckboxGroup>
                        </FormItem>
                    </Col>
                </Row>
            </Form>
        </div>
        <div slot="footer">
            <Button class="modalBtn" type="default" @click="closeModal" size="large">取消</Button>
            <Button class="modalBtn" type="primary" @click="handleSubmit" size="large">确定</Button>
        </div>
    </Modal>
</template>

<script>
export default {
    data(){
        return{
            evalData: {
                dealMan: '',
                assistant: [],
                advice:''
            },
            checkList:[],
            ruleValidate: {
            }
        }
    },
    created(){
        for(var i = 0;i<this.listScreen.length;i++){
            this.checkList.push(this.listScreen[i].title);
        }
    },
    props:{
        evalModal:{
            type: Boolean,
            default: false
        },
        closeModal: Function,
        listScreen:{
            tyep:Array,
            default:[]
        }
    },
    methods:{
        openPoint(status) {
            if(!status){
                this.$refs['evalData'].resetFields();
            }
        },
        handleSubmit() {
            this.$emit('checkData',this.checkList);
            // console.log(this.checkList);
        },
    }
}
</script>

<style scoped>
.ivu-form-item >>> .ivu-form-item-content{
    height:auto;
}
.ivu-checkbox-group >>> label{
    width: 100%;
    text-align: left;
    font-size: 16px;
}
.ivu-input-wrapper >>> textarea{
    resize: none;
}
</style>
