<script src="../../../../api/DataMining/LearningFormation.js"></script>
<template>
  <div class="dashboard-container">
    <div class="dashboard-editor-container">
    <el-row :gutter="32">
      <el-col style="margin-top:40px">
        <h4 style="margin-top:10px;text-align: center">钻井工程学习曲线</h4>
        <div>
          <span >地层 :</span>
          <el-select style="margin-top: 10px; margin-bottom: 10px" v-model="value" placeholder="全部">
            <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" :disabled="item.disabled " @click.native="getWellLabel(item)">
            </el-option>
          </el-select>
        </div>
        <div>
          <el-button
            type="primary"
            icon="el-icon-plus"
            size="mini"
            @click="handleAdd(1)"
          >添加</el-button>
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            @click="handleDeleteAll(1)"
          >清空</el-button>
<!--          <el-button-->
<!--            type="success"-->
<!--            icon="el-icon-caret-right"-->
<!--            size="mini"-->
<!--            @click="handleFit(1)"-->
<!--          >重新拟合</el-button>-->
<!--          <el-button-->
<!--            type="primary"-->
<!--            icon="el-icon-plus"-->
<!--            size="mini"-->
<!--            @click="handleAddNewTableAndChart(1)"-->
<!--          >添加新表</el-button>-->
<!--          <el-button-->
<!--            type="danger"-->
<!--            icon="el-icon-delete"-->
<!--            size="mini"-->
<!--            @click="handleDeleteNewTableAndChart"-->
<!--          >删除新表</el-button>-->
        </div>
        <el-table
          :data="data"
          :row-class-name="tableRowClassName"
          :cell-class-name="tableColumnClassName"
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
            <template slot-scope="scope" >
              <!--  在表格中编辑  -->
              <div class="inputDeep">
                <el-input v-model="scope.row[item.prop]" size="small" placeholder="请输入内容" :disabled=false :readonly="isReadonlyFlag['flag'+scope.$index]  === undefined ? true : isReadonly['flag'+scope.$index]" @change="handle(1,scope.$index, scope.row,item.prop,scope.row[item.prop])" />
              </div>
            </template>
          </el-table-column>
          <el-table-column label="操作">
            <template slot-scope="scope">
              <el-button
                size="mini"
                @click="handleEdit(1,scope.$index, scope.row)">编辑</el-button>
              <el-button
                size="mini"
                type="danger"
                @click="handleDelete(1,scope.$index, scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
        <el-button v-on:click.native="downExcel()" style="margin-top:20px; float: right; margin-right: 10px;" type="primary" icon="el-icon-document">
          Export Excel
        </el-button>
      </el-col>
      <el-col>
        <div style="margin-top: 10px">
          <div><el-button size="mini" type="primary">学习曲线模型参数:</el-button></div>
          <div v-if="learnParams !== undefined" style="margin-top: 5px">
            <el-button size="mini" type="success">a = {{learnParams.a}}</el-button>
            <el-button size="mini" type="success">b = {{ learnParams.b }}</el-button>
            <el-button size="mini" type="success">r = {{learnParams.r}}</el-button>
            <el-button size="mini" type="success">学习率 = {{learnParams.learningRate}}</el-button>
          </div>
        </div>
        <div ref="well" :class="className" :style="{height:height,width:width}" style="margin-top: 10px" />
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
      default: '600px'
    }
  },
  data() {
    return {
      filename: '钻井工程学习曲线',
      isReadonly: true,
      chart: null,
      chart2:null,
      newTableAndChart: false,
      activeWellDataPropLabelArray: [
        {
          label: '井名',
          prop: 'wellName'
        },
        {
          label: '地层厚度(m)',
          prop: 'md'
        },
        {
          label: '平均机械钻速(m/h)',
          prop: 'rop'
        },{
          label: '实际工期(day)',
          prop: 'userNum1'
        },
        {
          label: '钻井顺序',
          prop: 'drillSequence'
        },
        {
          label: '理论工期(day)',
          prop: 'userNum1_p'
        }
      ],
      isReadonlyFlag: {},
      data: [],
      chartData: [],
      downloadLoading: false,
      options: [{
        value: '选项8',
        label: '韩江组'
      }, {
        value: '选项2',
        label: '粤海组'
      },{
        value: '选项3',
        label: '文昌组',
      },{
        value: '选项4',
        label: '珠海组'
      }, {
        value: '选项5',
        label: '万山组'
      }, {
        value: '选项6',
        label: '恩平组'
      }, {
        value: '选项7',
        label: '珠江组'
      },{
        value: '选项1',
        label: '珠江及以上层段'
      }],
      //value: '韩江组',
      value: '珠江及以上层段',
      learnParams: undefined
    }
  },
  watch: {
    data: {
      deep: true,
      handler(newVal) {
        if (newVal) {
          this.chartData = []
          this.chartData = this.getChartData(1)
          this.initChart(1,this.chartData,this.$refs.well)
        }
      }
    }
  },
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
    getLayerScipy(formName) {
      var _this = this
      setTimeout(() => {
        request({
            url: 'dataMining/wellInfoByWellName/all',
            method: 'get',
            params: {
              formName: formName
            }
          }).then(res => {
          this.data = []
          // res 返回的是数组json 数组第一个元素为 基本数据字典的数组 第二个元素是学习率参数字典｛a:a,b:b,r:r,learningRate: 1-r｝
          var data = res['data']
          this.learnParams = undefined
          var wellData = data[0]
          wellData = JSON.parse(wellData)
          this.learnParams = data[1]
          wellData.forEach(function(value){
              // if(value['formName'] === formName){
              //   _this.data.push(value)
              // }
            _this.data.push(value)
          })
          this.chartData = []
          this.chartData = this.getChartData(1)
          this.initChart(1,this.chartData,this.$refs.well)
        })
      }, 100)
    },
    // getRefit(flag) {
    //   setTimeout(() => {
    //     let data
    //     if(flag === 2){
    //       data = this.data2
    //     }else {
    //       data = this.data
    //     }
    //     crudRefit()({
    //       url: 'api/getData/Refit',
    //       method: 'post',
    //       data: {
    //         fitData: data
    //       }
    //     }).then(res => {
    //       if(flag === 2){
    //         this.data2 = []
    //         this.data2 = JSON.parse(res)
    //         this.chartData2 = []
    //         this.chartData2 = this.getChartData(2)
    //         this.initChart(2,this.chartData2,this.$refs.well2)
    //       }else {
    //         this.data = []
    //         this.data = JSON.parse(res)
    //         this.chartData = []
    //         this.chartData = this.getChartData(1)
    //         this.initChart(1,this.chartData,this.$refs.well)
    //       }
    //     })
    //   }, 100)
    // },
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
      // columnIndex 为列名的index
      if(columnIndex === 5){
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
      if(flag === 2){
        this.data2.splice(index, 1)
      }else {
        this.data.splice(index, 1)
        delete this.isReadonlyFlag['flag' + index]
      }
    },
    handleDeleteAll(flag){
      if(flag === 2){
        this.data2 = []
      }else {
        this.data = []
        this.isReadonlyFlag = {}
      }
    },

    handleAdd(flag) {
      let obj = {
        'wellName': undefined,
        'md': undefined,
        'drillSequence': undefined,
        'userNum1': undefined,
        'userNum1_p': undefined
      }
      if(flag === 2){
        this.data2.push(obj)
      }else {
        this.data.push(obj)
      }
    },
    handleDeleteNewTableAndChart() {
      this.newTableAndChart = false;
      this.chart2 = null
      this.data2 = [];
      this.chartData2 = [];
    },
    handleAddNewTableAndChart() {
      this.newTableAndChart = true;
    },
    getChartData(flag) {
      var userNum1 = []
      var md = []
      var drillSequence = []
      var userNum1_p = []
      var wellName = []
      //  flag = 1 : 处理第一个图表的数据
      //  flag = 2 : 处理第二个图表的数据
      if(flag === 2){
        this.data2.forEach(function(value) {
          wellName.push(value.wellName)
          userNum1.push(value.userNum1)
          md.push(value.md)
          drillSequence.push(value.drillSequence)
          userNum1_p.push(value.userNum1_p)
        })
        return {
          'wellName': wellName,
          'userNum1': userNum1,
          'md': md,
          'drillSequence': drillSequence,
          'userNum1_p': userNum1_p
        }
      }
      this.data.forEach(function(value) {
        wellName.push(value.wellName)
        userNum1.push(value.userNum1)
        md.push(value.md)
        drillSequence.push(value.drillSequence)
        userNum1_p.push(value.userNum1_p)
      })
      return {
        'wellName': wellName,
        'userNum1': userNum1,
        'md': md,
        'drillSequence': drillSequence,
        'userNum1_p': userNum1_p
      }
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
    initChart(flag,data, myChart) {
      let chartData = data
      var option = {
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross',
            crossStyle: {
              color: '#999'
            }
          }
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
          data: ['井深', '实际钻井工期', '理论钻井工期']
        },
        xAxis: {
          name: '井',
          nameLocation: 'middle',
          nameTextStyle: {
            padding: [12, 12, 12, 12]
          },
          axisLabel: {
            fontSize: 15,
            color: '#000'
          },
          type: 'category',
          data: chartData.wellName,
          axisPointer: {
            type: 'shadow'
          },
          axisLine: {
            lineStyle: {
              color: '#ccc'
            }
          }
        },
        yAxis: [
          {
            type: 'value',
            name: '井深（m)',
            nameTextStyle: {
              fontSize: 15,
              color: "#000"
            },
            min: 0,
            axisLabel: {
              formatter: '{value} m',
              fontSize: 15,
              color: '#000'
            },
            axisLine: {
              lineStyle: {
                color: '#000'
              }
            }
          },
          {
            type: 'value',
            name: '工期（小时）',
            nameTextStyle: {
              fontSize: 15,
              color: "#000"
            },
            min: 0,
            axisLabel: {
              formatter: '{value} 小时',
              fontSize: 15,
              color: '#000'
            },
            axisLine: {
              lineStyle: {
                color: '#000'
              }
            }
          }
        ],
        series: [
          {
            name: '井深',
            type: 'bar',
            data: chartData.md,
            barWidth: '40%',
          },
          {
            name: '实际钻井工期',
            type: 'line',
            yAxisIndex: 1,
            data: chartData.userNum1,
            itemStyle: {
              normal: {
                lineStyle: {
                  width: 4//设置线条粗细
                }
              }
            }
          },
          {
            name: '理论钻井工期',
            type: 'line',
            yAxisIndex: 1,
            data: chartData.userNum1_p,
            itemStyle: {
              normal: {
                lineStyle: {
                  width: 4//设置线条粗细
                }
              }
            }
          }
        ]
      }
      if(flag === 2){
        this.chart2 = echarts.init(myChart,'macarons') // 'macarons'
        this.chart2.setOption(option)
      }else {
        this.chart = echarts.init(myChart,'macarons') // 'macarons'
        this.chart.setOption(option)
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
