<script src="../../../../api/Statistics/BaseInfo/WellPosition.js"></script>
<template>
  <div class="dashboard-container">
    <div class="dashboard-editor-container">
    <el-row :gutter="32">
      <el-col>
        <h4 style="margin-top:10px; text-align: center">总井信息</h4>
<!--        <div>-->
<!--          <span >井型 :</span>-->
<!--          <el-select style="margin-top: 10px; margin-bottom: 10px" v-model="value" placeholder="全部">-->
<!--            <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" :disabled="item.disabled">-->
<!--            </el-option>-->
<!--          </el-select>-->
<!--        </div>-->
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
import axios from "axios"
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
      tableHeader: [{
        label: '区块',
        prop: 'fieldName'
      },
        {
        label: '井名',
        prop: 'wellName'
      }, {
        label: 'X坐标(m)',
        prop: 'utmx'
      }, {
        label: 'Y坐标(m)',
        prop: 'utmy'
      },
        {
          label: '井深(m)',
          prop: 'md'
        },
        {
          label: '水深(m)',
          prop: 'waterDepth'
        },
        {
          label: '建井周期(day)',
          prop: 'wellConstructionDuration'
        },
        {
          label: '钻井周期(day)',
          prop: 'userNum1'
        },
        {
          label: '井型',
          prop: 'wellTyp1'
        },
        {
          label: '井别',
          prop: 'wellConfig'
        },
      ],
      data: [],
      //data: origin_data, // 用的是前端造的假数据
      downloadLoading: false,
      options: [{
        value: '选项1',
        label: '探井'
      },{
        value: '选项2',
        label: '开发井',
        disabled: true
      }],
      value: '',
      filename : '总井信息',
      fileIndex: 0
    }
  },
  watch: {
    data: {
      deep: true,
      handler(newVal) {
        if (newVal) {
          this.initChart()
        }
      }
    }
  },
  mounted() {
    this.getWellPosition()
  },
  methods: {
    getWellPosition() {
        setTimeout(() => {
            request({
              url: 'statistics/wellPosition/all',
              method: 'get',
            }).then(res => {
          this.data = null
          this.data = res
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
          var filename = this.filename + this.fileIndex
          fileDownload(res, filename + '.xlsx')
          this.fileIndex = this.fileIndex + 1
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
