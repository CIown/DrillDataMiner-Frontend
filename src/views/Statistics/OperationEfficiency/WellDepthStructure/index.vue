<script src="../../../../../api/Statistics/BaseInfo/WellSummary.js"></script>
<template>
  <div class="dashboard-container">
    <div class="dashboard-editor-container">
    <el-row :gutter="32">
      <el-col>
        <h4 style="margin-top:10px;text-align: center">不同井身结构下的钻进时效</h4>
        <div>
          <span >井名 :</span>
          <el-select style="margin-top: 10px; margin-bottom: 10px" v-model="wellNameValue" placeholder="全部">
            <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" :disabled="item.disabled " @click.native="getWellLabel(item)">
            </el-option>
          </el-select>
        </div>
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
          </el-table-column>
        </el-table>
        <el-button v-on:click.native="downExcel()" style="margin-top:20px; float: right; margin-right: 10px;" type="primary" icon="el-icon-document">
          Export Excel
        </el-button>
      </el-col>
      <el-col style="margin-top: 10px">
        <div ref="well" :class="className" :style="{height:height,width:width}" />
      </el-col>
      <el-col style="margin-top: 10px">
        <div ref="chart2" :class="className" :style="{height:height,width:width}" />
      </el-col>
    </el-row>
    </div>
  </div>
</template>

<script>
import * as echarts from 'echarts'
require('echarts/theme/macarons') // echarts theme
import { debounce } from '@/utils'// 图像自适应
import request from "@/utils/request";
import fileDownload from "js-file-download";

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
        label: '区块',
        prop: 'fieldName'
      }, {
        label: '井号',
        prop: 'wellName'
      }, {
        label: '尺寸(in)',
        prop: 'szODDrill'
        },
        {
        label: '型号',
        prop: 'model'
      },
        {
          label: '入井深(m)',
          prop: 'depthincalc'
        },
        {
          label: '出井深(m)',
          prop: 'depthoutcalc'
        },
        {
          label: '进尺(m)',
          prop: 'depthdrilledjobcalc'
        },
        {
          label: '纯钻时间(h)',
          prop: 'tmdrilledjobcalc'
        },
        {
          label: 'ROP(m/h)',
          prop: 'ropcalc'
        }],
      data: [],
      // data: data1,
      chartData: [],
      WellName: [],
      // 自定义图表数据
      CustomChartData: [],
      SeriesBarNum: new Map(), // key: 尺寸SzODDrill的值; value:[ 每个尺寸的数量 DepthInCalc DepthOutCalcMean DepthDrilledCalc ] 可以继续加数据
      // 折线图数据
      DepthInCalcMean: [],
      DepthOutCalcMean: [],
      DepthDrilledJobCalc: [],
      CustomXAxis: [],
      colorList: ['rgb(46,199,201)', 'rgb(200,182,235)', 'rgb(90,177,239)', 'rgb(255,185,128)', 'rgb(216,122,128)', 'rgb(153,162,183)', '#75d822', '#e0ef85', '#dcef25',
        '#72b341', '#75d825', '#e0e478', '#dc00dc', '#79b862'],
      // options: [
      //   {
      //   value: '选项1',
      //   label: 'HZ26-6-1'
      // }, {
      //   value: '选项2',
      //   label: 'HZ26-6-2'
      // },{
      //   value: '选项3',
      //   label: 'HZ26-6-3',
      // },{
      //   value: '选项4',
      //   label: 'HZ26-6-4'
      // }, {
      //   value: '选项5',
      //   label: 'HZ26-6-4&Sa'
      // }, {
      //   value: '选项6',
      //   label: 'HZ26-6-5'
      // }, {
      //   value: '选项7',
      //   label: 'HZ26-6-6d'
      // }, {
      //   value: '选项8',
      //   label: 'HZ26-6-7'
      // }
      // ],
      options: [],
      wellNameValue: 'HZ26-6-1',
      filename: '钻进时效',
      fileIndex: 1,
    }
  },
  // watch: {
  //   data: {
  //     deep: true,
  //     handler() {
  //       this.chartData = this.getChartData()
  //       this.getCustomData()
  //       this.initChart()
  //       this.initChart2()
  //     }
  //   }
  // },
  mounted() {
    this.getWellName()
    this.getWellboreFormation(this.wellNameValue)
    // this.chartData = this.getChartData()

    this.__resizeHandler = debounce(() => {
      if (this.chart) {
        this.chart.resize()
        this.chart2.resize()
      }
    }, 100)
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
    getWellName(){
      var _this = this
      setTimeout(() => {
        request({
          url: 'api/getData/wellHeader',
          method: 'get',
        }).then(res => {
          console.log(11111111111,res)
          this.options = []
          for(var i = 0; i <　res.length; i++){
            var tmp = {}
            tmp['value'] = '选项' + (i + 1)
            tmp['label'] = res[i]['wellName']
            _this.options.push(tmp)
          }
        })
      }, 100)
    },
    getWellboreFormation(wellName){
      var _this = this
      setTimeout(() => {
        request({
          url: 'statistics/wellboreFormation',
          method: 'get',
          params:{
            wellName: wellName
          }
        }).then(res => {
          this.data = []
          this.chartData = []
          this.data = res
          this.chartData = this.getChartData()
          this.getCustomData()
          this.initChart()
          this.initChart2()
        })
      }, 100)
    },
    getWellLabel(item) {
      this.getWellboreFormation(item.label)
    },
    tableRowClassName({ row, rowIndex }) {
      if (rowIndex % 2 !== 1) {
        return 'success-row'
      }
      return ''
    },
    downExcel(){
      var _this = this
      setTimeout(() => {
        request({
          url: 'api/getData/toExcel',
          method: 'post',
          data: {
            tableData: JSON.stringify(_this.data)
          },
          responseType: 'arraybuffer'
        }).then(res => {
          var filename = this.filename + this.fileIndex
          fileDownload(res, filename + '.xlsx')
          this.fileIndex = this.fileIndex + 1
        })
      }, 100)
    },
    getChartData() {
      var Model = []
      var SzODDrill = []
      var DepthInCalc = []
      var DepthOutCalc = []
      var DepthDrilledJobCalc = []
      var TmDrillCalc = []
      var ROP = []
      var WellName = []
      this.data.forEach(function(value) {
        WellName.push(value.wellName)
        SzODDrill.push(value.szODDrill)
        Model.push(value.model)
        DepthInCalc.push(value.depthincalc)
        DepthOutCalc.push(value.depthoutcalc)
        DepthDrilledJobCalc.push(value.depthdrilledjobcalc)
        TmDrillCalc.push(value.tmdrilledjobcalc)
        ROP.push(value.ropcalc)
      })
      return {
        'WellName': WellName,
        'Model': Model,
        'SzODDrill': SzODDrill,
        'DepthInCalc': DepthInCalc,
        'DepthOutCalc': DepthOutCalc,
        'DepthDrilledJobCalc': DepthDrilledJobCalc,
        'TmDrillCalc': TmDrillCalc,
        'ROP': ROP
      }
    },
    getCustomData() {
      // 初始化每个数据
      this.SeriesBarNum = new Map()
      this.DepthInCalcMean = []
      this.DepthOutCalcMean = []
      this.DepthDrilledJobCalc = []
      this.CustomChartData = []
      this.CustomXAxis = []
      // foreach和map里面的this并不指向当前Vue实例
      var that = this
      // 计算每个尺寸SzODDrill对应的数量 和 DepthInCalcMean， DepthOutCalcMean， TmDrillCalcSum
      this.data.forEach(function(value) {
        if (that.SeriesBarNum.get(value.szODDrill) === undefined) {
          that.SeriesBarNum.set(value.szODDrill, [1, value.depthincalc, value.depthoutcalc, value.depthdrilledjobcalc])
          return
        }
        if (that.SeriesBarNum.get(value.szODDrill) !== undefined) {
          // 先算每个尺寸SzODDrill对应的DepthInCalc DepthOutCalc TmDrillCalc值的总和
          that.SeriesBarNum.set(value.szODDrill, [
            that.SeriesBarNum.get(value.szODDrill)[0] + 1,
            that.SeriesBarNum.get(value.szODDrill)[1] + value.depthincalc,
            that.SeriesBarNum.get(value.szODDrill)[2] + value.depthoutcalc,
            that.SeriesBarNum.get(value.szODDrill)[3] + value.depthdrilledjobcalc
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
        that.DepthDrilledJobCalc.push(value[3])
        that.CustomXAxis.push(key)
      })
      // 计算CustomChartData
      var seriesIndex = 0
      var barIndex = 1
      this.data.forEach(function(value, index) {
        // series的索引，y轴起始位置，值，每个series中bar的索引或者是值的索引，每个series画几个bar
        if (index >= 1) {
          if (value.szODDrill !== that.data[index - 1 ].szODDrill) {
            seriesIndex += 1
            barIndex = 1
          }
        }
        // CustomChartData 分为五个维度，分别为{系列项}（从0开始 ）、y轴起始值(均为0)、实际值(此处为入井深DepthInCalc)、同系列中索引值（从1开始）、同系列数据项总数（每个系列需要画的柱子总数）
        that.CustomChartData.push([seriesIndex, 0, value.depthincalc, barIndex, that.SeriesBarNum.get(value.szODDrill)[0]])
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
      let width = api.size([0, api.value(2)])[0] * 0.5

      const num = api.value(4) // 每个系列柱子数
      const currentIndex = api.value(3)
      const isOdd = num % 2 === 0
      const midN = isOdd ? num / 2 : (num + 1) / 2

      var rect = ''

      width = width / num

      let rectX = start[0] - width / 2

      const FIXED_WIDTH = 5 // 柱子间距

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
      let oldUpdateData = this.chartData
      this.chart = echarts.init(this.$refs.well, 'macarons')
      this.chart.setOption({
        // backgroundColor: '#0f375f',
        tooltip: {
          trigger: 'item',
          axisPointer: {
            type: 'shadow'
          },
          formatter: function(params) {
            return params.marker + params.name + ': ' + params.value + ' m'
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
          },
          //show: true
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
          },
        {
          name: '出井深',
          type: 'bar',
          data: oldUpdateData.DepthOutCalc,
        }, {
          name: '进尺',
          type: 'bar',
          data: oldUpdateData.DepthDrilledJobCalc,
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
          }
           ]
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
        toolbox: {
          show: true,
          // orient: 'vertical',
          // 竖向居中
          // top: '7%',
          // right: '5%',
          feature: {
            dataZoom: {
              yAxisIndex: 'none'
            },
            dataView: {readOnly: false},
            magicType: {type: ['line', 'bar']},
            restore: {},
            saveAsImage: {}
          }
        },
        legend:{},
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
            name: '入井深',
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
            name: '总入井深',
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
            name: '总进尺',
            itemStyle: {
              opacity: 0.5,
              color: 'red'
            },
            data: this.DepthDrilledJobCalc,
            yAxisIndex: 1
          }]
      })
    }
  }
}
</script>
<style>
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
.el-dropdown-link {
  cursor: pointer;
  color: #409EFF;
}
.el-icon-arrow-down {
  font-size: 12px;
}
</style>

