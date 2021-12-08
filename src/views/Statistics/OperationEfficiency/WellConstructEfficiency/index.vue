
<template>
  <div class="dashboard-container">
    <div class="dashboard-editor-container">
    <el-row :gutter="32">
      <el-col style="margin-top:20px">
        <div>
          <span >井名 :</span>
          <el-select style="margin-top: 0px; margin-bottom: 10px" v-model="value" placeholder="全部">
            <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" :disabled="item.disabled " @click.native="getWellLabel(item)">
            </el-option>
          </el-select>
        </div>
      </el-col >
      <el-col :span="12" style = " text-align:center;margin-top: 40px">
        <div ref="well" :class="className" :style="{height:height,width:width}" style="margin-top: 10px" />
        <span >建井周期 : {{totalTime1}}天</span>
      </el-col>
      <el-col :span="12" style = " text-align:center; margin-top: 40px" >
        <div ref="well2" :class="className" :style="{height:height,width:width}" style="margin-top: 10px" />
        <span >钻井周期 : {{totalTime3}}天</span>
      </el-col>
    </el-row>
  </div>
  </div>
</template>

<script>
import * as echarts from 'echarts'
import request from "../../../../utils/request";
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
      default: '600px'
    }
  },
  data() {
    return {
      isReadonly: true,
      chart: null,
      chart2:null,
      // activeWellDataPropLabelArray: [
      //   {
      //     label: '井名',
      //     prop: 'wellName'
      //   },
      //   {
      //     label: '地层厚度',
      //     prop: 'md'
      //   },
      //   {
      //     label: '平均机械钻速',
      //     prop: 'rop'
      //   },{
      //     label: '实际工期',
      //     prop: 'userNum1'
      //   },
      //   {
      //     label: '钻井顺序',
      //     prop: 'drillSequence'
      //   },
      //   {
      //     label: '理论工期',
      //     prop: 'userNum1_p'
      //   }
      // ],
      isReadonlyFlag: {},
      data: [],
      chartData: [],
      totalTime1: null,
      totalTime3: null,
      downloadLoading: false,
      options: [{
        value: '选项1',
        label: 'HZ26-6-1'
      }, {
        value: '选项2',
        label: 'HZ26-6-2'
      },{
        value: '选项3',
        label: 'HZ26-6-3',
      },{
        value: '选项4',
        label: 'HZ26-6-4&Sa'
      }, {
        value: '选项5',
        label: 'HZ26-6-5'
      }, {
        value: '选项6',
        label: 'HZ26-6-6d'
      }, {
        value: '选项7',
        label: 'HZ26-6-7'
      }],
      value: 'HZ26-6-1',
      learnParams: undefined
    }
  },
  watch: {
    data: {
      deep: true,
      handler(newVal) {
        if (newVal) {
          this.getChartData()
          this.initChart(1,this.chartData,this.$refs.well,'建井时效分析图')
          this.initChart(2,this.chartData2,this.$refs.well2,'钻井时效分析图')
        }
      }
    }
  },
  mounted() {
    this.getWellName()
    this.getLayerScipy(this.value)
    this.__resizeHandler = debounce(() => {
      if (this.chart) {
        this.chart.resize()
      }
      if (this.chart2) {
        this.chart2.resize()
      }
    }, 100)
    window.addEventListener('resize', this.__resizeHandler)
  },
  beforeDestroy() {
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
    getLayerScipy(wellName) {
      var _this = this
      setTimeout(() => {
        request({
          url: 'statistics/wellConstructEfficiency',
          method: 'get',
          params: {
            wellName: wellName
          }
        }).then(res => {
          this.data = []
          this.data = res.slice(0,2)
          this.totalTime1 = res[2]
          this.totalTime3 = res[3]
          this.getChartData()
          this.initChart(1,this.chartData,this.$refs.well,'建井时效分析图')
          this.initChart(2,this.chartData2,this.$refs.well2,'钻井时效分析图')
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
      this.chartData = []
      this.chartData2 = []
      this.chartData = this.data[0].map(function(value){
        let o = new Object()
        o['value'] = value[1]
        o['name'] = value[0]
        return o
      })
      this.chartData2 = this.data[1].map(function(value){
        let o = new Object()
        o['value'] = value[1]
        o['name'] = value[0]
        return o
      })
    },
    initChart(flag,data, myChart,text) {
      let chartData = data
      console.log(chartData)

      var option = {
        title: {
          text: text,
          left: 'center'
        },
        tooltip: {
          trigger: 'item'
        },
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
        legend: {
          orient: 'vertical',
          left: 'left',
        },
        series: [
          {
            name: '',
            type: 'pie',
            radius: '50%',
            data: chartData,
            emphasis: {
              itemStyle: {
                shadowBlur: 10,
                shadowOffsetX: 0,
                shadowColor: 'rgba(0, 0, 0, 0.5)'
              }
            }
          }
        ]
      };
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
