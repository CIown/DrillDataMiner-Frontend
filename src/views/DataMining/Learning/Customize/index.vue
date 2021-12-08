
<template>
  <div class="dashboard-container">
    <div class="dashboard-editor-container">
    <el-row :gutter="32">
      <el-col style="margin-top:40px">
        <h4 style="margin-top:10px;text-align: center">钻井工程学习曲线</h4>
        <div>
          <span >区块 :</span>
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
            @click="handleAdd()"
          >添加</el-button>
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            @click="handleDeleteAll()"
          >清空</el-button>
          <el-button
            type="success"
            icon="el-icon-caret-right"
            size="mini"
            @click="handleFit()"
          >重新拟合</el-button>
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
                <el-input v-model="scope.row[item.prop]" size="small" placeholder="请输入内容" :disabled=false :readonly="false" @change="handle(scope.$index, scope.row,item.prop,scope.row[item.prop])" />
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
        <el-button :loading="downloadLoading" style="margin-top:20px; float: right; margin-right: 10px;" type="primary" icon="el-icon-document">
          Export Excel
        </el-button>
      </el-col>
      <el-col>
        <div ref="well" :class="className" :style="{height:height,width:width}" style="margin-top: 10px" />
      </el-col>
    </el-row>
  </div>
  </div>
</template>

<script>
import * as echarts from 'echarts'
import request from '@/utils/request'
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
      isReadonly: false,
      chart: null,
      activeWellDataPropLabelArray: [
        {
          label: '井名',
          prop: 'wellName'
        }, {
          label: '井深(m)',
          prop: 'md'
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
          label: '理论工期(day)',
          prop: 'userNum1_p'
        }],
      isReadonlyFlag: {},
      data: [],
      chartData: [],
      downloadLoading: false,
    }
  },
  watch: {
    data: {
      deep: true,
      handler(newVal) {
        if (newVal) {
          this.chartData = []
          this.chartData = this.getChartData(1)
          this.initChart(this.chartData,this.$refs.well)
        }
      }
    },
  },
  mounted() {
    this.__resizeHandler = debounce(() => {
      this.chart.resize()
    }, 100)
    window.addEventListener('resize', this.__resizeHandler)
  },
  beforeDestroy() {
    if (!this.chart) {
      return
    }
    this.chart.dispose()
    this.chart = null
  },
  methods: {
    getRefit(flag) {
      setTimeout(() => {
        let data
        data = this.data
        request({
          url: 'dataMining/refit',
          method: 'post',
          data: {
            fitData: data
          }
        }).then(res => {
            this.data = []
            this.data = JSON.parse(res)
            this.chartData = []
            this.chartData = this.getChartData()
            this.initChart(this.chartData,this.$refs.well)
        })
      }, 100)
    },
    handleFit(){
      this.getRefit()
    },
    tableRowClassName({ row, rowIndex }) {
      if (rowIndex % 2 !== 1) {
        return 'success-row' + ' row' + rowIndex
      }
      return ''
    },
    tableColumnClassName({flag,row, column, rowIndex, columnIndex}){
      if(columnIndex === 4){
        return 'disable-column'
      }
      return ''
    },
    handle(index, row, key, value){
    },
    handleEdit(index, row, key, value) {

    },
    handleDelete(index, row, key, value) {
        this.data.splice(index, 1)
        delete this.isReadonlyFlag['flag' + index]
    },
    handleDeleteAll(){
        this.data = []
    },

    handleAdd() {
      let obj = {
        'wellName': undefined,
        'md': undefined,
        'drillSequence': undefined,
        'userNum1': undefined,
        'userNum1_p': undefined
      }
      this.data.push(obj)
    },
    getChartData() {
      var userNum1 = []
      var md = []
      var drillSequence = []
      var userNum1_p = []
      var wellName = []
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
    initChart(data, myChart) {
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
            name: '工期（天）',
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
      this.chart = echarts.init(myChart,'macarons') // 'macarons'
      this.chart.setOption(option)

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
