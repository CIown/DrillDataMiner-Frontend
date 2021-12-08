
<template>
  <div class="dashboard-container">
    <div class="dashboard-editor-container">
    <el-row :gutter="32">
      <el-col style="margin-top:40px">
        <h4 style="margin-top:10px;text-align: center">钻井工程进度图</h4>
        <div>
          <span >井名 :</span>
<!--          <el-select style="margin-top: 10px; margin-bottom: 10px" v-model="value" placeholder="全部">-->
<!--            <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" :disabled="item.disabled " @click.native="getWellLabel(item)">-->
<!--            </el-option>-->
<!--          </el-select>-->

          <el-select v-model="value" filterable placeholder="请选择">
            <template slot="empty">
              <div>
                <p style="padding-left: 10px">正在加载中 <i class="el-icon-loading" ></i></p>
              </div>
            </template>
            <el-option
              v-for="item in wellNameOptions"
              :key="item.value"
              :label="item.label"
              :value="item.label"
              @click.native="getWellLabel(item)">
            </el-option>
          </el-select>
        </div>
        <!--        <el-table-->
        <!--          :data="data"-->
        <!--          :row-class-name="tableRowClassName"-->
        <!--          :cell-class-name="tableColumnClassName"-->
        <!--          style="width: 100%; margin-top: 20px"-->
        <!--          border-->
        <!--        >-->
        <!--          <el-table-column-->
        <!--            v-for="(item) in activeWellDataPropLabelArray"-->
        <!--            :key="item.prop"-->
        <!--            align="left"-->
        <!--            :prop="item.prop"-->
        <!--            :label="item.label"-->
        <!--          >-->
        <!--            &lt;!&ndash;-->
        <!--            <template slot-scope="scope">-->
        <!--              {{ scope.row[item.prop]?scope.row[item.prop]:'暂无数据' }}-->
        <!--            </template>-->
        <!--            &ndash;&gt;-->
        <!--            <template slot-scope="scope" >-->
        <!--              &lt;!&ndash;  在表格中编辑  &ndash;&gt;-->
        <!--              <div class="inputDeep">-->
        <!--                <el-input v-model="scope.row[item.prop]" size="small" placeholder="请输入内容" :disabled=false :readonly="isReadonlyFlag['flag'+scope.$index]  === undefined ? true : isReadonly['flag'+scope.$index]" @change="handle(1,scope.$index, scope.row,item.prop,scope.row[item.prop])" />-->
        <!--              </div>-->
        <!--            </template>-->
        <!--          </el-table-column>-->
        <!--          <el-table-column label="操作">-->
        <!--            <template slot-scope="scope">-->
        <!--              <el-button-->
        <!--                size="mini"-->
        <!--                @click="handleEdit(1,scope.$index, scope.row)">编辑</el-button>-->
        <!--              <el-button-->
        <!--                size="mini"-->
        <!--                type="danger"-->
        <!--                @click="handleDelete(1,scope.$index, scope.row)">删除</el-button>-->
        <!--            </template>-->
        <!--          </el-table-column>-->
        <!--        </el-table>-->
        <!--        <el-button :loading="downloadLoading" style="margin-top:20px; float: right; margin-right: 10px;" type="primary" icon="el-icon-document">-->
        <!--          Export Excel-->
        <!--        </el-button>-->
      </el-col>
      <el-col >
        <div ref="well" :class="className" :style="{height:height,width:width}" style="margin-top: 10px" />
      </el-col>
      <el-col>
        <div ref="well2" :class="className" :style="{height:height,width:width}" style="margin-top: 10px" />
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
      isReadonlyFlag: {},
      data: [],
      chartData: [],
      downloadLoading: false,
      wellNameOptions: null,
      value: 'HZ26-6-1',
      learnParams: undefined
    }
  },
  mounted() {
    this.getWellName()
    this.getAllWellNames()
    this.getLayerScipy(this.value)
    this.__resizeHandler = debounce(() => {
      this.chart.resize()
    }, 100)
    window.addEventListener('resize', this.__resizeHandler)
  },
  beforeDestroy() {
    this.chart.dispose()
    this.chart = null
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
    getAllWellNames(){
      setTimeout(() => {
        const _this = this
        request({
          url: 'statistics/wellName',
          method: 'get',
        }).then(res => {
          var wellNames = []
          this.wellNameOptions = []
          let index = 1
          wellNames = res['wellName']
          // console.log(111111111)
          wellNames.forEach((name) => {
            _this.wellNameOptions.push({
              value: index,
              label: name,
            });
            index++
          })
        })
      }, 100)
    },
    getLayerScipy(wellname) {
      var _this = this
      setTimeout(() => {
        request({
          url: 'statistics/schedule',
          method: 'get',
          params: {
            wellName: wellname
          }
        }).then(res => {
          this.data = null
          this.data = res
          this.initChart(1,this.data,this.$refs.well)
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
    initChart(flag,data, myChart) {
      console.log(111,data['plan'])
      var option = {
        legend: {
          type: 'scroll',
          width: '80%',
        },
        grid: {
          bottom:'25%'
        },
        toolbox: {
          show: true,
          orient: 'vertical',
          // 竖向居中
          top: '7%',
          right: '5%',
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
        yAxis: {
          type: 'value',
          // data: this.data.map(function(map){
          //   return map['Depth']
          // }),
          name: '结束深度(m)',
          inverse: true,
          axisPointer: {
            show: true
          },
        },
        xAxis: [
          {
            type: 'value',
            name: '时间(days)',
            position: 'top',
            // boundaryGap: ['40%', '40%'],
            minInterval: 0.2
          },
        ],
        series: [{
          name: '计划进度',
          type: 'line',
          data:  data['plan'],
          smooth: false,
          symbol: 'circle',
          symbolSize: 10,
          lineStyle: {
            // color: '#5470C6',
            width: 4,
            // 虚线
            type: 'dashed'
          },
          emphasis: {
            focus: 'series',
            blurScope: 'coordinateSystem'
          }
        },
          {
            name: '实际进度',
            type: 'line',
            data:  data['actual'],
            smooth: false,
            symbol: 'rect',
            symbolSize: 10,
            lineStyle: {
              // color: '#5470C6',
              width: 4,

            },
            emphasis: {
              focus: 'series',
              blurScope: 'coordinateSystem'
            }
          }]
      };
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
