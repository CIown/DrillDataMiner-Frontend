<script src="../../../../api/DataMining/LearningFormation.js"></script>
<template>
  <div class="dashboard-container">
    <div class="dashboard-editor-container">

      <el-row :gutter="32">
        <el-col :span="12" >
          <el-form ref="form1" :model="form" label-width="80px">
            <el-form-item label="选择模型">
              <el-select v-model="form.modelName" filterable placeholder="请选择">
                <template slot="empty">
                  <div >
                    <p style="padding-left: 10px">正在加载中 <i class="el-icon-loading" ></i></p>
                  </div>
                </template>
                <el-option
                  v-for="item in modelNameOptions"
                  :key="item.value"
                  :label="item.label"
                  :value="item.label">
                </el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="选择井">
              <el-select v-model="form.wellName" filterable placeholder="请选择">
                <template slot="empty">
                  <div>
                    <p style="padding-left: 10px">正在加载中 <i class="el-icon-loading" ></i></p>
                  </div>
                </template>
                <el-option
                  v-for="item in wellNameOptions"
                  :key="item.value"
                  :label="item.label"
                  :value="item.label">
                </el-option>
              </el-select>
            </el-form-item>

            <el-form-item label="开始深度" >
              <el-input-number controls-position="right" v-model="form.startDepth" :disabled="depthDisabled" clearable></el-input-number>
            </el-form-item>
            <el-form-item label="结束深度" >
              <el-input-number controls-position="right" v-model="form.endDepth" :disabled="depthDisabled" clearable></el-input-number>
            </el-form-item>
            <el-form-item label="全井段">
              <el-switch
                v-model="switchValue"
                active-color="#13ce66"
                inactive-color="#ff4949"
                @change="changeSwitch($event)"
              >
              </el-switch>
            </el-form-item>
          </el-form>
        </el-col>
        <el-col :span="12">
          <el-form ref="form2" :model="form" label-width="80px">
              <el-form-item label="工程参数: ">
                <el-radio v-model="modifyType" label="WOBA">WOB</el-radio>
                <el-radio v-model="modifyType" label="RPMA">RPM</el-radio>
              </el-form-item>
              <el-form-item label="优化方式: ">
                  <div>
                    <el-radio-group v-model="form.calWay" @change="selectCalWay">
                      <el-radio label="add"></el-radio>
                      <el-radio label="mul"></el-radio>
                      <el-radio label="fixed"></el-radio>
                    </el-radio-group>
                    <el-input-number controls-position="right" v-model="modifyNum " clearable style="margin-top: 20px"></el-input-number>
                    <el-button
                      type="primary"
                      @click="onConfirmModify">
                      确定修改
                    </el-button>
                  </div>
              </el-form-item>
            </el-form>
        </el-col>
      </el-row>

      <el-row :gutter="32">
          <el-col :span="12" style="text-align: center;">
  <!--            <el-button-->
  <!--              type="primary"-->
  <!--              :disabled="(form.modelName !== null && form.wellName !== null) ? false : true"-->
  <!--              @click="onSubmit">-->
  <!--                查看结果-->
  <!--            </el-button>-->
              <el-button
                :disabled="(form.modelName !== null && form.wellName !== null) ? false : true"
                @click="onWatch">
                {{!tableIsVisibled?'查看数据':'收起'}}
              </el-button>
          </el-col>
          <el-col :span="12" style="text-align: center;">
            <el-button
              type="primary"
              :disabled="(form.modelName !== null && form.wellName !== null) ? false : true"
              @click="onSubmit"
              :loading="loadingBtn">
              {{loadingBtn?'正在优化中':'开始优化'}}
            </el-button>
            <p
              v-show="textIsVisibled"
              style="margin-left: 20px">实际ROP: {{practicalRopMean}} 优化后预测ROP:{{predictRopMean}}</p>

  <!--            <el-button-->
  <!--              :disabled="(form.modelName !== null && form.wellName !== null) ? false : true"-->
  <!--              @click="onWatch">-->
  <!--              {{!tableIsVisibled?'查看数据':'收起'}}-->
  <!--            </el-button>-->
          </el-col>
        </el-row>

      <el-row>
        <el-col v-show="tableIsVisibled">
          <el-table
            :data="tableData"
            :row-class-name="tableRowClassName"
            :cell-class-name="tableColumnClassName"
            style="width: 100%; margin-top: 20px;"
            border
            height="600"
            ref="myTable"
          >
            <template slot="empty">
            </template>
            <el-table-column
              v-for="(item) in activeParamsDataPropLabelArray"
              :key="item.prop"
              align="left"
              :prop="item.prop"
              :label="item.label"
            >
              <template slot-scope="scope">
                {{ scope.row[item.prop]}}
              </template>
            </el-table-column>
            <!-- ele 暂无数据插槽 -->
            <template slot="empty">
              <div class="noData">
                <p style="padding-left: 10px"> 正在加载中 <i class="el-icon-loading"  > </i> </p>
              </div>
            </template>
          </el-table>
          <div class="pagination"  style="text-align: center;margin-top: 30px;">
            <el-pagination
              background
              layout="prev, pager, next"
              @current-change="handleCurrentChange"
              :page-count="pageCount">
            </el-pagination>
          </div>
        </el-col>
        <el-col style="margin-top: 10px">
          <div ref="chart" :class="className" :style="{height:height,width:width}" />
        </el-col>
        <el-col style="margin-top: 10px">
          <div ref="wobParamsChart" :class="className" :style="{height:height,width:width}" />
        </el-col>
        <el-col style="margin-top: 10px">
          <div ref="rpmParamsChart" :class="className" :style="{height:height,width:width}" />
        </el-col>

      </el-row>
      </div>
    </div>

</template>

<script>
import * as echarts from 'echarts'
import request from "../../../../utils/request"
require('echarts/theme/macarons') // echarts theme
import { debounce } from '@/utils'
import { getToken } from '@/utils/auth'
import {Notification} from "element-ui"; // getToken from cookie
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
      modifyType:'WOBA',
      modifyNum: 0,
      depthDisabled: false,
      total: 355,
      pageCount: 100,
      currentPage: 1, //默认显示页面为1
      pagesize: 19, //    每页的数据条数
      modelNameOptions: null,
      wellNameOptions: null,
      tableIsVisibled: false,
      tableQequestState: 1,
      isDisabled: false,
      form: {
        calWay: 'add',
        wellName: 'HZ26-6-7',
        modelName: 'HZ26-6-7',
        startDepth: 1,
        endDepth: 1000,
        modifyWOBANum: 0,
        modifyRPMANum: 0,
      },
      defaultNum: 1,
      isReadonlyFlag: {},
      data: [],
      downloadLoading: false,
      learnParams: undefined,
      tableData: null,
      originalTableData: null,
      changeTableData: this.originalTableData,
      activeParamsDataPropLabelArray: [
      ],
      chart:null,
      rpmParamsChart: null,
      wobParamsChart: null,
      paramsChartData: null,
      changeParamsChartData: null,
      paramsChartMap: {
        'WOBA': 1,
        'RPMA': 2
      },
      chartData: {
        'practicalRop': null,
        'optimizedROP': null,
        'depth': null,
        'originalRPM': null,
        'optimizedRPM': null,
        'originalWOB':null,
        'optimizedROB':null,
      },
      practicalRopMean: null,
      predictRopMean: null,
      loadingBtn: false,
      textIsVisibled: false
    }
  },
  watch: {
    'form.modelName': {
      deep: true,
      handler(newVal) {
        if (newVal) {
          if(this.form.modelName !== null && this.form.wellName !== null){
            this.tableData = null
            this.getTable()
          }
        }
      }
    },
    'form.wellName': {
      deep: true,
      handler(newVal) {
        if (newVal) {
          if(this.form.modelName !== null && this.form.wellName !== null){
            this.tableData = null
            this.getTable()
          }
        }
      }
    },
    // 'form.modifyWOBANum': {
    //   deep: true,
    //   handler(newVal) {
    //     if (newVal) {
    //       this.changeParams()
    //       this.initRpmParamsChart()
    //       this.initWobParamsChart()
    //     }
    //   }
    // },
    // 'form.modifyRPMANum': {
    //   deep: true,
    //   handler(newVal) {
    //     if (newVal) {
    //       this.changeParams()
    //       this.initRpmParamsChart()
    //       this.initWobParamsChart()
    //     }
    //   }
    // },
    // 'form.calWay': {
    //   deep: true,
    //   handler(newVal) {
    //     if (newVal) {
    //       this.changeParams()
    //       this.initWobParamsChart()
    //       this.initRpmParamsChart()
    //     }
    //   }
    // },
    // 'form.modifyParam': {
    //   deep: true,
    //   handler(newVal) {
    //     if (newVal) {
    //       this.changeParams()
    //       this.initWobParamsChart()
    //       this.initRpmParamsChart()
    //     }
    //   }
    // },
    // data: {
    //   deep: true,
    //   handler(newVal) {
    //     if (newVal) {
    //     }
    //   }
    // }
  },
  mounted() {
    this.initParams()
    this.getTable()
    this.__resizeHandler = debounce(() => {
      if (this.chart) {
        this.chart.resize()
      }
      if (this.wobParamsChart) {
        this.wobParamsChart.resize()
      }
      if (this.rpmParamsChart) {
        this.rpmParamsChart.resize()
      }
    }, 100)
    window.addEventListener('resize', this.__resizeHandler)
  },
  beforeDestroy() {
    this.chart.dispose()
    this.chart = null
    this.wobParamsChart.dispose()
    this.wobParamsChart = null
    this.rpmParamsChart.dispose()
    this.rpmParamsChart = null
  },
  methods: {
    changeSwitch(status){
      console.log(status)
      if(status){
        this.form.startDepth = 1;
        this.form.endDepth = 10000;
        this.depthDisabled = true;
        return;
      }
      this.depthDisabled = false;
      return
    },
    handleCurrentChange(currentPage) {
      this.currentPage = currentPage;
      this.getPageData(this.changeTableData)
    },
    getPageData(data){
      let start = (this.currentPage - 1) * this.pagesize
      let end = this.currentPage * this.pagesize
      this.tableData = data.slice(start,end)
    },
    initParams(){
      const _this = this
      setTimeout(() => {
        request({
          url: 'dataMining/BPPrediction',
          method: 'get',
          params: {
            wellName: JSON.stringify("wellName"),
            modelName: JSON.stringify("modelName")
          }
        }).then(res => {
          this.wellNameOptions = []
          this.modelNameOptions = []
          let wellNames = []
          let modelNames = []
          wellNames = res.wellName
          modelNames = res.modelName
          let index = 1
          wellNames.forEach((name) => {
            _this.wellNameOptions.push({
              value: index,
              label: name,
            });
            index++
          })
          index = 1
          modelNames.forEach((name) => {
            _this.modelNameOptions.push({
              value: index,
              label: name,
            });
            index++
          })
        })
      }, 100)
    },
    getTable(){
      const _this = this
      setTimeout(() => {
        request({
          url: 'dataMining/BPTable',
          method: 'get',
          params: {
            wellName: JSON.stringify(this.form.wellName),
            modelName: JSON.stringify(this.form.modelName),
            startDepth: JSON.stringify(this.form.startDepth),
            endDepth: JSON.stringify(this.form.endDepth)
          }
        }).then(res => {
          console.log(res)
          // this.pageCount = res.pageCount
          this.currentPage = 1
          this.activeParamsDataPropLabelArray = []
          this.originalTableData = []
          this.originalTableData = res['tableData']
          this.getPageData(this.originalTableData)
          this.paramsChartData = res['chartData']['data']
          Object.keys(this.tableData[0]).forEach(function(key){
            _this.activeParamsDataPropLabelArray.push({
              label: key,
              prop: key
            })
          })
          this.changeParams()
          this.isDisabled = false
          this.initWobParamsChart()
          this.initRpmParamsChart()
        })
      }, 100)
    },
    selectCalWay(label){
      if(label === 'mul'){
        this.modifyNum = 1
        this.form.modifyWOBANum = 1
        this.form.modifyRPMANum = 1
      }
      if(label === 'add'){
        this.modifyNum = 0
        this.form.modifyWOBANum = 0
        this.form.modifyRPMANum = 0
      }
      if(label === 'fixed'){
        this.modifyNum = 100
        this.form.modifyWOBANum = 100
        this.form.modifyRPMANum = 100
      }
    },
    changeParams(){
      let _this = this
      // let paramsName = this.modifyType
      let calWay = this.form.calWay
      // let num = this.modifyNum
      // if (paramsName === 'WOBA'){
      //   num = parseFloat(this.form.modifyNum)
      // }else {
      //   num = parseFloat(this.form.modifyNum)
      // }

      let originalTableData = JSON.parse(JSON.stringify(this.originalTableData)) //深拷贝 不改变原数组
      this.changeTableData = JSON.parse(JSON.stringify(this.originalTableData))
      this.changeParamsChartData = JSON.parse(JSON.stringify(this.paramsChartData))
      console.log(this.changeTableData)
      if(calWay === 'mul'){
        this.changeTableData = originalTableData.map(function(value){
          value['WOBA'] = value['WOBA'] * _this.form.modifyWOBANum
          value['RPMA'] = value['RPMA'] * _this.form.modifyRPMANum
          return value
        })
        this.changeParamsChartData = this.changeParamsChartData.map(function(value){
          value[_this.paramsChartMap['WOBA']] = value[_this.paramsChartMap['WOBA']] * _this.form.modifyWOBANum
          value[_this.paramsChartMap['RPMA']] = value[_this.paramsChartMap['RPMA']] * _this.form.modifyRPMANum
          return value
        })
        this.getPageData(this.changeTableData)
      }
      if(calWay === 'add'){
        this.changeTableData = originalTableData.map(function(value){
          value['WOBA'] = value['WOBA'] + _this.form.modifyWOBANum
          value['RPMA'] = value['RPMA'] + _this.form.modifyRPMANum
          return value
        })
        console.log(this.changeParamsChartData)
        this.changeParamsChartData = this.changeParamsChartData.map(function(value){
          value[_this.paramsChartMap['WOBA']] = value[_this.paramsChartMap['WOBA']] + _this.form.modifyWOBANum
          value[_this.paramsChartMap['RPMA']] = value[_this.paramsChartMap['RPMA']] + _this.form.modifyRPMANum
          return value
        })
        this.getPageData(this.changeTableData)
      }
      if(calWay === 'fixed'){
        this.changeTableData = originalTableData.map(function(value){
          value['WOBA'] = _this.form.modifyWOBANum
          value['RPMA'] = _this.form.modifyRPMANum
          return value
        })
        this.changeParamsChartData = this.changeParamsChartData.map(function(value){
          value[_this.paramsChartMap['WOBA']] = _this.form.modifyWOBANum
          value[_this.paramsChartMap['RPMA']] = _this.form.modifyRPMANum
          return value
        })
        this.getPageData(this.changeTableData)
      }
    },
    onConfirmModify(){
      let paramsName = this.modifyType
      if (paramsName === 'WOBA'){
        this.form.modifyWOBANum = this.modifyNum
      }else {
        this.form.modifyRPMANum = this.modifyNum
      }
      this.changeParams()
      this.initWobParamsChart()
      this.initRpmParamsChart()
      console.log(paramsName,this.modifyNum,this.form.modifyWOBANum, this.form.modifyRPMANum)

    },
    onSubmit() {
      this.isDisabled = false
      this.loadingBtn = true
      this.textIsVisibled = false
      setTimeout(() => {
        request({
          url: 'dataMining/BPPrediction',
          method: 'post',
          data: {
            paramsData: JSON.stringify(this.form)
          }
        }).then(res => {
          console.log(res)
          this.practicalRopMean = res.practicalRopMean
          this.predictRopMean = res.predictRopMean
          this.getChartData(res)
          this.initChart()
          this.isDisabled = true
          this.loadingBtn = false
          this.textIsVisibled = true
          Notification.success({
            title: '优化已完成，请在查看图表',
            duration: 5000
          })
        })
      }, 100)
    },
    onWatch() {
      if(this.tableIsVisibled === true) {
        this.tableIsVisibled = false
        return
      }
      this.tableIsVisibled = true
      return
    },
    getWellLabel(item){
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
    getChartData(data) {
      this.chartData = {}
      this.chartData.depth = data.depth
      this.chartData.optimizedROP = data.predictRop
      this.chartData.practicalRop = data.practicalRop
    },
    initWobParamsChart(){
      var option = {
        tooltip: {
          trigger: 'item'
        },
        legend: {
        },
        grid: {
          // bottom:'25%'
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
        dataZoom: [{
          start: 0,
          end: 1,
        }, {
          start: 0,
          end: 1,
          type: 'inside'
        }],
        xAxis: {
          type: 'category',
          name: '深度',
        },
        yAxis: {
          name: 'WOB',
          scale: true
        },
        dataset:[{
            source: this.paramsChartData
          },
          {
            source: this.changeParamsChartData
          }],
        series: [
          {
            name: 'WOB',
            type: 'line',
            datasetIndex: 0,
            encode: {
              x: [0],
              y: [1]
            }
          },
          {
            name: '优化WOB',
            type: 'line',
            datasetIndex: 1,
            encode: {
              x: [0],
              y: [1]
            }
          }
        ]
      }
      this.wobParamsChart = echarts.init(this.$refs.wobParamsChart,'macarons') // 'macarons'
      this.wobParamsChart.setOption(option)
    },
    initRpmParamsChart(){
      var option = {
        tooltip: {
          trigger: 'item'
        },
        legend: {
        },
        grid: {
          // bottom:'25%'
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
        dataZoom: [{
          start: 0,
          end: 1
        }, {
          start: 0,
          end: 1,
          type: 'inside'
        }],
        xAxis: {
          type: 'category',
          name: '深度',
        },
        yAxis: {
          name: 'RPM',
          scale: true
        },
        dataset:[{
          source: this.paramsChartData
        },
          {
            source: this.changeParamsChartData
          }],
        series: [
          {
            name: 'RPM',
            type: 'line',
            datasetIndex: 0,
            encode: {
              x: [0],
              y: [2]
            }
          }, {
            name: '优化RPM',
            type: 'line',
            datasetIndex: 1,
            encode: {
              x: [0],
              y: [2]
            }
          },
        ]
      }
      this.rpmParamsChart = echarts.init(this.$refs.rpmParamsChart,'macarons') // 'macarons'
      this.rpmParamsChart.setOption(option)
    },
    initChart() {
      var option = {
        tooltip: {
          trigger: 'item'
        },
        legend: {
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
        xAxis: {
          type: 'category',
          name: '深度',
          data: this.chartData['depth']
        },
        yAxis: {
          type: 'value',
          name: 'ROP',
          scale: true
        },
        series: [
          {
            name: '实际ROP',
            data: this.chartData.practicalRop,
            type: 'line'
          },
          {
            name: '优化预测ROP',
            data: this.chartData.optimizedROP,
            type: 'line'
          }]
      }
      this.chart = echarts.init(this.$refs.chart,'macarons') // 'macarons'
      this.chart.setOption(option)
    },

  }
}
</script>
<style scoped>
.dashboard-editor-container {
  padding: 32px;
  /*background-color: rgb(240, 242, 245);*/
  position: relative;
}
.noData .el-icon-loading {
  width: 10px;
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
