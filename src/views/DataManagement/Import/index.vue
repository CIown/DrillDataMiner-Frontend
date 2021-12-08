<template>
  <div class="dashboard-container">
    <div class="dashboard-editor-container">
      <el-upload
        class="upload-demo"
        drag
        action="http://192.168.23.132:5000/dataManagement/uploadData"
        :name="fileName"
        :data="database"
        multiple>
        <i class="el-icon-upload"></i>
        <div class="el-upload__text">请将文件拖拽入此处完成上传或 <em>点击此处进行上传</em></div>
        <div class="el-upload__tip" slot="tip">上传位于WellView安装文件中user/databas/*.mdb文件/</div>
      </el-upload>
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
      fileName : "",
      database : {access:""},
      132: 'http://192.168.23.132:5000/dataManagement/uploadData',
    }
  },
  mounted() {
    // this.uploadUrl()
  },
  methods: {
    uploadUrl() {
      // const filename = wellName
      const formData = new FormData()
      formData.append('file', param.file)
      // formData.append('filename', filename)
      setTimeout(() => {
        // axios.post(this.url,formData,{
        //   headers: {
        //     'Content-Type': 'multipart/form-data;'
        //   }}
        // ).then((response) => {
        //   if(response.data.code === 200){
        //     const h = this.$createElement;
        //     this.$notify({
        //       title: '标题名称',
        //       message: h('i', { style: 'color: teal'}, '文件上传成功')
        //     });
        //   }
        // }).catch((error) => {
        //   console.log(this.url,error)
        // })

      }, 100)
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
