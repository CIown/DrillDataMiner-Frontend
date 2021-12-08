<template>
  <div class="dashboard-container">
    <div class="dashboard-editor-container">
    <el-row :gutter="32">
      <el-col>
        <h4 style="margin-top:10px; text-align: center">钻遇地层情况</h4>
<!--        <el-select style="margin-top: 10px; margin-bottom: 10px" v-model="value" placeholder="请选择区块">-->
<!--          <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value">-->
<!--          </el-option>-->
<!--        </el-select>-->
        <span>单位:m</span>
        <el-table
          v-if="data.length> 0"
          :data="data"
          border
          style="width: 100%"
          :row-class-name="tableRowClassName"
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
        <el-button v-on:click.native="downExcel()" style="margin-top:20px; float: right; margin-right: 10px;" type="primary" icon="el-icon-document">
          Export Excel
        </el-button>
      </el-col>
      <el-col style="margin-top: 50px" :xs="24" :sm="24" :lg="24">
        <div class="block">
          <span class="demonstration">颜色选择器</span>
          <el-color-picker v-model="selectedColor"></el-color-picker>
        </div>
        <div ref="chart" :class="className" :style="{height:height,width:width}" />

      </el-col>
    </el-row>
  </div>
  </div>
</template>

<script>
import * as echarts from 'echarts'
import axios from 'axios'
require('echarts/theme/macarons') // echarts theme
import { debounce } from '@/utils'
import request from "../../../utils/request";
import fileDownload from "js-file-download";

const animationDuration = 6000

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
      default: '500px'
    }
  },
  data() {
    return {
      color: ['#5470c6', '#91cc75', '#fac858', '#ee6666', '#73c0de', '#3ba272', '#fc8452', '#9a60b4', '#ea7ccc'],
      selectedColor: '#5470c6',
      chart: null,
      activeWellDataPropLabelArray: [],
      // ？ data = null 图就消失不见了？ = [] 图就能出来 不知道原因是啥
      data: [],
      params: null,
      //data: origin_data, // 用的是前端造的假数据
      chartData: null,
      options: [{
        value: '选项1',
        label: 'HZ26-6-3'
      }],
      value: '',
      filename : '钻遇地层情况',
    }
  },
  mounted() {
    this.getBarChart()
    this.__resizeHandler = debounce(() => {
      if (this.chart) {
        this.chart.resize()
        console.log('resize')
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
  },
  methods: {
    getBarChart() {
      setTimeout(() => {
        request({
          url: 'statistics/getWellboreFormationBarChart',
          method: 'get',
        }).then(res => {
          let _this = this
          this.data = []
          this.params = []
          this.chartData = null
          this.data = res['tableData']
          this.params = res['params']
          this.chartData = res['chartData']
          this.activeWellDataPropLabelArray = [{
            label: ' ',
            prop: 'group'
          }]
          this.params.forEach(function(value){
            console.log(value)
            _this.activeWellDataPropLabelArray.push({
              label: value,
              prop: value
            })
          })
          this.initChart()
        })
      }, 100)
    },
    handleEdit(index, row, key, value) {
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
    initChart() {
      let _this = this
      let series = []
      console.log(series)
      // chartData['chart'] 中包含柱子的数据和一个折线图的数据
      for(var i = 0; i < this.chartData['chart'].length - 1; i++){
        console.log(i)
        series.push({
            type: 'bar',
            stack: 'stack'
          })
      }
      console.log(series)
      let line = {
        name: '工期',
        type: 'line',
        yAxisIndex: '1',
        data: this.chartData['userNum1']
      }
      series.push(line)
      this.chart = echarts.init(this.$refs.chart, 'macarons')
      let option =  {
        color: this.color,
        legend: {},
        tooltip: {},
        dataset: {
          dimensions: this.chartData['dimensions'],
          source: this.chartData['chart']
        },
        xAxis: {
          type: 'category',
          position: 'top',
          axisLabel: {
            show: true,
            lineStyle: {
              color: '#ccc'
            }
          },
          axisLine: {
            show: true,
            lineStyle: {
              color: '#ccc'
            }
          }
        },
        yAxis: [
          {
            type: 'value',
            name: 'm',
            inverse: true,
            axisLabel: {
              show: true,
              lineStyle: {
                color: '#ccc'
              }
            },
            axisLine: {
              show: true,
              lineStyle: {
                color: '#ccc'
              }
            }
          },
          //  右边y轴去买个煎饼果子呀
          {
            type: 'value',
            name: 'day',
            inverse: true,
            axisLabel: {
              show: true,
              interval: 'auto'
            },
            axisLine: {
              show: true,
              lineStyle: {
                color: '#ccc'
              }
            }
          }],
        // Declare several bar series, each will be mapped
        // to a column of dataset.source by default.
        series: series
      }
      this.chart.setOption(option )
      this.chart.on('dblclick',function(params){
        //点击的柱子的名称
        console.log(params.componentIndex)
        var index = params.componentIndex % _this.color.length
        _this.color[index] = _this.selectedColor
        //柱形图重构
        _this.chart = echarts.init(_this.$refs.chart,'macarons') // 'macarons'
        _this.chart.setOption(option);
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
</style>
