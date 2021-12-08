<script src="../../../../api/Statistics/BaseInfo/WellPosition.js"></script>
<template>
  <div class="dashboard-container">
    <div class="dashboard-editor-container">
    <el-row :gutter="32">
      <el-col>
        <h4 style="margin-top:10px; text-align: center">复杂情况与事故</h4>
        <div>
          <span >井名 :</span>
          <el-select style="margin-top: 10px; margin-bottom: 10px" v-model="wellNameValue" placeholder="全部">
            <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" :disabled="item.disabled " @click.native="getWellLabel(item)">
            </el-option>
          </el-select>
        </div>
        <el-table
          v-if="data.length> 0"
          :data="data"
          border
          style="width: 100%"
          :row-class-name="tableRowClassName"
        >
          <el-table-column
            v-for="(item) in tableHeader"
            :key="item.prop"
            :prop="item.prop"
            :label="item.label"
            :show-overflow-tooltip="true"
            align="left"
          >
            <template slot-scope="scope">
              {{ scope.row[item.prop] }}
            </template>
          </el-table-column>
        </el-table>
        <el-button v-on:click.native="downExcel()" :loading="downloadLoading" style="margin-top:20px; float: right; margin-right: 10px;" type="primary" icon="el-icon-document">
          Export Excel
        </el-button>
      </el-col>
      <el-col>
        <div style="height:80px"></div>
      </el-col>
    </el-row>
  </div>
  </div>
</template>

<script>
import request from "../../../utils/request";
require('echarts/theme/macarons') // echarts theme
import fileDownload from 'js-file-download'

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
      tableHeader: [
        {
          label: '井名',
          prop: 'WellName'
        }, {
          label: '描述',
          prop: 'Des'
        }, {
          label: '复杂情况与事故类型',
          prop: 'Typ'
        },
        {
          label: '发生时间',
          prop: 'DtTmStart'
        },
        {
          label: '结束时间',
          prop: 'DtTmEnd'
        },
        {
          label: '损失时间(h)',
          prop: 'EstLostTime'
        },
      ],
      data: [],
      //data: origin_data, // 用的是前端造的假数据
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
        label: 'HZ26-6-4'
      }, {
        value: '选项5',
        label: 'HZ26-6-4&Sa'
      }, {
        value: '选项6',
        label: 'HZ26-6-5'
      }, {
        value: '选项7',
        label: 'HZ26-6-6d'
      }, {
        value: '选项8',
        label: 'HZ26-6-7'
      }],
      wellNameValue: 'HZ26-6-1',
      filename : '复杂情况与事故',
      fileIndex: 0
    }
  },
  mounted() {
    this.getWellName()
    this.getJobIntervalProblem(this.wellNameValue)
  },
  methods: {
    getWellName(){
      var _this = this
      setTimeout(() => {
        request({
          url: 'api/getData/wellHeader',
          method: 'get',
        }).then(res => {
          console.log(11111111111,res)
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
    getJobIntervalProblem(wellName) {
      var _this = this
      console.log(wellName)
      setTimeout(() => {
        request({
          url: 'statistics/jobIntervalProblem',
          method: 'get',
          params:{
            wellName: wellName
          }
        }).then(res => {
          this.data = null
          this.data = res.data
        })
      }, 100)
    },
    tableRowClassName({ row, rowIndex }) {
      if (rowIndex % 2 !== 1) {
        return 'success-row'
      }
      return ''
    },
    handleEdit(index, row, key, value) {
    },
    getWellLabel(item) {
      this.getJobIntervalProblem(item.label)
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
