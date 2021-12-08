<script src="../../../../../api/Statistics/BaseInfo/WellSummary.js"></script>
<template>
  <div>
    <el-row :gutter="32">
      <el-col>
        <h4 style="margin-top:0px;text-align: center">不同井身结构下的钻进时效</h4>
        <el-table
          v-if="data.length> 0"
          :data="data"
          :row-class-name="tableRowClassName"
          style="width: 100%; margin-top: 20px"
          border
        >
          <el-table-column
            v-for="(item) in activeWellDataPropLabelArray"
            :key="item.prop"
            align="left"
            :prop="item.prop"
            :label="item.label"
          >
            <!--
            <template slot-scope="scope">
              {{ scope.row[item.prop]?scope.row[item.prop]:'暂无数据' }}
            </template>
            -->
            <template slot-scope="scope">
              <!--
              <el-input v-model="scope.row[item.prop]" size="small" placeholder="请输入内容" @change="handleEdit(scope.$index, scope.row,item.prop,scope.row[item.prop])" />
              -->
              {{ scope.row[item.prop] }}
            </template>
          </el-table-column></el-table>
      </el-col>
      <el-col style="margin-top: 10px">
        <div ref="well" :class="className" :style="{height:height,width:width}" />
      </el-col>
      <el-col style="margin-top: 10px">
        <div ref="chart2" :class="className" :style="{height:height,width:width}" />
      </el-col>
    </el-row></div>
</template>

<script>
import * as echarts from 'echarts'
require('echarts/theme/macarons') // echarts theme
import { debounce } from '@/utils'// 图像自适应
import axios from 'axios'

const animationDuration = 6000
const origin_data = [
  {
    'WellName': 'ABCDEFG-1',
    'SzODDrill': 12.25,
    'Model': 'KSD1952SGR',
    'DepthInCalc': 1027,
    'DepthOutCalc': 3305.7,
    'DepthDrilledCalc': 2278.7,
    'TmDrillCalc': 47.72,
    'ROP': 47.75
  },
  {
    'WellName': 'ABCDEFG-1',
    'SzODDrill': 12.25,
    'Model': 'HJT537',
    'DepthInCalc': 3305.7,
    'DepthOutCalc': 3423,
    'DepthDrilledCalc': 117.3,
    'TmDrillCalc': 27.21,
    'ROP': 4.31
  },
  {
    'WellName': 'ABCDEFG-1',
    'SzODDrill': 12.25,
    'Model': 'HP636MI',
    'DepthInCalc': 3423,
    'DepthOutCalc': 3744.26,
    'DepthDrilledCalc': 321.26,
    'TmDrillCalc': 50.86,
    'ROP': 6.32
  },
  {
    'WellName': 'ABCDEFG-1',
    'SzODDrill': 8.5,
    'Model': 'U613M',
    'DepthInCalc': 3744.26,
    'DepthOutCalc': 3818.66,
    'DepthDrilledCalc': 72.4,
    'TmDrillCalc': 10.35,
    'ROP': 7
  },
  {
    'WellName': 'ABCDEFG-1',
    'SzODDrill': 8.5,
    'Model': 'U613M',
    'DepthInCalc': 3818.66,
    'DepthOutCalc': 3885.45,
    'DepthDrilledCalc': 68.79,
    'TmDrillCalc': 33.19,
    'ROP': 2.07
  },
  {
    'WellName': 'ABCDEFG-1',
    'SzODDrill': 8.5,
    'Model': 'DA636ME',
    'DepthInCalc': 3885.45,
    'DepthOutCalc': 4383.31,
    'DepthDrilledCalc': 86.31,
    'TmDrillCalc': 28.61,
    'ROP': 3
  },
  {
    'WellName': 'ABCDEFG-1',
    'SzODDrill': 8.5,
    'Model': 'R616',
    'DepthInCalc': 3971.76,
    'DepthOutCalc': 4128,
    'DepthDrilledCalc': 156.24,
    'TmDrillCalc': 35.72,
    'ROP': 4.4
  },
  {
    'WellName': 'ABCDEFG-1',
    'SzODDrill': 8.5,
    'Model': 'DA336ME',
    'DepthInCalc': 4128,
    'DepthOutCalc': 4236,
    'DepthDrilledCalc': 108,
    'TmDrillCalc': 35.15,
    'ROP': 3.07
  },
  {
    'WellName': 'ABCDEFG-1',
    'SzODDrill': 8.5,
    'Model': 'U613M',
    'DepthInCalc': 4236,
    'DepthOutCalc': 4276,
    'DepthDrilledCalc': 40,
    'TmDrillCalc': 8.33,
    'ROP': 4.8
  },
  {
    'WellName': 'ABCDEFG-1',
    'SzODDrill': 11,
    'Model': 'U613M',
    'DepthInCalc': 4236,
    'DepthOutCalc': 4276,
    'DepthDrilledCalc': 20,
    'TmDrillCalc': 8.33,
    'ROP': 4.8
  },
  {
    'WellName': 'ABCDEFG-1',
    'SzODDrill': 11,
    'Model': 'U613M',
    'DepthInCalc': 4236,
    'DepthOutCalc': 4276,
    'DepthDrilledCalc': 50,
    'TmDrillCalc': 8.33,
    'ROP': 4.8
  },
  {
    'WellName': 'ABCDEFG-1',
    'SzODDrill': 11,
    'Model': 'U613M',
    'DepthInCalc': 4236,
    'DepthOutCalc': 4276,
    'DepthDrilledCalc': 40,
    'TmDrillCalc': 8.33,
    'ROP': 4.8
  },
  {
    'WellName': 'ABCDEFG-1',
    'SzODDrill': 11,
    'Model': 'U613M',
    'DepthInCalc': 4236,
    'DepthOutCalc': 4276,
    'DepthDrilledCalc': 10,
    'TmDrillCalc': 8.33,
    'ROP': 4.8
  },
  {
    'WellName': 'ABCDEFG-1',
    'SzODDrill': 11,
    'Model': 'U613M',
    'DepthInCalc': 4236,
    'DepthOutCalc': 4276,
    'DepthDrilledCalc': 10,
    'TmDrillCalc': 8.33,
    'ROP': 4.8
  }
]
/*
var WellName = []
var CustomChartData = []
var SeriesBarNum = new Map() // key: 尺寸SzODDrill的值; value:[ 每个尺寸的数量 DepthInCalc DepthOutCalcMean DepthDrilledCalc ]
// 折线图数据
var DepthInCalcMean = []
var DepthOutCalcMean = []
var DepthDrilledCalc = []
var CustomXAxis = []

// 计算每个尺寸SzODDrill对应的数量 和 DepthInCalcMean， DepthOutCalcMean， TmDrillCalcSum

data.forEach(function(value) {
  if (SeriesBarNum.get(value.SzODDrill) === undefined) {
    SeriesBarNum.set(value.SzODDrill, [1, value.DepthInCalc, value.DepthOutCalc, value.DepthDrilledCalc])
    return
  }
  if (SeriesBarNum.get(value.SzODDrill) !== undefined) {
    // 先算每个尺寸SzODDrill对应的DepthInCalc DepthOutCalc TmDrillCalc值的总和
    SeriesBarNum.set(value.SzODDrill, [
      SeriesBarNum.get(value.SzODDrill)[0] + 1,
      SeriesBarNum.get(value.SzODDrill)[1] + value.DepthInCalc,
      SeriesBarNum.get(value.SzODDrill)[2] + value.DepthOutCalc,
      SeriesBarNum.get(value.SzODDrill)[3] + value.DepthDrilledCalc
    ])
    return
  }
})

// 计算DepthInCalcMean DepthOutCalcMean
SeriesBarNum.forEach(function(value, key) {
  var num = value[0]
  SeriesBarNum.set(key, [value[0], value[1] / num, value[2] / num, value[3]])
  DepthInCalcMean.push(value[1])
  DepthOutCalcMean.push(value[2])
  DepthDrilledCalc.push(value[3])
  CustomXAxis.push(key)
})

var seriesIndex = 0
var barIndex = 1
data.forEach(function(value, index) {
  // series的索引，y轴起始位置，值，每个series中bar的索引或者是值的索引，每个series画几个bar
  if (index >= 1) {
    if (value.SzODDrill !== data[index - 1 ].SzODDrill) {
      seriesIndex += 1
      barIndex = 1
    }
  }
  CustomChartData.push([seriesIndex, 0, value.DepthInCalc, barIndex, SeriesBarNum.get(value.SzODDrill)[0]])
  barIndex += 1
})

var data = []
data.push([0, 0, 120, 1, 5])
data.push([0, 0, 220, 2, 5])
data.push([0, 0, 150, 3, 5])

data.push([0, 0, 200, 4, 5])

data.push([0, 0, 200, 5, 5])
// data.push({ itemStyle: { normal: { color: '#715881' } }, name: '2011', value: [0, 0, 200, 5, 5] })
data.push([1, 0, 250, 1, 3])
data.push([1, 0, 280, 2, 3])
data.push([1, 0, 280, 3, 3])
// data.push({ itemStyle: { normal: { color: 'lightgreen' } }, name: '2012', value: [0, 150, 350, 200] })
// data.push({ itemStyle: { normal: { color: 'lightgreen' } }, name: '2012', value: [1, 250, 320, 200] })
data.push([2, 0, 330, 1, 1])
data.push([3, 0, 430, 1, 2])
data.push([3, 0, 230, 2, 2])

var colorList = ['rgb(46,199,201)', 'rgb(200,182,235)', 'rgb(90,177,239)', 'rgb(255,185,128)', 'rgb(216,122,128)', 'rgb(153,162,183)', '#75d822', '#e0ef85', '#dcef25',
  '#72b341', '#75d825', '#e0e478', '#dc00dc', '#79b862']

CustomChartData = CustomChartData.map(function(item, index) {
  return {
    name: index + '',
    value: item,
    itemStyle: {
      color: colorList[item[3]]
    }
  }
})
*/
//  图表联动 表格编辑后改动的是welltabledata里面的数据

// Generate data
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
      default: '400px'
    }
  },
  data() {
    return {
      chart: null,
      chart2: null,
      activeWellDataPropLabelArray: [{
        label: '井号',
        prop: 'WellName'
      }, {
        label: '尺寸',
        prop: 'SzODDrill'
      }, {
        label: '型号',
        prop: 'Model'
      },
        {
          label: '入井深',
          prop: 'DepthInCalc'
        },
        {
          label: '出井深',
          prop: 'DepthOutCalc'
        },
        {
          label: '进尺',
          prop: 'DepthDrilledCalc'
        },
        {
          label: '纯钻时间',
          prop: 'TmDrillCalc'
        },
        {
          label: 'ROP',
          prop: 'ROP'
        }],
      // data: [],
      data: origin_data,
      chartData: [],
      WellName: [],
      // 自定义图表数据
      CustomChartData: [],
      SeriesBarNum: new Map(), // key: 尺寸SzODDrill的值; value:[ 每个尺寸的数量 DepthInCalc DepthOutCalcMean DepthDrilledCalc ] 可以继续加数据
      // 折线图数据
      DepthInCalcMean: [],
      DepthOutCalcMean: [],
      DepthDrilledCalc: [],
      CustomXAxis: [],
      colorList: ['rgb(46,199,201)', 'rgb(200,182,235)', 'rgb(90,177,239)', 'rgb(255,185,128)', 'rgb(216,122,128)', 'rgb(153,162,183)', '#75d822', '#e0ef85', '#dcef25',
        '#72b341', '#75d825', '#e0e478', '#dc00dc', '#79b862']
    }
  },
  watch: {
    data: {
      deep: true,
      handler() {
        this.chartData = this.getChartData()
        this.getCustomData()
        this.initChart()
        this.initChart2()
      }
    }
  },
  mounted() {
    /**
     axios.get('http://127.0.0.1:9000/api/WellboreFormation')
     .then(response => {
        this.data = response.data[0].data
        this.chartData = this.getChartData()
        this.getCustomData()
        this.initChart()
        this.initChart2()
      })
     **/
    this.chartData = this.getChartData()
    this.getCustomData()
    this.initChart()
    this.initChart2()
    this.__resizeHandler = debounce(() => {
      if (this.chart) {
        this.chart.resize()
        this.chart2.resize()
      }
    }, 100)
    this.handleEdit()
    window.addEventListener('resize', this.__resizeHandler)
  },
  beforeDestroy() {
    if (!this.chart) {
      return
    }
    window.removeEventListener('resize', this.__resizeHandler)
    this.chart.dispose()
    this.chart = null
    this.chart2.dispose()
    this.chart2 = null
  },
  methods: {
    /**
     handleEdit(index, row, key, value) {
      console.log(index, row, key, value)
    },
     **/
    tableRowClassName({ row, rowIndex }) {
      if (rowIndex % 2 !== 1) {
        return 'success-row'
      }
      return ''
    },
    getChartData() {
      var Model = []
      var SzODDrill = []
      var DepthInCalc = []
      var DepthOutCalc = []
      var DepthDrilledCalc = []
      var TmDrillCalc = []
      var ROP = []
      this.data.forEach(function(value) {
        SzODDrill.push(value.SzODDrill)
        Model.push(value.Model)
        DepthInCalc.push(value.DepthInCalc)
        DepthOutCalc.push(value.DepthOutCalc)
        DepthDrilledCalc.push(value.DepthDrilledCalc)
        TmDrillCalc.push(value.TmDrillCalc)
        ROP.push(value.ROP)
      })
      return {
        'Model': Model,
        'SzODDrill': SzODDrill,
        'DepthInCalc': DepthInCalc,
        'DepthOutCalc': DepthOutCalc,
        'DepthDrilledCalc': DepthDrilledCalc,
        'TmDrillCalc': TmDrillCalc,
        'ROP': ROP
      }
    },
    getCustomData() {
      // 初始化每个数据
      this.SeriesBarNum = new Map()
      this.DepthInCalcMean = []
      this.DepthOutCalcMean = []
      this.DepthDrilledCalc = []
      this.CustomChartData = []
      this.CustomXAxis = []
      // foreach和map里面的this并不指向当前Vue实例
      var that = this
      // 计算每个尺寸SzODDrill对应的数量 和 DepthInCalcMean， DepthOutCalcMean， TmDrillCalcSum
      this.data.forEach(function(value) {
        if (that.SeriesBarNum.get(value.SzODDrill) === undefined) {
          that.SeriesBarNum.set(value.SzODDrill, [1, value.DepthInCalc, value.DepthOutCalc, value.DepthDrilledCalc])
          return
        }
        if (that.SeriesBarNum.get(value.SzODDrill) !== undefined) {
          // 先算每个尺寸SzODDrill对应的DepthInCalc DepthOutCalc TmDrillCalc值的总和
          that.SeriesBarNum.set(value.SzODDrill, [
            that.SeriesBarNum.get(value.SzODDrill)[0] + 1,
            that.SeriesBarNum.get(value.SzODDrill)[1] + value.DepthInCalc,
            that.SeriesBarNum.get(value.SzODDrill)[2] + value.DepthOutCalc,
            that.SeriesBarNum.get(value.SzODDrill)[3] + value.DepthDrilledCalc
          ])
          return
        }
      })
      // 计算DepthInCalcMean DepthOutCalcMean 可以加数据
      this.SeriesBarNum.forEach(function(value, key) {
        var num = value[0]
        that.SeriesBarNum.set(key, [value[0], value[1] / num, value[2] / num, value[3]])
        that.DepthInCalcMean.push(value[1])
        that.DepthOutCalcMean.push(value[2])
        that.DepthDrilledCalc.push(value[3])
        that.CustomXAxis.push(key)
      })
      // 计算CustomChartData
      var seriesIndex = 0
      var barIndex = 1
      this.data.forEach(function(value, index) {
        // series的索引，y轴起始位置，值，每个series中bar的索引或者是值的索引，每个series画几个bar
        if (index >= 1) {
          if (value.SzODDrill !== that.data[index - 1 ].SzODDrill) {
            seriesIndex += 1
            barIndex = 1
          }
        }
        // CustomChartData 分为五个维度，分别为{系列项}（从0开始 ）、y轴起始值(均为0)、实际值(此处为入井深DepthInCalc)、同系列中索引值（从1开始）、同系列数据项总数（每个系列需要画的柱子总数）
        that.CustomChartData.push([seriesIndex, 0, value.DepthInCalc, barIndex, that.SeriesBarNum.get(value.SzODDrill)[0]])
        barIndex += 1
      })
      this.CustomChartData = this.CustomChartData.map(function(item) {
        return {
          name: '进尺',
          value: item,
          itemStyle: {
            color: that.colorList[item[3]]
          }
        }
      })
    },
    renderItem: function(params, api) {
      this.colorIndex = this.colorIndex + 1
      const categoryIndex = api.value(0)
      const start = api.coord([categoryIndex, api.value(1)])
      const end = api.coord([categoryIndex, api.value(2)])
      let width = api.size([0, api.value(2)])[0] * 0.8

      const num = api.value(4) // 每个系列柱子数
      const currentIndex = api.value(3)
      const isOdd = num % 2 === 0
      const midN = isOdd ? num / 2 : (num + 1) / 2

      var rect = ''

      width = width / num

      let rectX = start[0] - width / 2

      const FIXED_WIDTH = 0 // 柱子间距

      // 数据处理，结构为 { itemStyle: { normal: { color: 'lightgreen' } }, name: '2011', value: [0, 0, 150, 2, 5] }
      // 其中value 分为五个维度，分别为{系列项}（从0开始）、y轴起始值(均为0)、实际值、同系列中索引值（从1开始）、同系列数据项总数

      if (num > 1) {
        if (isOdd) {
          if (currentIndex <= midN) {
            // 中位数左侧
            rectX =
              start[0] - width / 2 - width / 2 + (currentIndex - midN) * width - FIXED_WIDTH * (midN + 1 - currentIndex)
          } else {
            // 中位数右侧
            rectX =
              start[0] - width / 2 + width / 2 + (currentIndex - midN - 1) * width + FIXED_WIDTH * (currentIndex - midN)
          }
        } else {
          rectX = start[0] - width / 2 + (currentIndex - midN) * (width + FIXED_WIDTH)
        }
      }

      rect = {
        type: 'rect',
        shape: echarts.graphic.clipRectByRect(
          { x: rectX, y: end[1], width: width, height: start[1] - end[1] },
          {
            x: params.coordSys.x,
            y: params.coordSys.y,
            width: params.coordSys.width,
            height: params.coordSys.height
          }
        ),
        style: api.style()
      }

      return rect
    },
    initChart() {
      var oldUpdateData = this.chartData
      this.chart = echarts.init(this.$refs.well, 'macarons')
      this.chart.setOption({
        // backgroundColor: '#0f375f',
        tooltip: {
          trigger: 'item',
          axisPointer: {
            type: 'shadow'
          }
        },
        legend: {
          data: ['入井深', '出井深', '进尺', '纯钻时间', 'ROP'],
          textStyle: {
            color: '#ccc'
          }
        },
        label: {
          position: 'top',
          fontWeight: 'lighter',
          fontSize: 10,
          formatter: function(param) {
            return param.data[0]
          }
          // show: true
        },
        xAxis: {
          data: oldUpdateData.SzODDrill,
          name: 'SzODDrill',
          nameLocation: 'middle',
          // 名称和轴的距离
          nameGap: 30,
          axisLine: {
            lineStyle: {
              color: '#ccc'
            }
          },
          axisTick: {
            alignWithLabel: true
          }
        },
        yAxis: [{
          name: 'm',
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
          {
            name: 'h',
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
          }],
        series: [{
          name: '入井深',
          type: 'bar',
          // showAllSymbol: true,
          // symbol: 'emptyCircle',
          // symbolSize: 15,
          data: oldUpdateData.DepthInCalc,
          animationDuration
        }, {
          name: '出井深',
          type: 'bar',
          data: oldUpdateData.DepthOutCalc,
          animationDuration
        }, {
          name: '进尺',
          type: 'bar',
          data: oldUpdateData.DepthDrilledCalc,
          animationDuration
        }, {
          name: '纯钻时间',
          type: 'line',
          symbol: 'rect',
          data: oldUpdateData.TmDrillCalc,
          yAxisIndex: 1
        },
          {
            name: 'ROP',
            type: 'line',
            symbol: 'rect',
            data: oldUpdateData.ROP,
            yAxisIndex: 1
          }]
      })
    },
    initChart2() {
      this.chart2 = echarts.init(this.$refs.chart2, 'macarons')
      this.chart2.setOption({
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'shadow'
          }
        },
        xAxis: [{
          type: 'category',
          name: 'SzODDrill',
          nameLocation: 'middle',
          // 名称和轴的距离
          nameGap: 30,
          data: this.CustomXAxis,
          splitLine: { show: true },
          axisLabel: {
            show: true,
            interval: 'auto'
          },
          axisLine: {
            lineStyle: {
              color: '#ccc'
            }
          }
        }
          /* {
              type: 'category',
              data: ['系列1', '系列2', '系列3', '系列4'],
              splitArea: {},
              splitLine: { show: true }
            }*/],
        yAxis: [{
          type: 'value',
          name: 'm',
          axisLabel: {
            show: true,
            interval: 'auto',
            name: 'y'
          },
          axisLine: {
            lineStyle: {
              color: '#ccc'
            }
          }
        }, {
          name: 'h',
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
        }],
        series: [
          {
            type: 'custom',
            renderItem: this.renderItem,
            itemStyle: {
              normal: {
                opacity: 2
              }
            },
            encode: {
              // y 对应CustomChartData的第1, 2 列（维度）
              y: [1, 2],
              // x对应CustomChartData的第0 列（维度）
              x: 0
            },
            data: this.CustomChartData
          },
          {
            type: 'line',
            animation: false,
            name: 'DepthInCalcMean',
            itemStyle: {
              opacity: 0.5,
              color: 'red'
            },
            data: this.DepthInCalcMean,
            yAxisIndex: 1
          },
          {
            type: 'line',
            animation: false,
            name: 'DepthDrilledCalc',
            itemStyle: {
              opacity: 0.5,
              color: 'red'
            },
            data: this.DepthDrilledCalc,
            yAxisIndex: 1
          }]
      })
    }
  }
}
</script>
<style>
.el-table .success-row{
  background:#f0f9eb;
}
.el-table td, .el-table th {
  padding: 3px 0;
}

</style>

