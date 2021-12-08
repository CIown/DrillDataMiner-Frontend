<script src="../../../../api/DataMining/Scipy.js"></script>
<template>
  <div class="dashboard-container">
    <div class="dashboard-editor-container">
    <el-row :gutter="32">
      <el-col style="margin-top:40px">
        <h4 style="margin-top:10px;text-align: center">钻井工程学习曲线</h4>
<!--        <div>-->
<!--          <span >区块 :</span>-->
<!--          <el-select style="margin-top: 10px; margin-bottom: 10px" v-model="value" placeholder="全部">-->
<!--            <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" :disabled="item.disabled " @click.native="getWellLabel(item)">-->
<!--            </el-option>-->
<!--          </el-select>-->
<!--        </div>-->
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
          <el-button
            type="success"
            icon="el-icon-caret-right"
            size="mini"
            @click="handleFit(1)"
          >重新拟合</el-button>
          <el-button
            type="primary"
            icon="el-icon-plus"
            size="mini"
            @click="handleAddNewTableAndChart(1)"
          >添加新表</el-button>
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            @click="handleDeleteNewTableAndChart"
          >删除新表</el-button>
          <el-button
            type="success"
            icon="el-icon-caret-right"
            size="mini"
            @click="handlePredicted"
          >预测</el-button>
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

      <!--                      第二个表格和图表                      -->
      <el-col v-if="newTableAndChart" style="margin-top:40px">
        <div>
          <el-button type="primary" icon="el-icon-plus" size="mini" @click="handleAdd(2)">添加</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini" @click="handleDeleteAll(2)">清空</el-button>
          <el-button type="success" icon="el-icon-caret-right" size="mini" @click="handleFit(2)">重新拟合</el-button>
        </div>
        <el-table
          :data="data2"
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
                <el-input v-model="scope.row[item.prop]" size="small" placeholder="请输入内容" @change="handle(2,scope.$index, scope.row,item.prop,scope.row[item.prop])" />
              </div>
            </template>
          </el-table-column>
          <el-table-column label="操作">
            <template slot-scope="scope">
              <el-button
                size="mini"
                @click="handleEdit(2,scope.$index, scope.row)">编辑</el-button>
              <el-button
                size="mini"
                type="danger"
                @click="handleDelete(2,scope.$index, scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
        <el-button :loading="downloadLoading" style="margin-top:20px; float: right; margin-right: 10px;" type="primary" icon="el-icon-document">
          Export Excel
        </el-button>
      </el-col>
      <el-col v-if="newTableAndChart">
        <div ref="well2" :class="className" :style="{height:height,width:width}" style="margin-top: 10px" />
      </el-col>
    </el-row>
    <el-drawer
      title="编辑/添加"
      :before-close="handleClose"
      :visible.sync="dialog"
      direction="rtl"
      custom-class="demo-drawer"
      ref="drawer"
    >
      <div class="demo-drawer__content">
        <el-form class="form" :model="form">
          <el-form-item label="钻井顺序" :label-width="formLabelWidth">
            <el-input v-model="form.drillSequence" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="井深" :label-width="formLabelWidth">
            <el-input v-model="form.md" autocomplete="off"></el-input>
          </el-form-item>
        </el-form>
        <el-form class="form" :model="form2" >
          <el-form-item label="预测工期" :label-width="formLabelWidth">
            <el-input disabled=true v-model="form2.userNum1_p" autocomplete="off"></el-input>
          </el-form-item>
        </el-form>
        <div class="demo-drawer__footer" >
          <el-button @click="cancelForm">取 消</el-button>
          <el-button type="primary" @click="predicted" :loading="loading">{{ loading ? '预测中 ...' : '预测' }}</el-button>
        </div>
      </div>
    </el-drawer>
  </div>
  </div>
</template>

<script>
import * as echarts from 'echarts'
require('echarts/theme/macarons') // echarts theme
import { debounce } from '@/utils'
import { getToken } from '@/utils/auth'
import request from "../../../../utils/request";
import fileDownload from "js-file-download"; // getToken from cookie

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
      dialog: false,
      loading: false,
      isReadonly: true,
      chart: null,
      chart2:null,
      newTableAndChart: false,
      activeWellDataPropLabelArray: [
        {
          label: '井名',
          prop: 'wellName'
        }, {
          label: '井深(m)',
          prop: 'md'
        }, {
          label: '开钻时间',
          prop: 'dtTmSpud'
        },
        {
          label: '实际工期(day)',
          prop: 'userNum1'
        },
        {
          label: '钻井顺序',
          prop: 'drillSequence'
        },
        {
          label: '理论工期',
          prop: 'userNum1_p'
        }],
      isReadonlyFlag: {},
      data: [],
      data2: [],
      chartData: [],
      chartData2: [],
      downloadLoading: false,
      filename: '钻井工程学习曲线',
      learnParams: undefined,
      handleMethod: null,
      form: {
        'md': null,
        'drillSequence':null,
      },
      form2: {
        'userNum1_p': null
      }
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
    },
    data2: {
      deep: true,
      handler(newVal) {
        if (newVal) {
          this.chartData2 = []
          this.chartData2 = this.getChartData(2)
          this.initChart(2,this.chartData2,this.$refs.well2)
        }
      }
    }
  },
  mounted() {
    this.getScipy()
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
    if (!this.chart) {
      return
    }
    this.chart.dispose()
    this.chart = null
    if (!this.chart2) {
      return
    }
    this.chart2.dispose()
    this.chart2 = null
  },
  methods: {
    predicted(){
      setTimeout(() => {
        request({
          url: 'dataMining/scipy/predicted',
          method: 'get',
          params: {
            md: this.form.md,
            drillSequence:this.form.drillSequence
          }
        }).then(res => {
          console.log(res)
          this.form2.userNum1_p = res['data']
        })
      }, 100)
    },
    handleClose(done) {
      if (this.loading) {
        return;
      }
      this.$confirm('确定要提交表单吗？')
        .then(_ => {
          // this.loading = true;
          this.timer = setTimeout(() => {
            done();
            // 动画关闭需要一定的时间
            setTimeout(() => {
              // this.loading = false;
            }, 400);
          }, 200);
        })
        .catch(_ => {
        });
    },
    cancelForm() {
      this.loading = false;
      this.dialog = false;
      clearTimeout(this.timer);
    },
    getScipy() {
      setTimeout(() => {
        request({
          url: 'dataMining/scipy',
          method: 'get',
        }).then(res => {
          console.log(res)
          this.data = []
          this.data = res['data'][0]
          this.chartData = []
          this.chartData = this.getChartData(1)
          this.learnParams = undefined
          this.learnParams = res['data'][1]
          this.initChart(1,this.chartData,this.$refs.well)
        })
      }, 100)
    },
    getRefit(flag) {
      setTimeout(() => {
        let data
        if(flag === 2){
          data = this.data2
        }else {
          data = this.data
        }
        request({
          url: 'dataMining/refit',
          method: 'post',
          data: {
            fitData: data
          }
        }).then(res => {

          if(flag === 2){
            this.data2 = []
            this.data2 = JSON.parse(res)
            this.chartData2 = []
            this.chartData2 = this.getChartData(2)
            this.initChart(2,this.chartData2,this.$refs.well2)
          }else {
            this.data = []
            this.data = JSON.parse(res)
            this.chartData = []
            this.chartData = this.getChartData(1)
            this.initChart(1,this.chartData,this.$refs.well)
          }
        })
      }, 100)
    },
    handlePredicted(){
      this.dialog = true
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
        legend: {
          data: ['井深', '实际钻井工期', '理论钻井工期']
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
          name: '钻井顺序',
          nameLocation: 'middle',
          nameTextStyle: {
            padding: [12, 12, 12, 12]
          },
          type: 'category',
          data: chartData.drillSequence,
          axisPointer: {
            type: 'shadow'
          },
          axisLabel: {
            fontSize: 15,
            color: '#000'
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
            min: 0,
            axisLabel: {
              formatter: '{value}',
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
            name: '工期（天）',
            min: 0,
            axisLabel: {
              formatter: '{value}',
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

        this.chart2.on('click',function(params){
          //点击的柱子的名称
          console.log(params.componentIndex)
          // var index = params.componentIndex % color.length
          // color[index] = "#111111"
          //柱形图重构
          this.chart2 = echarts.init(myChart,'macarons') // 'macarons'
          this.chart2.setOption(option);
        })
      }else {
        this.chart = echarts.init(myChart,'macarons') // 'macarons'
        this.chart.setOption(option)

        this.chart.on('click',function(params){
          //点击的柱子的名称
          console.log(params.componentIndex)
          // var index = params.componentIndex % color.length
          // color[index] = "#111111"
          //柱形图重构
          this.chart = echarts.init(myChart,'macarons') // 'macarons'
          this.chart.setOption(option);
        })
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
.dashboard-editor-container {
  padding: 32px;
  /*background-color: rgb(240, 242, 245);*/
  position: relative;
}
.el-drawer__body {
  overflow: auto;
}
.demo-drawer__footer{
  text-align: center;
  margin-top: 10px;
  margin-bottom: 10px;
}
.form {
  margin-left: 20px;
}
</style>
