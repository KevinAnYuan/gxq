<!-- 计算资源面板 -->
<template>
  <Layout>
    <Content>
      <Breadcrumb>
        <BreadcrumbItem>计算资源面板</BreadcrumbItem>
      </Breadcrumb>
      <Card>
        <div class="card-container">
          <div class="card-list">
            <Card style="height:160px;background: #fff;color:#495060;">
              <div class="card-content" style="width:100%;">
                <p style="line-height: 42px;"><span style="color:#0099CC">CPU：</span>{{cpu }}核</p>
              </div>
              <div class="card-content">
                <p style="line-height: 42px;"><span>本月新增：</span>{{curMonAdd}}核</p>
              </div>
              <div class="card-content">
                <p style="line-height: 42px;"><span>新增占比：</span>{{addPercent}}%</p>
              </div>
              <div class="card-content">
                <p style="line-height: 42px;"><span>新增环比：</span>{{ciraddPercent}}%</p>
              </div>
              <div class="card-content">
                <p style="line-height: 42px;"><span>净增：</span>{{jadd}}GB</p>
              </div> 
              <div class="card-content">
                <p style="line-height: 42px;"><span>本月回收：</span>{{curMonSub}}核</p>
              </div>
              <div class="card-content">
                <p style="line-height: 42px;"><span>回收占比：</span>{{subPercent}}%</p>
              </div>
              <div class="card-content">
                <p style="line-height: 42px;"><span>回收环比：</span>{{revertPercent}}%</p>
              </div>
              <div class="card-content">
                <p style="line-height: 42px;"><span>净增占比：</span>{{jaddPercent}}%</p>
              </div>
            </Card>
          </div>
        </div>
        <Row type="flex" justify="space-between">
          <Col style="width: 49%">
            <chart-card title="各部门计算资源数量统计">
              <barChart ref="serviceNum">
                <div id="serviceNum" style="height: 540px;"></div>
              </barChart>
            </chart-card>
          </Col>
          <Col style="width: 49%">
            <chart-card title="各部门计算资源占比统计">
              <chartLine ref="databaseType" el="databaseType" :lineOption="databaseType">
                <div id="databaseType" style="height: 500px;"></div>
              </chartLine>
            </chart-card>
          </Col>
          <Col style="width: 100%">
            <chart-card title="前12个月计算资源数量统计">
              <chartLine ref="databaseCount" el="databaseCount" :lineOption="databaseCount">
                <div id="databaseCount" style="height: 400px;"></div>
              </chartLine>
            </chart-card>
          </Col>
        </Row>
      </Card>
    </Content>
  </Layout>
</template>

<script>
// echart图外层容器，包括卡片样式
import chartCard from '@/view/home/chartCard.vue'
import lineChart from '@/view/home/lineChart.vue'
import barChart from '@/view/home/barChart.vue'
import chartLine from './common/ChartLine.vue'
import api from '@/api/axiosApi'
import softhardApiList from '@/api/softhardApiList'
import {dataPie,multipleLine} from '@/assets/js/echartOption'
export default {
  components:{
    chartCard,
    lineChart,
    barChart,
    chartLine
  },
  data() {
    return {
      cpu :0,//数据库数量
      curMonAdd: 0,//新增
      curMonSub: 0,//回收
      addPercent: 0,//新增占比
      subPercent:0,//回收占比
      ciraddPercent: 0,//新增环比
      revertPercent:0,//回收环比
      jadd: 0,//净增
      jaddPercent: 0,//净增占比
      databaseType: dataPie(),
      databaseCount: multipleLine(),
    }
  },
  mounted () {
    this.init();
  },
  methods: {
    init(){//初始化
      this.getPanelCpuRes();
      this.getPanelMonthOrgCpuRes();
      this.getPanelOrgCpuRes();
    },
    getPanelCpuRes() {//查询计算资源面板cpu增比等情况
      api(softhardApiList.getPanelCpuRes).then((res) => {
        if(res.status == 200 && res.data.data) {
          let data = res.data.data;
          this.cpu = data.cpu ;
          this.curMonAdd = data.curMonAdd;
          this.curMonSub = data.curMonSub;
          this.addPercent = data.addPercent.toFixed(2);
          this.subPercent = data.subPercent.toFixed(2);
          this.ciraddPercent  = data.ciraddPercent.toFixed(2);
          this.revertPercent  = data.revertPercent.toFixed(2);
          this.jadd  = data.jadd;
          this.jaddPercent  = data.jaddPercent.toFixed(2);
        }
      }, (err) => {})
    },
    getPanelOrgCpuRes() {//查询各单位计算资源面板cpu等情况
      let dataVal=[],arr=[],name = [];
      api(softhardApiList.getPanelOrgCpuRes).then((res) => {
        if(res.status == 200 && res.data.data) {
          res.data.data.forEach(item => {
            name.push(item.orgname);
            arr.push(parseInt(item.cpuCount));
            dataVal.push({value:item.cpuCount,name:item.orgname});
          });
          this.drawServiceNum(name,arr)
          this.getDatabaseType(name,dataVal)
        }
      }, (err) => {})
    },
    getPanelMonthOrgCpuRes() {//查询各单位每月计算资源cpu情况
      let name =[],arr=[];
      api(softhardApiList.getPanelMonthOrgCpuRes).then((res) => {
        if(res.status == 200) {
          res.data.data.orgCpuRes.forEach(item => {
            name.push(item.orgname);
            arr.push({
              name: item.orgname,
              type: 'line',
              data: item.cpuCount
            });
          });
          this.getDatabaseCount(name,res.data.data.months,arr);
          // this.drawServiceNum(name,arr)
        }
      }, (err) => {})
    },
    drawServiceNum (name,arr) {
      const vm = this
      const opts = {
        el: 'serviceNum',
        legend: {
          left: 'center',
          bottom: 10,
          textStyle: {
            fontSize: '16px'
          },
          data: name
        },
        xAxis: {
          data: name,
          axisLabel: {
            interval:0,
            rotate:40
          }
        },
        yAxis: {
          name: '核数',
          min: 0,
          max: Math.max.apply(null, arr) + (Math.max.apply(null, arr)*0.2),
        },
        series: [{
          name: 'CPU核数',
          data: arr,
          barWidth : (arr.length > 12) ? 20 : 40,
          itemStyle:{
              normal:{//柱子颜色
                  color:'#4F81BD'
              }
          },
        }]
      }
      vm.$refs.serviceNum.refresh(opts)
    },
    getDatabaseType (legendName,dataVal) {
      this.databaseType.legend.data = legendName;
      this.databaseType.series[0].data = dataVal;
      this.databaseType.title.text = '各部门计算资源占比统计';
      this.$refs.databaseType.refresh()
    },
    getDatabaseCount (legendName,timeData,dataVal) {
      this.databaseCount.legend.data = legendName;
      this.databaseCount.xAxis.data = timeData;
      this.databaseCount.series = dataVal;
      // this.databaseCount.title.text = '各部门近12个月计算资源数量统计';
      this.$refs.databaseCount.refresh()
    },
  }
}
</script>

<style lang="less" scoped>
.card-container{
  &:after{
    content: '';
    clear: both;
    display: block;
  }
  .card-list{
    float: left;
    width: 100%;
    margin-left: 4%;
    text-align: center;
    &:nth-child(1){
      margin-left: 0;
    }
    .card-content{
      width:25%;
      float:left;
      display: inline-block;
      text-align: center;
      font-size: 18px;
      font-weight: bold;
      height: 42px;
    }
  }
}
</style>
