<!-- 柱状图 -->
<template>
  <div>
    <slot></slot>
  </div>
</template>

<script>
export default {
  props: {
    barOption: {
      type: Object,
      default: () => {
        return {}
      }
    },
    legendDataBar:{
      type: Array,
      default: () => {
        return []
      }
    },
    el: {
      type: String,
      required: true
    },
    title: {
      type: String,
      default: ''
    }
  },
  methods: {
    refresh () {
      const vm = this
      const option = {
        color: ['#487EC1', '#48B9C1', '#A27DE2', '#32C0D6'],
        title: {
          text: vm.title
        },
        tooltip : {
          trigger: 'axis',
          axisPointer : {            // 坐标轴指示器，坐标轴触发有效
            type : 'shadow'        // 默认为直线，可选为：'line' | 'shadow'
          }
        },
        grid: {
          left: '60px',
          right: '60px',
          bottom: '60px',
          containLabel: true
        },
        ...vm.barOption
      }
      const line = vm.$echarts.init(document.getElementById(vm.el))
      line.setOption(option)
    }
  }
}
</script>
