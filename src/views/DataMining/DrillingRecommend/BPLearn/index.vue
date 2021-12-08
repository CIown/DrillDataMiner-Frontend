<script src="../../../../api/DataMining/LearningFormation.js"></script>
<template>
  <div class="dashboard-container">
    <div class="dashboard-editor-container">
    <el-form ref="form" :model="form" label-width="80px">
    <el-row :gutter="20">
      <el-col :span="12" >
<!--          <el-form-item label="选井">-->
<!--            <el-select v-model="form.wellName" multiple filterable placeholder="请选择">-->
<!--              <el-option-->
<!--                v-for="item in options"-->
<!--                :key="item.value"-->
<!--                :label="item.label"-->
<!--                :value="item.label">-->
<!--              </el-option>-->
<!--            </el-select>-->
<!--          </el-form-item>-->
          <el-form-item label="井列表" style="border: black solid 1px" >
            <el-transfer
              filterable
              filter-placeholder="请输入参数名称"
              :titles="['井列表', '已选井']"
              v-model="form.wellName"
              :data="wellNamesTransferData">
            </el-transfer>
          </el-form-item>
      </el-col>
      <el-col :span="12" >
        <el-form-item label="参数" style="border: black solid 1px">
          <el-transfer
            filterable
            :titles="['参数列表', '已选参数']"
            filter-placeholder="请输入参数名称"
            v-model="form.wellParams"
            :data="wellParamsTransferData">
          </el-transfer>
        </el-form-item>
      </el-col>
    </el-row>
      <el-row>
      <el-col :span="8">
        <p>中间节点</p>
        <el-form-item >
          <el-input-number v-model="form.nodeNum" :min="10" :max="100" label="中间节点"></el-input-number>
        </el-form-item>
      </el-col>
      <el-col :span="8">
        <p>训练集比例</p>
        <el-form-item>
          <el-slider
            v-model="form.trainNum"
            :step="5"
            :min="50"
            :max="100"
            :marks="marks"
            show-stops>
          </el-slider>
        </el-form-item>
      </el-col>
      <el-col :span="8" style="text-align: center">
        <p >最大迭代次数</p>
        <el-form-item >
          <el-input-number v-model="form.iterationNum" :min="1" clearable></el-input-number>
        </el-form-item>
      </el-col>
      </el-row>
      <el-row>
      <el-col :span="8" style="float: left">
        <p>模型名称</p>
        <el-form-item >
          <el-input v-model="form.modelName" clearable></el-input>
        </el-form-item>
      </el-col>
      <el-col :span="24" style="text-align: center">
        <el-form-item >
          <el-button
            type="primary"
            @click="onSubmit"
            :disabled="disabled"
            :loading="loadingBtn">
            {{loadingBtn?'正在训练':'开始训练'}}
          </el-button>
          <el-button
            type="primary"
            @click="onWatch"
            :disabled="disabled"
            :loading="loadingBtn">
            {{loadingBtn?'正在查看':'查看结果'}}
          </el-button>
          <el-button
            @click="onReset"
            :disabled="disabled">
            重置参数
          </el-button>
        </el-form-item>
      </el-col>
      <el-col style="margin-top: 10px">
        <div ref="chart" :class="className" :style="{height:height,width:width}" />
      </el-col>
      <el-col style="margin-top: 10px">
        <div ref="chart2" :class="className" :style="{height:height,width:width}" />
      </el-col>
      <el-col style="margin-top: 10px">
        <div ref="chart3" :class="className" :style="{height:height,width:width}" />
      </el-col>
    </el-row>
    </el-form>
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
    const generateData = _ => {
      const wellParamsTransferData = [];
      const wellParams = [
        //  后端已添加
        // 'DEPTH',
        'WOBA',
        'HKLA',
        'RPMA', //RPMA.RPM 或者 RPM
        'TQA',
        'SPPA',
        'MDIA',
        'MFOA',
        'BDTI',
        // 'BDDI',
        'MFIA',
        'TVA',
        'ECDT',
        // 'MFFP',
        // 'DECD'
      ];
      wellParams.forEach((param) => {
        wellParamsTransferData.push({
          label: param,
          key: param,
          disabled: param === 'WOBA' || param === 'RPMA'
        });
      });

      return wellParamsTransferData;
    };
    return {
      transferValue: [],
      wellNames: [
        'DEPTH',
        'WOBA',],
      form: {
        wellName: ['HZ26-6-1'],
        wellParams: ['WOBA','RPMA'],
        nodeNum: 20,
        trainNum: 80,
        iterationNum: 50,
        modelName: 'BPMODEL1',
      },
      wellParamsTransferData: generateData(),
      wellNamesTransferData: [],
      defaultNum: 1,
      isReadonly: true,
      disabled: false,
      loadingBtn: false,
      chart: null,
      chart2:null,
      chart3: null,
      isReadonlyFlag: {},
      ropChartData: null,
      lossChartData: null,
      depthChartData: null,
      downloadLoading: false,
      learnParams: undefined,
      marks: {
        50: '50%',
        80: '80%',
        100: '100%'
      }
    }
  },
  mounted() {
    this.initParams()
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
    this.chart3.dispose()
    this.chart3 = null
  },
  methods: {
    initParams(){
      setTimeout(() => {
        const _this = this
        request({
          url: 'dataMining/BPLearning',
          method: 'get',
        }).then(res => {
          this.wellNamesTransferData = [];
          var wellNames = []
          wellNames = res
          wellNames.forEach((name) => {
            _this.wellNamesTransferData.push({
              label: name,
              key: name,
              // disabled: name === 'WOBA' || name === 'RPM'
            });
          })
        })
      }, 100)
    },
    onSubmit() {
      this.disabled = true
      this.loadingBtn = true
      setTimeout(() => {
        request({
          url: 'dataMining/BPLearning',
          method: 'post',
          data: {
            paramsData: JSON.stringify(this.form)
          }
        }).then(res => {
          console.log(res)
          if(res.status === 2){
            this.disabled = false
            this.loadingBtn = false
            const h = this.$createElement;
            this.$notify({
              title: '提示',
              message: h('i', { style: 'color: teal'}, res.message)
            });
            return;
          }
          console.log('success')
          this.disabled = false
          this.loadingBtn = false
        })
      }, 100)

    },
    onWatch() {
      this.disabled = true
      this.loadingBtn = true
      setTimeout(() => {
        request({
          url: 'dataMining/watchLearnData',
          method: 'get',
        }).then(res => {
          if(res.status === 3){
            this.disabled = false
            this.loadingBtn = false
            const h = this.$createElement;
            this.$notify({
              title: '提示',
              message: h('i', { style: 'color: teal'}, res.message)
            });
            return;
          }
          console.log('55',res)
          let data = res
          this.getChartData(data)
          this.getDepthChartData(data,Object.keys(data['well']).slice(-1))
          this.initChart()
          this.initChart2()
          this.initChart3()
          this.disabled = false
          this.loadingBtn = false
        })
      }, 100)
    },
    onReset(){
      this.form = null
      this.form = {
        wellName: ['HZ26-6-1'],
          wellParams: ['WOBA','RPMA'],
          nodeNum: 20,
          trainNum: 80,
          iterationNum: 50,
          modelName: 'BPMODEL1',
      }
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
      this.ropChartData = {}
      this.lossChartData = {}
      this.ropChartData['practicalRop'] = data['practicalRop']
      this.ropChartData['predictRop'] = data['predictRop']
      this.ropChartData['scatter'] = data['scatter']
      this.lossChartData['mse'] = data['loss']
      console.log(this.ropChartData)
    },
    getDepthChartData(data,wellName) {
      let wellData = data['well'][wellName]
      this.depthChartData = {}
      this.depthChartData['depth'] = wellData['depth']
      this.depthChartData['practicalRop'] = wellData['rop']
      this.depthChartData['predictRop'] = data['predictRop']
      console.log(this.depthChartData)
    },
    initChart() {

      var option = {
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
        tooltip: {
          trigger: 'item',
          axisPointer: {
            type: 'cross'
          }
        },
        dataset: [{
          source: this.ropChartData['scatter']
        }, {
          transform: {
            type: 'ecStat:regression'
            // 'linear' by default.
            // config: { method: 'linear', formulaOn: 'end'}
          }
        }],
        xAxis: {
          name: '实际ROP',
        },
        yAxis: {
          name: '预测ROP',
          scale: true
        },
        series: [{
          name: '预测-实际:ROP',
          type: 'scatter'
        }]
      }
      this.chart = echarts.init(this.$refs.chart,'macarons') // 'macarons'
      this.chart.setOption(option)
    },
    initChart2() {
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
          name: 'epoch',
          data: [...Array(this.lossChartData['mse'].length).keys()].map((el, i) => 1 + i)
        },
        yAxis: {
          type: 'value',
          name: 'MSE',
          scale: true
        },
        series: [{
          name: '收敛曲线',
          data: this.lossChartData['mse'],
          type: 'line'
        }]
      }
      this.chart2 = echarts.init(this.$refs.chart2,'macarons') // 'macarons'
      this.chart2.setOption(option)
    },
    initChart3() {
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
          data: this.depthChartData['depth']
        },
        yAxis: {
          type: 'value',
          name: 'ROP',
          scale: true
        },
        dataZoom: [
          {
            type: 'inside',
            start: 0,
            end: 10
          },
          {
            type: 'slider', // 这个 dataZoom 组件是 slider 型 dataZoom 组件
            start: 0,      // 左边在 10% 的位置。
            end: 10         // 右边在 60% 的位置。
          },
        ],
        series: [
          {
            name: '预测ROP',
            data: this.depthChartData['predictRop'],
            type: 'line'
          },
          {
            name: '实际ROP',
            data: this.depthChartData['practicalRop'],
            type: 'line'
          }]
      }
      this.chart3 = echarts.init(this.$refs.chart3,'macarons') // 'macarons'
      this.chart3.setOption(option)
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
