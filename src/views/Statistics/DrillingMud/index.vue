
<template>
  <div class="dashboard-container">
    <div class="dashboard-editor-container">
    <el-row :gutter="32">
      <el-col style="margin-top:40px">
        <h4 style="margin-top:10px;text-align: center">钻井液当量密度图</h4>
      </el-col>
      <el-col>
        <div ref="well" :class="className" :style="{height:height,width:width}" style="margin-top: 10px" />
      </el-col>
      <el-col>
        <div ref="well2" :class="className" :style="{height:height,width:width}" style="margin-top: 10px" />
      </el-col>
    </el-row>
  </div>
  </div>
</template>

<script>
import * as echarts from 'echarts'
import request from "@/utils/request";
require('echarts/theme/macarons') // echarts theme
import { debounce } from '@/utils'
import { getToken } from '@/utils/auth' // getToken from cookie


export default {
  props: {
    className: {
      type: String,
      default: 'chart'
    },
    width: {
      type: String,
      default: '100%'
    },
    height: {
      type: String,
      default: '1200px'
    }
  },
  data() {
    return {
      chart: null,
      data: [],
      chartData: [],
    }
  },
  watch: {
    data: {
      deep: true,
      handler(newVal) {
        if (newVal) {
          this.getChartData()
          this.initChart(1,this.chartData,this.$refs.well)
        }
      }
    }
  },
  mounted() {
    this.getLayerScipy()
    this.__resizeHandler = debounce(() => {
      if (this.chart) {
        this.chart.resize()
      }
    }, 100)
    window.addEventListener('resize', this.__resizeHandler)
  },
  beforeDestroy() {
    this.chart.dispose()
    this.chart = null
  },
  methods: {
    getLayerScipy() {
      var _this = this
      setTimeout(() => {
        request({
          url: 'statistics/drillingMud',
          method: 'get',
        }).then(res => {
          this.data = []
          this.data = res
          this.initChart(this.$refs.well)
        })
      }, 100)
    },
    initChart( myChart) {
      // let chartData = data
      let _this = this
      console.log('data',this.data)
      let series = [
        {
          name: '坍塌压力',
          type: 'line',
          smooth: 0,
          // data: this.data.map(function(item) {
          //   return item[1];
          // }),
          data: this.data['Collapse']
        },
        {
          name: '孔隙压力',
          type: 'line',
          smooth: 0,
          // data: this.data.map(function(item) {
          //   return item[1];
          // }),
          data: this.data['Pore']
        },
        {
          name: '漏失压力',
          type: 'line',
          smooth: 0,
          // data: this.data.map(function(item) {
          //   return item[1];
          // }),
          data: this.data['Leakage']
        },{
          name: '破裂压力',
          type: 'line',
          smooth: 0,
          // data: this.data.map(function(item) {
          //   return item[1];
          // }),
          data: this.data['Burst']
        }]
      let selected = {
      }
      Object.keys(this.data['wellDensity']).forEach(function(key,index){
          if(index > 0){
            selected[key + ':钻井液密度'] = false
          }
          series.push({
            name: key + ':钻井液密度',
            type: 'line',
            smooth: 0,
            data: _this.data['wellDensity'][key]
          })
        }
      )
      let option = {
        legend: {
          type: 'scroll',
          width: '80%',
          selected: selected
        },
        grid: {
          bottom:'25%'
        },
        toolbox: {
          show: true,
          orient: 'vertical',
          // 竖向居中
          top: '7%',
          right: '5%',
          feature: {
            dataZoom: {
              yAxisIndex: 'none'
            },
            dataView: {
              show: false,
              // readOnly: false
            },
            magicType: {type: ['line', 'bar']},
            restore: {},
            saveAsImage: {}
          }
        },
        yAxis: {
          type: 'value',
          // data: this.data.map(function(map){
          //   return map['Depth']
          // }),
          name: '深度(m)',
          inverse: true,
          axisPointer: {
            show: true
          },
          max: function (value) {
            return value.max + 100;
          },
          min: function (value) {
            return value.min;
          },
        },
        xAxis: [
          {
            type: 'value',
            name: '密度',
            position: 'top',
            boundaryGap: ['40%', '40%'],
            max: function (value) {
              return value.max + 0.1;
            },
            min: 0.9,
            minInterval: 0.2
          },
        ],
        series: series
      }
      this.chart = echarts.init(myChart,'macarons') // 'macarons'
      this.chart.setOption(option)
    }
  }
}
</script>
<style scoped>
.dashboard-editor-container {
  padding: 32px;
  /*background-color: rgb(240, 242, 245);*/
  position: relative;
}
.el-table .success-row{
  background:#f0f9eb;
}
.el-table td, .el-table th {
  padding: 3px 0;
}
/* 利用穿透，设置input边框隐藏 */
.inputDeep>>>.el-input__inner {
  border: 0;                /*去掉边框*/
  background: transparent;  /*input背景透明*/
}
/* 如果你的 el-input type 设置成textarea ，就要用这个了 */
.inputDeep>>>.el-textarea__inner {
  border: 0;
  resize: none; /* 这个是去掉 textarea 下面拉伸的那个标志，如下图 */
}
.disable-column .inputDeep>>>.el-input__inner {
  pointer-events: none;
}
</style>
