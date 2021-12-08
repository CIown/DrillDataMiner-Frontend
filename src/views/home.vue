<template>
  <div class="dashboard-container" :style="'background-image:url('+ Background +');'">
    <div class="dashboard-editor-container">
      <el-row style="padding:16px 16px 0;margin-bottom:32px;height:500px">
        <h1>钻井数据挖掘与分析系统 </h1>
        <h1><b>D</b>rilling <b>D</b>ata <b>M</b>iner </h1>
        <p> 版本: V.0.2</p>
      </el-row>

      <el-row :gutter="32">
        <el-col :xs="24" :sm="24" :lg="2">
          <br/>
        </el-col>
        <el-col :xs="24" :sm="24" :lg="5">
          <el-button type="primary" icon="el-icon-folder-opened" round><a href="http://192.168.23.128/DataManagement/Import">数据管理</a></el-button>
        </el-col>
        <el-col :xs="24" :sm="24" :lg="5">
          <el-button type="success" icon="el-icon-data-analysis" round><a href="http://192.168.23.128/Statistics/WellSummary">数据分析</a></el-button>
        </el-col>
        <el-col :xs="24" :sm="24" :lg="5">
          <el-button type="info" icon="el-icon-data-line" round><a href="http://192.168.23.128/Mining/Analysis">数据挖掘</a></el-button>
        </el-col>
        <el-col :xs="24" :sm="24" :lg="5">
          <el-button type="warning" icon="el-icon-setting" round><a href="http://192.168.23.128/System/index">系统设置</a></el-button>
        </el-col>
        <el-col :xs="24" :sm="24" :lg="2">
          <br/>
        </el-col>
      </el-row>
    </div>
  </div>
</template>

<script>
import Background from '@/assets/images/background_home.jpg'

export default {
  name: 'Dashboard',
  data() {
    return {
      Background: Background,
    }
  },
  methods: {
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
.dashboard-editor-container  {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  background-size: cover;
}

.dashboard-editor-container {
  padding: 32px;

  .github-corner {
    position: absolute;
    top: 0;
    border: 0;
    right: 0;
  }

  .chart-wrapper {
    background: #fff;
    padding: 16px 16px 0;
    margin-bottom: 32px;
  }
}

@media (max-width:1024px) {
  .chart-wrapper {
    padding: 8px;
  }
}
</style>
