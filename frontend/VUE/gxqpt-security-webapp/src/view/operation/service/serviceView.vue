<template>
    <div class="ivu-layout serviceDetail">
        <Row style="margin-bottom: 15px; text-align: right;">
            <Col span="24">
                <Button type="primary" @click="goBack">返回</Button>
            </Col>
        </Row>
        <serviceDetailCard v-on:getDetail="getDetail"></serviceDetailCard>
        <auditDetailCard v-if="serverData.status>1" :serverData="serverData" class="marginTop"></auditDetailCard>
        <dealDetailCard v-if="serverData.status==4" :serverData="serverData" class="marginTop"></dealDetailCard>
        <feedbackCard v-if="serverData.status==4 && serverData.score" :serverData="serverData" class="marginTop"></feedbackCard>
    </div>
</template>

<script>
import serviceDetailCard from "./serviceCard/serviceDetailCard.vue";
import auditDetailCard from "./serviceCard/auditDetailCard.vue";
import dealDetailCard from "./serviceCard/dealDetailCard.vue";
import feedbackCard from "./serviceCard/feedbackCard.vue";
export default {
	data(){
		return {
			serverData:{},
		}
	},
    components: {
        serviceDetailCard,
        auditDetailCard,
        dealDetailCard,
        feedbackCard
    },
    methods:{
    	getDetail(data){
    		this.serverData=data;
    	},
        goBack() {
            this.$router.go(-1)
        }
    }
}
</script>

<style lang='less' scoped>
.marginTop{
	margin-top: 20px;
}
.serviceDetail{
    background:#eee;
    padding: 20px;
    width:80%;
    height:auto;
}
</style>
