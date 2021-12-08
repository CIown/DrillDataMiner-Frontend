<template>
  <div class="dashboard-container">
    <div class="dashboard-editor-container">
      <el-row :gutter="32">
        <el-col :span="12">
          <h4 style="margin-top:10px;text-align: center">已存在井</h4>
          <el-table
            v-if="data.length > 0"
            highlight-current-row
            :data="data"
            :row-class-name="tableRowClassName"
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
              <template slot-scope="scope">
                <!--
                <el-input v-model="scope.row[item.prop]" size="small" placeholder="请输入内容" @change="handleEdit(scope.$index, scope.row,item.prop,scope.row[item.prop])" />
                -->
                {{ scope.row[item.prop] }}
              </template>
            </el-table-column>
            <el-table-column
              fixed="right"
              label="操作">
              <template slot-scope="scope">
                <el-select v-model="scope.row[scope.column.property]" placeholder="请选择">
                  <el-option
                    v-for="item in options"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value">
                  </el-option>
                </el-select>
              </template>
            </el-table-column>
            <el-table-column
              fixed="right"
              label="操作"
              width="100">
              <template slot-scope="scope">
                <el-upload
                  class="upload-demo"
                  :action="action"
                  :http-request="(params) =>upload(params,scope.row['wellName'])"
                  :on-preview="handlePreview"
                  :before-remove="beforeRemove"
                  accept=".las"
                  multiple
                  :limit="1"
                  :on-exceed="handleExceed"
                  :file-list="fileList">
                  <el-button size="small" type="primary">点击上传</el-button>
  <!--                <div slot="tip" class="el-upload__tip">只能上传las文件，且不超过</div>-->
                </el-upload>
              </template>
            </el-table-column>
          </el-table>
          </el-col>
      </el-row>
    </div>
  </div>
</template>
<script>

import request from "../../../utils/request";
import axios from "axios";

export default {
  name: "Import",
  data() {
    return {
      fileList: [],
      action: 'http://192.168.23.132:5000' + '/dataManagement/uploadLas',
      activeWellDataPropLabelArray:[{
        label: '井号',
        prop: 'wellName'
      },{
        label: '数据类型',
        prop: 'type'
      }  ],
      currentRow: null,
      data: [],
      options: [{
        value: '选项1',
        label: '测井曲线'
      }, {
        value: '选项2',
        label: '实时数据'
      }, {
        value: '选项3',
        label: '录井数据'
      }],
      value: ''
    }
  },
  mounted() {
    this.getWellName()
  },
  methods: {
    getWellName(){
      var _this = this
      setTimeout(() => {
        request({
          url: 'api/getData/wellHeader',
          method: 'get',
        }).then(res => {
          this.data = res
          for(var i = 0; i <　res.length; i++){
            this.data[i]['type'] = '实时数据'
          }
        })
      }, 100)
    },
    upload(param,wellName) {
      const filename = wellName
      const formData = new FormData()
      formData.append('file', param.file)
      formData.append('filename', filename)
      setTimeout(() => {
        // request({
        //   url: 'api/getData/upload',
        //   method: 'post',
        //   headers: { "Content-Type": "multipart/form-data" },
        //   data: formData,
        // }).then(res => {
        //   console.log('上传文件成功')
        // })
        axios.post(this.action,formData,{
          headers: {
            'Content-Type': 'multipart/form-data;'
          }}
        ).then((response) => {
          if(response.data.code === 200){
            const h = this.$createElement;
            this.$notify({
              title: '标题名称',
              message: h('i', { style: 'color: teal'}, '文件上传成功')
            });
          }
        }).catch((error) => {
            console.log(error)
          })

      }, 100)
    },
    tableRowClassName({ row, rowIndex }) {
      if (rowIndex % 2 !== 1) {
        return 'success-row'
      }
      return ''
    },
    handleRemove(file, fileList) {
      console.log(file, fileList);
    },
    handlePreview(file) {
      console.log(file);
    },
    handleExceed(files, fileList) {
      this.$message.warning(`当前限制选择 1 个文件，本次选择了 ${files.length} 个文件，共选择了 ${files.length + fileList.length} 个文件`);
    },
    beforeRemove(file, fileList) {
      return this.$confirm(`确定移除 ${ file.name }？`);
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
.el-dropdown-link {
  cursor: pointer;
  color: #409EFF;
}
.el-icon-arrow-down {
  font-size: 12px;
}
</style>
