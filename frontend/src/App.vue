<template>
  <el-container class="app-container">
    <el-header>
      <h1>智能问答系统-问天</h1>
    </el-header>
    
    <el-main>
      <router-view></router-view>
    </el-main>
  </el-container>
</template>

<script setup>
import { ElContainer, ElHeader, ElMain } from 'element-plus'
</script>

<style scoped>
.app-container {
  min-height: 100vh;
}

.el-header {
  background-color: #409EFF;
  color: white;
  text-align: center;
  line-height: 25px;
  padding: 0;
}

.el-main {
  padding: 20px;
  background-color: #f5f7fa;
}
</style> 