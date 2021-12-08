<template>
  <div class="dashboard-container">
    <div class="dashboard-editor-container">
    <el-row :gutter="32">
      <el-col style="margin-top: 50px" :xs="24" :sm="24" :lg="24">
        <div ref="chart" :class="className" :style="{height:height,width:width}" style="margin-bottom: 10px"/>
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
import request from "../../../../utils/request";
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
      default: '600px'
    }
  },
  data() {
    return {
      chart: null,
      // ？ data = null 图就消失不见了？ = [] 图就能出来 不知道原因是啥
      data: [],
      options: [{
        value: '选项1',
        label: '万山组',
      },
        {
          value: '选项2',
          label: '粤海组',
        },
        {
          value: '选项3',
          label: '韩江组',
        },
        {
          value: '选项4',
          label: '珠海组',
        },
        {
          value: '选项5',
          label: '珠江组上部',
        },
        {
          value: '选项6',
          label: '珠江组下部',
        },{
          value: '选项7',
          label: '文昌组',
        },
        {
          value: '选项8',
          label: '潜山组',
        },
        {
          value: '选项9',
          label: '恩平组',
        },
      ],
      value: '',
      symbols: {
        'pdc': 'image://' + require('@/assets/images/pdc.png'),
        '牙轮钻头': 'image://' + require('@/assets/images/牙轮钻头.png'),
        'Diamond': 'image://' + require('@/assets/images/狮虎钻头.png'),
        'FishTail': 'image://' + require('@/assets/images/狮虎钻头.png'),
      }
    }
  },
  mounted() {
    this.getDrillbitUsage()
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
    getDrillbitUsage() {
      setTimeout(() => {
        request({
          url: 'dataMining/wellDrillbitUsage/all',
          method: 'get',
          params: {
          }
        }).then(res => {
          this.data = []
          this.data = res.data
          this.initChart()
        })
      }, 1)
    },

    initChart() {
      console.log(this.data['columns'],this.data['data'])
      const _this = this
      this.chart = echarts.init(this.$refs.chart ) //'macarons'
      this.chart.setOption({
        legend: {},
        tooltip: {},
        grid: {
          // backgroundColor: 'rgb(6,33,68)',
          // borderColor: '#fff',
          // shadowColor: '#0f375f',
          // show: true,
        },
        dataset: {
          dimensions: this.data['columns'],
          source: this.data['data']
        },
        xAxis: {
          name: '机械钻速(m/h)',
          // nameLocation: 'middle',
          splitLine: { // 网格线
            lineStyle: {
              type: 'solid'  // 'dashed'设置网格线类型 dotted：虚线   solid:实线
            },
            show: true // 隐藏或显示
          },
          axisLabel: {
            show: true,
            lineStyle: {
              color: '#ccc'
            }
          },
        },
        yAxis: {
          type: 'value',
          name: '进尺(m)',
          splitLine: { // 网格线
            lineStyle: {
              type: 'solid'  // 'dashed'设置网格线类型 dotted：虚线   solid:实线
            },
            show: true // 隐藏或显示
          },
          // nameLocation: 'center',
          axisLabel: {
            show: true,
            lineStyle: {
              color: '#ccc'
            }
          },
          // axisLine: {
          //   show: true,
          //   lineStyle: {
          //     color: '#ccc'
          //   }
          // }

        },
        // Declare several bar series, each will be mapped
        // to a column of dataset.source by default.
        series: [{
          // data: this. ,
          type: 'scatter',
          encode: {
            x : '钻速(m/h)',
            y : '进尺(m)',
            tooltip: this.data['columns'],
            // itemName: 1,
          },
          symbol: function(val,param){
            switch(val[5]) {
              case 'PDC钻头':
                return _this.symbols['pdc']
              case '牙轮钻头':
                return _this.symbols['牙轮钻头']
              case 'Fish Tail':
                return _this.symbols['FishTail']
              case 'Diamond':
                return _this.symbols['Diamond']
              default:
                return 'circle'
            }
          },
          symbolSize: 20,
      }]
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

/*@import url("//unpkg.com/element-ui@2.15.5/lib/theme-chalk/index.css");*/

.el-drawer__body {
  overflow: auto;
}

.el-table .success-row{
  background:#f0f9eb;
}
.el-table td, .el-table th {
  padding: 3px 0;
}
</style>
