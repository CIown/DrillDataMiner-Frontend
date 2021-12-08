<template>
  <div class="dashboard-container">
    <div class="dashboard-editor-container">
    <el-row :gutter="32">
      <el-col style="margin-top:40px">
        <h4 style="margin-top:10px;text-align: center">井身结构深度示意图</h4>
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
import request from "../../../utils/request";
require('echarts/theme/macarons') // echarts theme
import { debounce } from '@/utils'

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
      default: '600px'
    }
  },
  data() {
    return {
      isReadonly: true,
      tilte: '水深分析图',
      chart: null,
      isReadonlyFlag: {},
      data: [],
      chartData: undefined,
      downloadLoading: false,
      learnParams: undefined,
      formDepth:undefined
    }
  },
  // watch: {
  //   data: {
  //     deep: true,
  //     handler(newVal) {
  //       if (newVal) {
  //         this.getChartData()
  //         this.initChart(1,this.chartData,this.$refs.well,this.title)
  //       }
  //     }
  //   }
  // },
  mounted() {
    this.getLayerScipy(this.value)
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
          url: 'statistics/waterDepth',
          method: 'get',
        }).then(res => {
          this.data = []
          this.data = res['waterDepthData']
          this.formDepth = res['formDepth']
          this.getChartData()
          this.initChart(1,this.chartData,this.$refs.well,this.title)
        })
      }, 100)
    },
    getWellLabel(item){
      this.getLayerScipy(item.label)
    },
    handleFit(flag){
      this.getRefit(flag)
    },
    tableRowClassName({ row, rowIndex }) {
      if (rowIndex % 2 !== 1) {
        return 'success-row' + ' row' + rowIndex
      }
      return ''
    },
    tableColumnClassName({flag,row, column, rowIndex, columnIndex}){
      if(columnIndex === 4 || columnIndex === 5){
        return 'disable-column'
      }
      return ''
    },
    handle(flag,index, row, key, value){
    },
    handleEdit(flag,index, row, key, value) {
      for (value in this.isReadonlyFlag){
        this.isReadonlyFlag[value] = true
      }
      this.$set(this.isReadonlyFlag,'flag' + index, false);
    },
    handleDelete(flag,index, row, key, value) {
    },
    handleDeleteAll(flag){
    },

    handleAdd(flag) {
    },
    handleDeleteNewTableAndChart() {
    },
    handleAddNewTableAndChart() {
    },
    getChartData(flag) {
      var _this = this
      this.chartData = {}
      this.data.forEach(function(value) {
        if((value.wellName in _this.chartData) === true){
          _this.chartData[value.wellName].push({
            'sz':value.sz,
            'waterDepth': value.waterDepth,
            'fieldName': value.fieldName,
            'depthBtmActual': value.depthBtmActual
          })
        }else{
          _this.chartData[value.wellName] = []
          _this.chartData[value.wellName].push({
              'sz': value.sz,
              'waterDepth': value.waterDepth,
              'fieldName': value.fieldName,
              'depthBtmActual': value.depthBtmActual
            })
         }
      })
      // 按sz的大小排序
      Object.keys(this.chartData).forEach(function(key){
        _this.chartData[key].sort()
      })
      console.log('this.chartData',this.chartData)
    },

    initChart(flag,data, myChart,text) {
      console.log(this.formDepth)
      let chartData = data
      var seriesData = []
      var dimensionName = []
      var sourceData = []
      var waterDepth = []
      var szValue = []
      var formColors = {
        '粤海组': 'rgba(189,215,238)',
        '韩江组': 'rgba(248,203,173)',
        '珠江组': 'rgba(219,219,219)',
        '珠海组': 'rgba(255,192,0)',
        '恩平组': 'rgba(68,114,196)',
        '文昌组': 'rgba(153,115,0)',
      }
      // 维度名为 井名 各个尺寸的大小（每个不同尺寸对应不同的井深）
      dimensionName = ['wellName']
      var keySet = Object.keys(chartData)
      for(var i in keySet) {
        // key 是井名
        var key = keySet[i]
        waterDepth.push(chartData[key][0]['waterDepth'])
      }

      for(var i in keySet){
        // key 是井名
        var key = keySet[i]
        var tmp = {'wellName': key}
        chartData[key].forEach(function(value){
          szValue.push(value.sz)
          tmp['尺寸-' + value.sz] = value.depthBtmActual
        })
        sourceData.push(tmp)
      }
      // szValue去重
      szValue = szValue.sort()
      var array= [szValue[0]];
      for (var i = 1; i < szValue.length; i++) {
        if (szValue[i] !== szValue[i-1]) {
          array.push(szValue[i]);
        }
      }
      szValue = array

      szValue.forEach(function(value){
        dimensionName.push('尺寸-'+value)
      })
      var legendData = dimensionName.slice(1)
      // legendData.unshift(0,'水深')


      for(var i = 1; i < dimensionName.length; i++){
        // 辅助柱子 使柱状图浮空
        seriesData.push({
          name: i,
          type: 'bar',
          stack: i,
          data: waterDepth,
          itemStyle: {
            barBorderColor: 'rgba(0,0,0,0)',
            color: 'rgba(0,0,0,0)'
          },
          emphasis: {
            itemStyle: {
              barBorderColor: 'rgba(0,0,0,0)',
              color: 'rgba(0,0,0,0)',
              show: false,
            }
          },
          tooltip: {
            trigger: 'item',
            axisPointer: {
              type: 'shadow'
            },
            formatter: function (params) {
              return '';
            }
          },
        })
        // 真实柱子
        seriesData.push({
          name: dimensionName[i],
          type: 'bar',
          stack: i,
        })
      }
      seriesData.push({
        name: '水深',
        type: 'line',
        stack: 'Total',
        lineStyle: {
          width: 0
        },
        areaStyle: {
          color: 'rgba(0, 221, 255)'
        },
        emphasis: {
          focus: 'series'
        },
        data: waterDepth,
        step: 'middle '
      })
      legendData.unshift('水深')
      this.formDepth.forEach(function(value){
        legendData.unshift(Object.keys(value)[0])
        seriesData.push({
          name: Object.keys(value)[0],
          type: 'line',
          stack: 'Total',
          lineStyle: {
            width: 0
          },
          areaStyle: {
            color: formColors[Object.keys(value)[0]]
          },
          emphasis: {
            show: true,
            focus: 'series'
          },
          // emphasis:　{
          //   show: false,
          // },
          data: Object.values(value)[0],
          // step: 'middle '
        })
      })
      // 配置项
      var option = {
        // backgroundColor: '#0f375f',
        tooltip: {
          trigger: 'item',
          axisPointer: {
            type: 'shadow'
          },
        },
        legend: {
          data: legendData
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
              readOnly: false
            },
            magicType: {type: ['line', 'bar']},
            restore: {},
            saveAsImage: {}
          }
        },
        dataset: {
          dimensions: dimensionName,
          source: sourceData
        },
        grid:{
          top: '15%'
        },
        xAxis: {
          position: 'top',
          type: 'category',
          axisLabel: {
            // formatter:function(value){
            //   return value.split("").join("\n");
            // },
            interval: 0,
            rotate: -40,
            margin: 10
          },

        },
        yAxis: [{
          name: 'm',
          // y翻转
          inverse: true,
          splitLine: { show: false },
          type: 'value',
          axisLabel: {
            show: true,
            interval: 'auto'
          },
          axisLine: {
            lineStyle: {
              color: '#ccc'
            }
          }
        },
          // {
          //   name: 'h',
          //   splitLine: { show: false },
          //   type: 'value',
          //   axisLabel: {
          //     show: true,
          //     interval: 'auto'
          //   },
          //   axisLine: {
          //     lineStyle: {
          //       color: '#ccc'
          //     }
          //   }
          // }
            ],
        series: seriesData,

      }
      if(flag === 1 ){
        this.chart = echarts.init(myChart,'macarons') // 'macarons'
        this.chart.setOption(option)
      }else{
        this.chart2 = echarts.init(myChart,'macarons');
        this.chart2.setOption(option)
      }
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
