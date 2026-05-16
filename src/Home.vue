<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'
import axios from 'axios'


// 响应式状态
//const urlInput = ref('')//URL
const isDragging = ref(false)
const fileInputRef = ref(null)
const navItems = ['识别', '用户管理', 'Github', '文档', '登录']
const activeNav = ref('识别')
const userQuestion = ref('')
const selectedFile = ref(null)
const previewUrl = ref('')
const resultUrl = ref('')
const resultimage = ref('')

// 允许的文件配置
const MAX_FILE_SIZE = 10 * 1024 * 1024 

// 数据绑定
//const imageUrl = ref('')
const isLoading = ref(false)
const resultData = ref('')

// 后端API基础地址
const API_BASE_URL = 'http://localhost:8000/api'

// 文件校验核心逻辑（解决文件类型报错问题）
const validateFile = (event) => {
  // 释放URL对象
  if (previewUrl.value) {
    URL.revokeObjectURL(previewUrl.value)
    previewUrl.value = ''
  }

  const allowedExts = ['.jpg', '.jpeg', '.png', '.webp']
  const fileExt = event.target.files[0]
  // 1. 校验MIME类型
  //if (!ALLOWED_MIME_TYPES.includes(file.type)) {
    //alert('当前不支持该文件类型，请尝试其他文件')
    //return false
  //}

  //2. 校验文件后缀名
  //if (!allowedExts.includes(fileExt)) {
  //  alert('当前不支持该文件类型，请尝试其他文件')
  //  return false
  //}

  // 3. 校验文件大小
  if (event.size > MAX_FILE_SIZE) {
    alert('文件大小不能超过10MB，请尝试其他文件')
    return false
  }

  selectedFile.value = fileExt
  alert('图片上传成功')

  // 生成本地预览URL
  previewUrl.value = URL.createObjectURL(fileExt)
}

// 导航切换方法
const switchNav = (item) => {
  activeNav.value = item
  // 切换页面时清空结果
  resultData.value = ''
}

// 触发文件选择
const triggerFileSelect = () => {
  fileInputRef.value.click()
  //alert(fileInputRef.value)
}

// 文件上传处理
const handleFileChange = async () => {
  if (!selectedFile.value) {
    alert('请先选择一张图片')
    return
  }

  if (!userQuestion.value.trim()) {
    alert('请输入问题')
    return
  }

  isLoading.value = true
  resultData.value = ''

  // 构建FormData
  const formData = new FormData()
  formData.append('file', selectedFile.value)
  formData.append('question', userQuestion.value.trim())

  try {
    const response = await axios.post(`/api/recognize/upload`, formData, {
      headers: {
        'Content-Type': 'multipart/form-data'
      }
    })

    if (response.data.code === 0) {
      resultData.value = response.data.data
      resultimage.value = response.data.data.image
      alert('识别成功！')
    } else {
      alert(`识别失败`)
    }
  } catch (error) {
    console.error('文件上传错误:', error)
    alert(`上传失败：${error.response?.data?.detail || '网络错误，请检查后端是否运行'}`)
  } finally {
    isLoading.value = false
  }
}

// 拖拽相关事件
const handleDragOver = (e) => {
  e.preventDefault()
  e.stopPropagation()
  isDragging.value = true
}

const handleDragLeave = (e) => {
  e.preventDefault()
  e.stopPropagation()
  isDragging.value = false
}

const handleDrop = (e) => {
  e.preventDefault()
  e.stopPropagation()
  isDragging.value = false

  const file = e.dataTransfer?.files?.[0]
  if (!file) return

  validateFile(file) && handleRecognizeFile(file)
}

// 清除选中的文件
const clearSelectedFile = () => {
  selectedFile.value = null
  resultData.value = null
}

// 处理URL识别
/*const handleUrlRecognize = async () => {
  if (!urlInput.value.trim()) {
    alert('请输入有效的图片URL')
    return
  }

  isLoading.value = true
  resultData.value = null

  try {
    const response = await axios.post(`/api/recognize/url`, {
      url: imageUrl.value.trim(),
      highQualityMode: true
    })

    if (response.data.code === 0) {
      resultData.value = response.data.data
      alert('识别成功！')
    } else {
      alert(`识别失败：${response.data.message}`)
    }
  } catch (error) {
    console.error('URL识别错误:', error)
    alert(`识别失败：${error.response?.data?.detail || '网络错误，请检查后端是否运行'}`)
  } finally {
    isLoading.value = false
  }
}
*/

// 处理文件识别
const handleRecognizeFile = (file) => {
  // 后续可在这里添加上传文件到识别接口的逻辑
  console.log('开始识别文件：', file)
}

// 处理剪贴板粘贴图片
const handlePaste = (event) => {
  const clipboardData = event.clipboardData || window.clipboardData
}

// 生命周期：监听粘贴事件
onMounted(() => {
  window.addEventListener('paste', handlePaste)
})

onUnmounted(() => {
  window.removeEventListener('paste', handlePaste)
})
</script>

<template>
  <div class="home-container">
    <!-- 顶部导航栏 -->
    <header class="header-nav">
      <div class="nav">
        <div class="logo">
          <span class="logo-text">图像识别</span>
        </div>
        <el-button
          v-for="item in navItems"
          :key="item"
          :class="{ 
            'recognize-nav-btn': activeNav === item,
            'non-recognize-nav-btn': activeNav !== item
            }"
          @click="switchNav(item)"
        >
          {{ item }}
        </el-button>
      </div>
    </header>
    <!-- 主内容区 -->
    <main v-if="activeNav === '识别'" class="main-content">
      <!-- 左侧上传识别区 -->
      <section class="upload-section">
        <div class="upload-header">
          <div class="upload-icon">
            <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M12 3L12 15" stroke="#6366F1" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
              <path d="M5 10L12 3L19 10" stroke="#6366F1" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
          </div>
          <h1 class="upload-title">图像识别</h1>
          <p class="upload-subtitle">图像识别</p>
        </div>
        <!-- 问题输入区域 -->
        <div class="question-input-area">
          <el-input
            v-model="userQuestion"
            type="textarea"
            :rows="3"
            placeholder="请输入问题..."
            :disabled="isLoading"
            class="question-input"
            @paste="handlePaste"
          ></el-input>
          </div>
        <!-- URL输入区域 
        <div class="url-input-area">
          <label class="input-label">
            <svg class="label-icon" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
              <path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
            请输入URL
          </label>
          <div class="url-input-group">
            <el-input
              v-model="urlInput"
              placeholder="请输入URL"
              class="url-input"
            />
            <el-button type="primary" @click="handleUrlRecognize" class="url-btn">
              <svg class="btn-small-icon" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path d="M11 19H5C3.89543 19 3 18.1046 3 17V7C3 5.89543 3.89543 5 5 5H19C20.1046 5 21 5.89543 21 7V13" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                <path d="M16 17L21 22L26 17" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                <path d="M21 17V22" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
              识别
            </el-button>
          </div>
        </div>
        -->

        <!-- 文件上传区域 -->
        <div class="file-upload-area">
          <label class="upload-label">
            <svg class="label-icon" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M14 2H6C4.89543 2 4 2.89543 4 4V20C4 21.1046 4.89543 22 6 22H18C19.1046 22 20 21.1046 20 20V8L14 2Z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
              <path d="M14 2V8H20" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
            文件上传
          </label>

          <!-- 拖拽上传容器 -->
           <input
              ref="fileInputRef"
              type="file"
              class="file-input"
              accept=".jpg,.jpeg,.png,.webp"
              @change="validateFile"
              hidden
            />
          <div 
            v-if="!selectedFile" 
            class="drop-container"
            :class="{ 'is-dragover': isDragging }"
            @drop="handleDrop" 
            @dragover="handleDragOver"
            @dragleave="handleDragLeave"
            @click="triggerFileSelect"
          >
            <div class="drop-icon">
              <svg viewBox="0 0 48 48" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path d="M24 38V10" stroke="#6366F1" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"/>
                <path d="M12 22L24 10L36 22" stroke="#6366F1" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"/>
                <rect x="8" y="30" width="32" height="10" rx="2" fill="#6366F1" fill-opacity="0.2"/>
              </svg>
            </div>
            <p class="drop-title">拖拽上传图片</p>
            <p class="drop-subtitle">
              或者 <span class="click-select">点击选择</span>
            </p>
            <div class="format-tip">
              支持格式: JPEG, JPG, PNG, WEBP (最大10MB)
            </div>
          </div>
          <div v-else class="drop-container" @click="clearSelectedFile" :disabled="isLoading">
            <img :src="previewUrl" alt="预览图" class="preview-image">
          </div>
        </div>

        <!-- 底部提示 -->
        <div class="bottom-tips">
          <p class="tip-item">
            <svg class="tip-icon" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M12 22C17.5228 22 22 17.5228 22 12C22 6.47715 17.5228 2 12 2C6.47715 2 2 6.47715 2 12C2 17.5228 6.25329 22 12 22Z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
              <path d="M12 16V12" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
              <path d="M12 8H12.01" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
            图片支持拖拽、粘贴、上传
          </p>
          <el-button 
            type="primary" 
            @click="handleFileChange" 
            :loading="isLoading"
            :disabled="isLoading || !selectedFile"
            class="submit-button"
          >
            识别
          </el-button>
        </div>
      </section>

      <!-- 右侧结果展示区 -->
      <section class="result-section">
        <div class="result-header">
          <div class="result-eye-icon">
            <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M2 12C2 12 6 6 12 6C18 6 22 12 22 12C22 12 18 18 12 18C6 18 2 12 2 12Z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
              <path d="M12 15C13.6569 15 15 13.6569 15 12C15 10.3431 13.6569 9 12 9C10.3431 9 9 10.3431 9 12C9 13.6569 10.3431 15 12 15Z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
          </div>
          <h2 class="result-title">识别结果</h2>
          <p class="result-subtitle">识别结果</p>
        </div>

        <!-- 结果空状态/内容区 -->
        <div class="result-content">
          <!-- 加载状态 -->
          <div v-if="isLoading" class="loading-state">
            <svg class="loading-icon" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <circle cx="12" cy="12" r="10" stroke="#6366F1" stroke-width="2" stroke-linecap="round" stroke-dasharray="30 70"/>
            </svg>
            <p class="loading-text">AI正在思考中，请稍候...</p>
          </div>

          <div v-else-if="!resultData" class="empty-state">
            <svg class="empty-icon" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <rect x="3" y="3" width="18" height="18" rx="2" ry="2" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
              <circle cx="8.5" cy="8.5" r="1.5" fill="currentColor"/>
              <path d="M21 15L16 10L5 21" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
            <p class="empty-text">上传图片后，结果将显示在这里</p>
          </div>
          <div v-else class="result-success">
            <!-- 识别结果内容 -->
            <div class="result-image-container" >
              <img :src="`data:image/jpeg;base64,${resultimage}`" class="result-image"/>
            </div>
            <div class="result-text-container">
              <h3 class="result-text-title">AI Answer</h3>
              <div class="result-text">
                 {{ resultData.answer }}
              </div>
            </div>
          </div>
        </div>
      </section>
    </main>
    <main v-else class="placeholder-page">
      <h2>
          {{ activeNav }}页面
      </h2>
      <p>
        {{ activeNav }}的功能区域还未开发完成，请以后再来吧
      </p>
    </main>
  </div>
</template>

<style scoped>
.home-container {
  width: 100%;
  max-width: 1800px;
  min-height: 100vh;
  background: linear-gradient(180deg, #eac4ff 0%, #ffffff 100%);
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  box-sizing: border-box;
}

/* 顶部导航栏样式 */
.header-nav {
  width: 100%;
  height: 72px;
  padding: 40px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: #ffffff;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.04);
  box-sizing: border-box;
}

.nav {
  display: flex;
  align-items: center;
  gap: 24px;
}

.logo {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 22px;
  font-weight: 800;
  color: #6366F1;
}

.logo-text {
  width: 100px;
}

.recognize-nav-btn {
  background: #bc5fff;
  color:#ffffff;
  display: flex;
  align-items: center;
  width: 84px;
  height: 42px;
  gap: 4px;
  font-weight: 600;
  border: 0;
  border-radius: 8px;
}

.non-recognize-nav-btn {
  background: #ffffff;
  display: flex;
  align-items: center;
  width: 84px;
  height: 42px;
  gap: 4px;
  font-weight: 600;
  border: 0;
  border-radius: 8px;
}

.non-recognize-nav-btn:hover{
  background: #bc5fff;
  color:#ffffff;
  transition:color 1s;
}

.btn-icon {
  width: 16px;
  height: 16px;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: 6px;
  color: #64748b;
  text-decoration: none;
  font-size: 14px;
  transition: color 0.2s;
}

.nav-item:hover {
  color: #6366F1;
}

.nav-icon {
  width: 18px;
  height: 18px;
}

/* 主内容区样式 */
.main-content {
  max-width: 1600px;
  margin: 32px auto;
  padding: 0 24px;
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 32px;
  box-sizing: border-box;
  
}

/* 左侧上传区 */
.upload-section {
  background: #ffffff;
  border-radius: 16px;
  padding: 40px 32px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.05);
}

.upload-header {
  text-align: center;
  margin-bottom: 32px;
}

.upload-icon {
  width: 48px;
  height: 48px;
  margin: 0 auto 16px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.upload-title {
  font-size: 24px;
  font-weight: 700;
  color: #1e293b;
  margin: 0 0 8px;
}

.upload-subtitle {
  font-size: 14px;
  color: #94a3b8;
  margin: 0;
}

/* URL输入区域 */
.url-input-area {
  margin-bottom: 24px;
}

.input-label {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 14px;
  color: #475569;
  margin-bottom: 12px;
}

.label-icon {
  width: 16px;
  height: 16px;
}

.url-input-group {
  display: flex;
  gap: 8px;
}

.url-input {
  flex: 1;
}

.url-btn {
  flex-shrink: 0;
  display: flex;
  align-items: center;
  gap: 4px;
}

.btn-small-icon {
  width: 14px;
  height: 14px;
}

/* 分割线 */
.divider {
  display: flex;
  align-items: center;
  color: #94a3b8;
  font-size: 14px;
  margin: 24px 0;
}

.divider::before,
.divider::after {
  content: '';
  flex: 1;
  height: 1px;
  background: #e2e8f0;
}

.divider span {
  padding: 0 16px;
}

/* 文件上传区域 */
.file-upload-area {
  margin-bottom: 32px;
}

.drop-container {
  margin-top: 12px;
  border: 2px dashed #cbd5e1;
  border-radius: 12px;
  padding: 48px 24px;
  text-align: center;
  background: #f8fafc;
  transition: all 0.3s;
  cursor: pointer;
}

.drop-container.is-dragover {
  border-color: #6366F1;
  background: #eef2ff;
}

.drop-container:hover {
  border-color: #94a3b8;
}

.file-input {
  display: none;
}

.drop-icon {
  width: 64px;
  height: 64px;
  margin: 0 auto 16px;
}

.drop-title {
  font-size: 18px;
  font-weight: 600;
  color: #1e293b;
  margin: 0 0 8px;
}

.drop-subtitle {
  font-size: 14px;
  color: #64748b;
  margin: 0 0 16px;
}

.click-select {
  color: #6366F1;
  font-weight: 600;
  cursor: pointer;
}

.format-tip {
  display: inline-block;
  padding: 6px 12px;
  background: #ffffff;
  border-radius: 6px;
  font-size: 12px;
  color: #94a3b8;
  border: 1px solid #e2e8f0;
}

.preview-image {
  width: 100%;
  max-height: 400px;
  object-fit: contain;
  display: block;
}

/* 底部提示 */
.bottom-tips {
  border-top: 1px solid #e2e8f0;
  padding-top: 20px;
}

.tip-item {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 14px;
  color: #64748b;
  margin: 0 0 12px;
}

.tip-icon {
  width: 16px;
  height: 16px;
  flex-shrink: 0;
}

.quality-checkbox {
  font-size: 14px;
  color: #475569;
}

.submit-button{
  float:right;
}

/* 右侧结果区 */
.result-section {
  background: #ffffff;
  border-radius: 16px;
  padding: 40px 32px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.05);
  display: flex;
  flex-direction: column;
}

.result-header {
  text-align: center;
  margin-bottom: 32px;
}

.result-eye-icon {
  width: 32px;
  height: 32px;
  margin: 0 auto 8px;
  color: #6366F1;
}

.result-title {
  font-size: 22px;
  font-weight: 700;
  color: #1e293b;
  margin: 0 0 8px;
}

.result-subtitle {
  font-size: 14px;
  color: #94a3b8;
  margin: 0;
}

.result-content {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 400px;
  border-top: 1px solid #e2e8f0;
}

.loading-state {
  text-align: center;
  padding: 80px 20px;
}

.loading-icon {
  width: 64px;
  height: 64px;
  color: #6366F1;
  margin: 0 auto 16px;
  animation: rotate 1s linear infinite;
}

.loading-text {
  font-size: 16px;
  color: #6366F1;
  margin: 0;
}

.empty-state {
  text-align: center;
}

.empty-icon {
  width: 64px;
  height: 64px;
  color: #cbd5e1;
  margin: 0 auto 16px;
}

.empty-text {
  font-size: 16px;
  color: #94a3b8;
  margin: 0;
}

.result-success {
  text-align: center;
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.result-image-container {
  width: 100%;
  border-radius: 8px;
  overflow: hidden;
  border: 1px solid #e2e8f0;
}

.result-image {
  width: 100%;
  max-height: 400px;
  object-fit: contain;
  display: block;
}

.result-text-container {
  flex: 1;
}

.result-text-title {
  font-size: 18px;
  font-weight: 600;
  color: #1e293b;
  margin: 0 0 16px;
  padding-bottom: 8px;
  border-bottom: 1px solid #e2e8f0;
}

.result-text {
  max-width: 600px;
  font-size: 15px;
  color: #334155;
  line-height: 1.8;
  white-space: pre-wrap;
  word-wrap: break-word;
}

.placeholder-page {
  text-align: center;
  padding: 60px 20px;
  color: #606266;
}

/* 响应式适配 */
@media (max-width: 1024px) {
  .main-content {
    grid-template-columns: 1fr;
  }

  .nav-right {
    display: none;
  }
}
</style>