<template>
  <div class="dashboard-container">
    <div class="dashboard-editor-container">
<!--    style="height: 800px-->
    <el-row :gutter="32">
      <el-col>
        <h4 style="margin-top:10px; text-align: center">钻头优选</h4>
        <!--        <el-select style="margin-top: 10px; margin-bottom: 10px" v-model="value" placeholder="请选择区块">-->
        <!--          <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value">-->
        <!--          </el-option>-->
        <!--        </el-select>-->
        <el-table
          :data="data"
          border
          stripe
          :height="height"
          style="width: 100%"
          :row-class-name="tableRowClassName"
        >
          <!-- v-if="data.length > 0"-->
          <el-table-column
            v-for="(item) in tableHeader"
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
            <template slot-scope="scope">
              <!--
              <el-input v-model="scope.row[item.prop]" size="small" placeholder="请输入内容" @change="handleEdit(scope.$index, scope.row,item.prop,scope.row[item.prop])" />
              -->
              {{ scope.row[item.prop]?scope.row[item.prop]:'暂无数据' }}
            </template>
          </el-table-column>
          <el-table-column
            label="操作"
            fixed="right"
            width="130">
            <template slot-scope="scope">
              <el-button
                size="mini"
                type="text"
                @click="handleCreate(scope.$index, scope.row)">添加</el-button>
              <el-button
                size="mini"
                type="text"
                @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
              <el-button
                size="mini"
                type="text"
                @click="handleDelete(scope.$index, scope.row)">删除</el-button>
            </template>
          </el-table-column>

        </el-table>
        <div class="pagination"  style="text-align: center;margin-top: 30px;">
          <el-pagination
            background
            layout="prev, pager, next"
            @current-change="handleCurrentChange"
            :page-count="pageCount">
          </el-pagination>
        </div>

        <el-button v-on:click.native="downExcel()" style="margin-top:20px; float: right; margin-right: 10px;" type="primary" icon="el-icon-document">
          Export Excel
        </el-button>
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
          <el-form-item label="井名" :label-width="formLabelWidth">
            <el-input v-model="form.wellName" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="下井序号" :label-width="formLabelWidth">
            <el-input v-model="form.serialNumber" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="尺寸(mm)" :label-width="formLabelWidth">
            <el-input v-model="form.sizeMM" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="尺寸(inch)" :label-width="formLabelWidth">
            <el-input v-model="form.sizeInch" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="钻头编号" :label-width="formLabelWidth">
            <el-input v-model="form.drillbitNumber" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="钻头类型" :label-width="formLabelWidth">
            <el-input v-model="form.drillbitType" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="钻头型号" :label-width="formLabelWidth">
            <el-input v-model="form.drillbitModel" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="厂家" :label-width="formLabelWidth">
            <el-input v-model="form.drillingSectionStart" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="起始井深(m)" :label-width="formLabelWidth">
            <el-input v-model="form.drillingSectionEnd" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="终止井深(m)" :label-width="formLabelWidth">
            <el-input v-model="form.factory" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="钻遇地层" :label-width="formLabelWidth">
            <el-input v-model="form.formName" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="进尺(m)" :label-width="formLabelWidth">
            <el-input v-model="form.depthDrilled" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="纯钻时(h)" :label-width="formLabelWidth">
            <el-input v-model="form.tmDrilled" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="纯钻速(h)" :label-width="formLabelWidth">
            <el-input v-model="form.rop" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="钻压(kN)" :label-width="formLabelWidth">
            <el-input v-model="form.wob" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="转速(rmp)" :label-width="formLabelWidth">
            <el-input v-model="form.rpmString" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="排量(L/s)" :label-width="formLabelWidth">
            <el-input v-model="form.liquidInjRate" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="喷嘴(1/32°)" :label-width="formLabelWidth">
            <el-input v-model="form.dia" autocomplete="off"></el-input>
          </el-form-item>
        </el-form>
        <div class="demo-drawer__footer" >
          <el-button @click="cancelForm">取 消</el-button>
          <el-button type="primary" @click="$refs.drawer.closeDrawer()" :loading="loading">{{ loading ? '提交中 ...' : '确 定' }}</el-button>
        </div>
      </div>
    </el-drawer>
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
      default: '35rem'
    }
  },
  data() {
    return {
      dialog: false,
      loading: false,
      timer: null,
      total: 355,
      pageCount: 80,
      currentPage: 1, //默认显示页面为1
      pagesize: 10, //    每页的数据条数
      tableHeader: [
        {
          label: '井名',
          prop: 'wellName'
        }, {
          label: '下井序号',
          prop: 'serialNumber'
        }, {
          label: '尺寸(mm)',
          prop: 'sizeMM'
        },
        {
          label: '尺寸(inch)',
          prop: 'sizeInch'
        },
        {
          label: '钻头编号',
          prop: 'drillbitNumber'
        },
        {
          label: '钻头类型',
          prop: 'drillbitType'
        },
        {
          label: '钻头型号',
          prop: 'drillbitModel'
        }, {
          label: '厂家',
          prop: 'factory'
        }, {
          label: '起始井深(m)',
          prop: 'drillingSectionStart'
        },
        {
          label: '终止井深(m)',
          prop: 'drillingSectionEnd'
        },
        {
          label: '钻遇地层',
          prop: 'formName'
        },
        {
          label: '进尺(m)',
          prop: 'depthDrilled'
        },
        {
          label: '纯钻时(h)',
          prop: 'tmDrilled'
        },
        {
          label: '钻速(m/h)',
          prop: 'rop'
        },
        {
          label: '钻压(kN)',
          prop: 'wob'
        },
        {
          label: '转速(rpm)',
          prop: 'rpmString'
        }, {
          label: '排量(L/s)',
          prop: 'liquidInjRate'
        }, {
          label: '喷嘴(1/32°)',
          prop: 'dia'
        },
      ],
      formLabelWidth: '80px',
      form: {
      },
      // ？ data = null 图就消失不见了？ = [] 图就能出来 不知道原因是啥
      data: [],
      params: null,
      //data: origin_data, // 用的是前端造的假数据
      chartData: null,
      options: [{
        value: '选项1',
        label: 'HZ26-6-3'
      }],
      value: '',
      filename : '钻头优选',
      dataId: null,
      handleMethod: null,
    }
  },
  mounted() {
    this.getCount()
    this.getDrillbitUsage()

  },
  beforeDestroy() {

  },
  methods: {
    getCount() {
      console.log(this.pageCount)
      setTimeout(() => {
        request({
          url: 'dataMining/wellDrillbitUsage/count',
          method: 'get',
        }).then(res => {
          this.pageCount = Math.ceil(res / this.pagesize)
        })
      }, 1)
    },
    getDrillbitUsage() {
      console.log(this.pageCount)
      setTimeout(() => {
        request({
          url: 'dataMining/wellDrillbitUsage/page',
          method: 'get',
          params: {
            page: this.currentPage,
            size: this.pagesize,
          }
        }).then(res => {
          console.log(res.data)
          // this.data = null
          this.data = res.data
        })
      }, 1)
    },
    handleCurrentChange(currentPage) {
      this.currentPage = currentPage;
      this.getDrillbitUsage()
    },
    handleEdit(index, row) {
      this.handleMethod = 'edit'
      this.dialog = true
      this.dataId = this.data[index]['id']
      this.form = {
        id: this.dataId,
        wellName: row['wellName'],
        serialNumber: row['serialNumber'],
        sizeMM: row['sizeMM'],
        sizeInch: row['sizeInch'],
        drillbitNumber: row['drillbitNumber'],
        drillbitType: row['drillbitType'],
        drillbitModel: row['drillbitModel'],
        drillingSectionStart: row['drillingSectionStart'],
        drillingSectionEnd: row['drillingSectionEnd'],
        factory: row['factory'],
        formName: row['formName'],
        tmDrilled: row['tmDrilled'],
        depthDrilled: row['depthDrilled'],
        rop: row['rop'],
        wob: row['wob'],
        rpmString: row['rpmString'],
        liquidInjRate: row['liquidInjRate'],
        dia: row['dia'],
      }
    },
    edit() {
      request({
        url: 'dataMining/wellDrillbitUsage/update',
        method: 'Post',
        data: {
          form: this.form
        }
      }).then(res => {
        this.getDrillbitUsage()
      })

    },
    created() {
      request({
        url: 'dataMining/wellDrillbitUsage/created',
        method: 'post',
        data: {
          form: this.form
        }
      }).then(res => {
        this.getDrillbitUsage()
      })
    },
    handleCreate(index, row) {
      this.handleMethod = 'created'
      this.form = {
        wellName: null,
        serialNumber: null,
        sizeMM: null,
        sizeInch: null,
        drillbitNumber: null,
        drillbitType: null,
        drillbitModel: null,
        drillingSectionStart: null,
        drillingSectionEnd: null,
        factory: null,
        formName: null,
        tmDrilled: null,
        depthDrilled: null,
        rop: null,
        wob: null,
        rpmString: null,
        liquidInjRate: null,
        dia: null,
      }
      this.dialog = true
    },
    handleDelete(index, row) {
      this.dataId = this.data[index]['id']
      this.$confirm('确定要删除吗？')
        .then(_ => {
          this.delete()
        })
        .catch(_ => {
        });
    },
    delete() {
      request({
        url: 'dataMining/wellDrillbitUsage/delete',
        method: 'delete',
        params: {
          id: this.dataId,
        }
      }).then(res => {
        console.log('delete')
        this.getDrillbitUsage()
      })
    },
    handleClose(done) {
      if (this.loading) {
        return;
      }
      this.$confirm('确定要提交表单吗？')
        .then(_ => {
          this.loading = true;
          if (this.handleMethod === 'edit') {
            this.edit()
          } else {
            this.created()
          }
          this.timer = setTimeout(() => {
            done();
            // 动画关闭需要一定的时间
            setTimeout(() => {
              this.loading = false;
            }, 400);
          }, 2000);
        })
        .catch(_ => {
        });
    },
    cancelForm() {
      this.loading = false;
      this.dialog = false;
      clearTimeout(this.timer);
    },
    tableRowClassName({row, rowIndex}) {
      if (rowIndex % 2 !== 1) {
        return 'success-row'
      }
      return ''
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

/*@import url("//unpkg.com/element-ui@2.15.5/lib/theme-chalk/index.css");*/

.el-drawer__body {
  overflow: auto;
}
.demo-drawer__footer{
  text-align: center;
  margin-bottom: 10px;
}
.form {
  margin-left: 20px;
}
.el-table .success-row{
  background:#f0f9eb;
}
.el-table td, .el-table th {
  padding: 3px 0;
}
</style>
