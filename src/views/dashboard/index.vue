<template>
  <div class="dashboard-container">
    <el-row :gutter="20">
      <el-col :span="6">
        <el-card class="data-card">
          <template #header>
            <div class="card-header">
              <span>总用户数</span>
              <el-icon><User /></el-icon>
            </div>
          </template>
          <div class="card-value">1,234</div>
          <div class="card-footer">较昨日 +12</div>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card class="data-card">
          <template #header>
            <div class="card-header">
              <span>今日活跃</span>
              <el-icon><View /></el-icon>
            </div>
          </template>
          <div class="card-value">423</div>
          <div class="card-footer">较昨日 +5</div>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card class="data-card">
          <template #header>
            <div class="card-header">
              <span>系统消息</span>
              <el-icon><Message /></el-icon>
            </div>
          </template>
          <div class="card-value">45</div>
          <div class="card-footer">较昨日 +8</div>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card class="data-card">
          <template #header>
            <div class="card-header">
              <span>待处理任务</span>
              <el-icon><List /></el-icon>
            </div>
          </template>
          <div class="card-value">15</div>
          <div class="card-footer">较昨日 -3</div>
        </el-card>
      </el-col>
    </el-row>

    <el-row :gutter="20" class="chart-row">
      <el-col :span="12">
        <el-card class="chart-card">
          <template #header>
            <div class="card-header">
              <span>用户增长趋势</span>
            </div>
          </template>
          <div class="chart-placeholder">
            <el-empty description="暂无数据" />
          </div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card class="chart-card">
          <template #header>
            <div class="card-header">
              <span>活跃度分析</span>
            </div>
          </template>
          <div class="chart-placeholder">
            <el-empty description="暂无数据" />
          </div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script setup lang="ts">
import { User, View, Message, List } from '@element-plus/icons-vue'
</script>

<style scoped>
.dashboard-container {
  padding: 20px;
}

.data-card {
  margin-bottom: 20px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.card-value {
  font-size: 28px;
  font-weight: bold;
  color: #303133;
  margin: 10px 0;
}

.card-footer {
  font-size: 12px;
  color: #67c23a;
}

.chart-row {
  margin-top: 20px;
}

.chart-card {
  margin-bottom: 20px;
}

.chart-placeholder {
  height: 300px;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>