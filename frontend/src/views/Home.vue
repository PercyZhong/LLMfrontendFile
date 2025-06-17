<template>
  <div class="home-container">
    <el-row :gutter="20">
      <!-- 左侧问答区域 -->
      <el-col :span="16">
        <el-card class="qa-card">
          <template #header>
            <div class="card-header">
              <span>智能问答</span>
            </div>
          </template>
          
          <div class="chat-container">
            <div v-for="(message, index) in chatHistory" :key="index" class="message" :class="message.type">
              <el-avatar :icon="message.type === 'user' ? 'User' : 'Assistant'" />
              <div class="message-content">{{ message.content }}</div>
            </div>
          </div>

          <div class="input-area">
            <el-input
              v-model="userInput"
              type="textarea"
              :rows="3"
              placeholder="请输入您的问题..."
              @keyup.enter.ctrl="sendQuestion"
            />
            <el-button type="primary" @click="sendQuestion" :loading="loading">
              发送问题
            </el-button>
          </div>
        </el-card>
      </el-col>

      <!-- 右侧知识库管理区域 -->
      <el-col :span="8">
        <el-card class="kb-card">
          <template #header>
            <div class="card-header">
              <span>知识库管理</span>
            </div>
          </template>

          <el-tabs>
            <el-tab-pane label="网页导入">
              <el-form>
                <el-form-item label="URL">
                  <el-input v-model="webUrl" placeholder="请输入网页URL" />
                </el-form-item>
                <el-button type="primary" @click="importWebPage">导入网页</el-button>
              </el-form>
            </el-tab-pane>

            <el-tab-pane label="文件导入">
              <el-upload
                class="upload-demo"
                drag
                action="/api/upload"
                :on-success="handleUploadSuccess"
                :on-error="handleUploadError"
                :before-upload="beforeUpload"
              >
                <el-icon class="el-icon--upload"><upload-filled /></el-icon>
                <div class="el-upload__text">
                  拖拽文件到此处或 <em>点击上传</em>
                </div>
                <template #tip>
                  <div class="el-upload__tip">
                    支持 .txt, .md, .pdf 格式文件
                  </div>
                </template>
              </el-upload>
            </el-tab-pane>

            <el-tab-pane label="知识库列表">
              <el-table :data="knowledgeBase" style="width: 100%">
                <el-table-column prop="name" label="名称" />
                <el-table-column prop="type" label="类型" />
                <el-table-column prop="date" label="导入时间" />
                <el-table-column label="操作">
                  <template #default="scope">
                    <el-button type="danger" size="small" @click="deleteKnowledge(scope.row)">
                      删除
                    </el-button>
                  </template>
                </el-table-column>
              </el-table>
            </el-tab-pane>
          </el-tabs>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { ElMessage } from 'element-plus'
import { UploadFilled } from '@element-plus/icons-vue'
import axios from 'axios'

const userInput = ref('')
const loading = ref(false)
const webUrl = ref('')
const chatHistory = ref([])
const knowledgeBase = ref([])

const sendQuestion = async () => {
  if (!userInput.value.trim()) return
  
  chatHistory.value.push({
    type: 'user',
    content: userInput.value
  })
  
  loading.value = true
  try {
    const response = await axios.post('/api/ask', {
      question: userInput.value
    })
    
    chatHistory.value.push({
      type: 'assistant',
      content: response.data.answer
    })
  } catch (error) {
    ElMessage.error('获取答案失败，请稍后重试')
  } finally {
    loading.value = false
    userInput.value = ''
  }
}

const importWebPage = async () => {
  if (!webUrl.value) {
    ElMessage.warning('请输入有效的URL')
    return
  }
  
  try {
    await axios.post('/api/import/web', {
      url: webUrl.value
    })
    ElMessage.success('网页导入成功')
    webUrl.value = ''
    loadKnowledgeBase()
  } catch (error) {
    ElMessage.error('网页导入失败')
  }
}

const handleUploadSuccess = (response) => {
  ElMessage.success('文件上传成功')
  loadKnowledgeBase()
}

const handleUploadError = () => {
  ElMessage.error('文件上传失败')
}

const beforeUpload = (file) => {
  const validTypes = ['.txt', '.md', '.pdf']
  const extension = file.name.substring(file.name.lastIndexOf('.')).toLowerCase()
  if (!validTypes.includes(extension)) {
    ElMessage.error('只支持 .txt, .md, .pdf 格式文件')
    return false
  }
  return true
}

const deleteKnowledge = async (row) => {
  try {
    await axios.delete(`/api/knowledge/${row.id}`)
    ElMessage.success('删除成功')
    loadKnowledgeBase()
  } catch (error) {
    ElMessage.error('删除失败')
  }
}

const loadKnowledgeBase = async () => {
  try {
    const response = await axios.get('/api/knowledge')
    knowledgeBase.value = response.data
  } catch (error) {
    ElMessage.error('获取知识库列表失败')
  }
}

// 初始加载知识库列表
loadKnowledgeBase()
</script>

<style scoped>
.home-container {
  padding: 20px;
}

.qa-card, .kb-card {
  margin-bottom: 20px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.chat-container {
  height: 400px;
  overflow-y: auto;
  padding: 20px;
  background: #f9f9f9;
  border-radius: 4px;
  margin-bottom: 20px;
}

.message {
  display: flex;
  margin-bottom: 20px;
  align-items: flex-start;
}

.message.user {
  flex-direction: row-reverse;
}

.message-content {
  margin: 0 10px;
  padding: 10px;
  border-radius: 4px;
  max-width: 70%;
}

.user .message-content {
  background: #409EFF;
  color: white;
}

.assistant .message-content {
  background: #f4f4f5;
}

.input-area {
  display: flex;
  gap: 10px;
}

.input-area .el-button {
  align-self: flex-end;
}
</style> 