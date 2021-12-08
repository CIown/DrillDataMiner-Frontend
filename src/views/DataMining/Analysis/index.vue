<template>
  <div class="dashboard-container">
    <div class="dashboard-editor-container">
    <el-row :gutter="32">
      <el-col>
        <h4 style="margin-top:10px;text-align: center">各井段机械钻速</h4>
<!--        <div>-->
<!--          <span >区块 :</span>-->
<!--          <el-select style="margin-top: 10px; margin-bottom: 10px" v-model="value" placeholder="全部">-->
<!--            <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" :disabled="item.disabled">-->
<!--            </el-option>-->
<!--          </el-select>-->
<!--        </div>-->
        <span>单位：m/h</span>
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
            :prop="item.prop"
            :label="item.label"
            :show-overflow-tooltip="true"
            align="left"
            :class-name="getColumnName()"
          >
            <template slot-scope="scope">
              {{ scope.row[item.prop] }}
            </template>
          </el-table-column>
        </el-table>
        <el-button v-on:click.native="downExcel()" style="margin-top:20px; float: right; margin-right: 10px;" type="primary" icon="el-icon-document">
          Export Excel
        </el-button>
      </el-col>
      <el-col v-for="(item,index) in chartList" class="chart-item-area" style="margin-top:20px" :xs="24" :sm="24" :lg="12">
        <div class="echarts" :id="item.id" :style="{height:height,width:width}"></div>
      </el-col>
    </el-row>
    </div>
  </div>
</template>

<script>
import * as echarts from 'echarts'
require('echarts/theme/macarons') // echarts theme
import request from "../../../utils/request";
import fileDownload from "js-file-download";

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
      chart1: null,
      chart2: null,
      chart3: null,
      activeWellDataPropLabelArray: [],
      data: [],
      chartList: [],
      chartData: null,
      char1Data: [],
      char2Data: [],
      char3Data: [],
      downloadLoading: false,
      options: [{
        value: '选项1',
        label: 'HZ26-6'
      }],
      value: '',
      filename: '各井段机械钻速'
    }
  },
  mounted() {
    this.initParams()

    // this.__resizeHandler = debounce(() => {
    //   if (this.chart1) {
    //     this.chart1.resize()
    //     this.chart2.resize()
    //     this.chart3.resize()
    //   }
    // }, 100)
    window.addEventListener('resize', this.__resizeHandler)
  },
  beforeDestroy() {
    if (this.chartList.length === 0) {
      return
    }
    for(var i = 0; i < this.chartList.length; i++){
      this.chartList[i]['chart'].dispose()
      this.chartList[i]['chart'] = null
    }
  },
  methods: {
    initParams(){
      setTimeout(() => {
        const _this = this
        request({
          url: 'dataMining/analysis',
          method: 'get',
        }).then(res => {
          let _this = this
          this.data = []
          this.chartData = {}
          this.data = res['tableData']
          this.chartData = res['chartData']
          let params = res['tableParams']
          this.activeWellDataPropLabelArray = [{
            label: '井眼尺寸(in)',
            prop: 'wellSize'
          }]
          params.forEach(function(value){
            _this.activeWellDataPropLabelArray.push({
              label: value,
              prop: value
            })
          })

          this.activeWellDataPropLabelArray.push({
            label: '平均',
            prop: 'mean'
          })
          this.createdEcharts()
        })
      }, 100)
    },
    getColumnName() {
      return '123'
    },
    tableRowClassName({ row, rowIndex }) {
      if (rowIndex % 2 !== 1) {
        return 'success-row'
      }
      return ''
    },
    handleEdit(index, row, key, value) {
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
          var filename = this.filename
          fileDownload(res, filename + '.xlsx')
        })
      }, 100)
    },
    createdEcharts(){
      let _this = this
      let arr = [];
      let wellSize = this.chartData['wellSize']
      let length = Object.keys(wellSize).length
      for(let i = 0; i < length; i++) {
        let item = {
          chart: '',       // chart 对象实例
          id: 'id' + i,       // 为了标示 id 不同
        }
        arr.push(item);
      }
      this.chartList = arr;

      this.$nextTick(() => {
        Object.keys(wellSize).forEach(function(key,index){
          console.log(key,index)
          _this.chartList[index].chart = echarts.init(document.getElementById(_this.chartList[index].id),'macarons');
          _this.initChart(_this.chartList[index].chart,wellSize[key],key);
        })
      })
    },
    initChart(chart,data,key) {
      chart.setOption({
        title: {
          text:'井眼尺寸: ' + key,
          left:'center', //水平安放位置，默认为'left'，可选为：'center' | 'left' | 'right' | {number}（x坐标，单位px）
          top : 'top',
        },
        legend: {
          top:"6%",
        },
        tooltip: {},
        grid: {
          bottom:'25%'
        },
        toolbox: {
          show: true,
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
        dataset: {
          dimensions: ['wellName', 'rop',],
          source: data['barData']
        },
        xAxis: {
          type: 'category',
          axisLabel: {
            // formatter:function(value){
            //   return value.split("").join("\n");
            // },
            interval: 0,
            rotate: 40
          },
        },
        yAxis: {},
        // Declare several bar series, each will be mapped
        // to a column of dataset.source by default.
        series: [
          {type: 'bar',name: '钻速'},
          {type: 'line',name: '平均钻速',data: data['mean'],}
        ]
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
