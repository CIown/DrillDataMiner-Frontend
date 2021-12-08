<script src="../../../../api/DataMining/LayerRecommend.js"></script>

<template>
  <div class="dashboard-container">
    <div class="dashboard-editor-container">
    <el-row :gutter="32">
      <el-col style="margin-top: 50px">
        <div>
          <span >井型 :</span>
          <el-select style="margin-top: 10px; margin-bottom: 10px" v-model="value" placeholder="全部">
            <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" :disabled="item.disabled">
            </el-option>
          </el-select>
        </div>
        <div ref="chart" :class="className" :style="{height:height,width:width}" />
      </el-col>
    </el-row>
  </div>
  </div>
</template>

<script>

import * as echarts from 'echarts'
require('echarts/theme/macarons') // echarts theme
import { debounce } from '@/utils'
import request from "../../../utils/request";

var icon = "image://data:image/png;base64,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"


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
      chart: null,
      data: [],
      chartData: [],
      types: {
        '灰岩': 'rgb(127,127,127)' , // 深灰
        '砾岩': 'rgb(91,155,213)' , // 棕黄
        '泥岩': 'rgb(127,127,127)' , // 浅灰
        '砂岩': 'rgb(255,192,0)' , // 黄色
        '岩浆岩': 'rgb(197,90,17)', // 红棕
      },
      options: [{
        value: '选项1',
        label: 'HZ26-6-1',
        disabled: true
      },{
        value: '选项2',
        label: 'HZ26-6-2',
        disabled: true
      },{
        value: '选项3',
        label: 'HZ26-6-3',
      },{
        value: '选项4',
        label: 'HZ26-6-4',
        disabled: true
      }],
      value: 'HZ26-6-3'
    }
  },
  mounted() {
    this.getLayerRecommend()
    this.__resizeHandler = debounce(() => {
      if (this.chart) {
        this.chart.resize()
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
    getLayerRecommend() {
      setTimeout(() => {
        request({
          url: 'dataMining/layerRecommend',
          method: 'get',
        }).then(res => {
          this.data = JSON.parse(res)
          console.log(this.data)
          this.initChart()
        })
      }, 100)
    },
    initChart() {
      var data0 = [];
      var ydata = [];
      var categories = ['categoryA'];
      var types = [
        { name: '灰岩', color: 'rgb(127,127,127)' }, // 深灰
        { name: '砾岩', color: 'rgb(91,155,213)' }, // 棕黄
        { name: '泥岩', color: 'rgb(127,127,127)' }, // 浅灰
        { name: '砂岩', color: 'rgb(255,192,0)' }, // 黄色
        { name: '岩浆岩', color: 'rgb(197,90,17)' }, // 红棕
      ]
      var sum = 0
      var bar = []
      var _this = this
      this.data.forEach(function (item, index) {
        _this.types[item[5]]
        ydata.push(item[0])
        data0.push({
          name: item[5],
          value: [
            index,
            sum,
            sum += item[0],
            item[0]
          ],
          itemStyle: {
            normal: {
              color: _this.types[item[5]]
            }
          }
        })
        bar.push({
          name: 'default',
          type: 'bar',
          stack: 'default',
          // emphasis: {
          //   focus: 'series'
          // },
          xAxisIndex: 4,
          yAxisIndex: 4,
          data: [item[6]],
          itemStyle: {
            color: _this.types[item[5]]
          }
        },)
      })
      console.log(this.data)
      function renderItem(params, api) {
        var categoryIndex = api.value(0);
        var start = api.coord([categoryIndex,api.value(1),]);
        var end = api.coord([categoryIndex,api.value(2),]);
        var width = api.size([0, 1])[0] * 0.4 ;

        return  {
          type: 'rect',
          transition: ['shape'],
          shape: {
            x: start[0] - width / 2,
            y: start[1] ,
            height: end[1] - start[1],
            width: width
          },
          style: api.style()
        };
      }

      this.chart = echarts.init(this.$refs.chart, 'macarons')
      this.chart.setOption({
        title: [
          {
            text:'钻压 (klbs)',
            top: '1%',
            right:'81%',
            textAlign:'center'
          },
          {
            text:'转速 (rpm)',
            top: '1%',
            right:'63%',
            textAlign:'center'
          },
          {
            text:'机械转速 (m/h)',
            top: '1%',
            right:'40%',
            textAlign:'center'
          },
          {
            text:'可钻性级值',
            top: '1%',
            right:'27%',
            textAlign:'center'
          },
          {
            text:'岩性',
            top: '1%',
            right:'17%',
            textAlign:'center'
          }
        ],
        legend: {
          orient: 'vertical',
          right: '5%',
          bottom: '11%',
          itemHeight: 150, //  修改icon图形大小
          itemWidth: 50,
          icon: icon
        },
        tooltip: {
          //trigger: 'axis',
          formatter: function(params) {
            // if (params.seriesIndex === 0) {
            //   return params.seriesName + '\n' + params.axisValue;
            // }
            // if (params.seriesIndex === 1) {
            //   return params.seriesName + '\n' + params.axisValue;
            // }
            if (params.seriesIndex === 4) {
              return params.marker + params.name + ': ' + params.value[3] + ' m'
            }
          }
        },

        grid: [
          { right: '81%', top: '12%', width: '15%', },
          { right: '63%', top: '12%', width: '15%', },
          { right: '45%', top: '12%', width: '15%', },
          { right: '27%', top: '12%', width: '15%', },
          { right: '12%', top: '12%', width: '18%', },
        ],
        yAxis: [
          {
            type: 'category',
            gridIndex: 0,
            // position: 'right',
            data: this.data.map(function(item) {
              return item[0]
            }),
            inverse: 'true',
            axisPointer: {
              show: true
            }
          },
          {
            type: 'category',
            show: false,
            gridIndex: 1,
            data: this.data.map(function(item) {
              return item[0]
            }),
            inverse: 'true',
            axisPointer: {
              show: true
            }
          },
          {
            type: 'category',
            show: false,
            gridIndex: 2,
            data: this.data.map(function(item) {
              return item[0]
            }),
            inverse: 'true',
            axisPointer: {
              show: true
            }
          },
          {
            type: 'category',
            show: false,
            gridIndex: 3,
            data: this.data.map(function(item) {
              return item[0]
            }),
            inverse: 'true',
            axisPointer: {
              show: true
            }
          },
          {
            type: 'value', // 设置为category dataZoom放大到最大会消失不见
            show: false,
            gridIndex: 4,
            scale: true,
            inverse: 'true',
          }],
        xAxis: [
          {
            type: 'value',
            gridIndex: 0,
            position: 'top',
            max: 'dataMax'
          },
          {
            type: 'value',
            gridIndex: 1,
            position: 'top',
            max: 'dataMax',
            show: true
          },
          {
            type: 'value',
            gridIndex: 2,
            position: 'top',
            max: 'dataMax',
            show: true
          },
          {
            type: 'value',
            gridIndex: 3,
            position: 'top',
            max: 'dataMax',
            show: true
          },
          {
            type: 'category',
            gridIndex: 4,
            data: ' ',
            show: false
          }
        ],
        toolbox: {
          right: 10,
          show: true,
          feature: {
            dataZoom: {
              xAxisIndex: 'none'
            },
            dataView: {readOnly: false},
            magicType: {type: ['line', 'bar']},
            restore: {},
            saveAsImage: {}
          }
        },
        dataZoom: [
          {
            show: true,
            yAxisIndex: [0, 1, 2, 3, 4],
            type: 'slider',
            filterMode: 'weakFilter',
            showDataShadow: false,
            labelFormatter: '',
            start: 0,
            end: 100
          },
          {
            type: 'inside', // 滑轮缩放
            yAxisIndex: [0, 1, 2, 3, 4],
            filterMode: 'weakFilter',
            start: 0,
            end: 100
          }],
        series: [
          {
            name: ' ',
            type: 'line',
            data: this.data.map(function(item) {
              return item[1];
            }),
            // data: this.data,
            xAxisIndex: 0,
            yAxisIndex: 0,
          },
          {
            type: 'line',
            data: this.data.map(function (item) {
              return item[2];
            }),
            //data: this.data,
            xAxisIndex: 1,
            yAxisIndex: 1,
          },
          {
            type: 'line',
            data: this.data.map(function (item) {
              return item[3];
            }),
            //data: this.data,
            xAxisIndex: 2,
            yAxisIndex: 2,
          },
          {
            type: 'line',
            data: this.data.map(function (item) {
              return item[4];
            }),
            //data: this.data,
            xAxisIndex: 3,
            yAxisIndex: 3,
          },
          {
            type: 'custom',
            clip: 'true',
            xAxisIndex: 4,
            yAxisIndex: 4,
            renderItem: renderItem,
            itemStyle: {
              opacity: 0.8
            },
            encode: {
              y: [1, 2],
              x: 0
            },
            data: data0
          }
          ]
      })
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
</style>
