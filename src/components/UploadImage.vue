<template>
  <div 
    class="upload-container"
    @click="triggerUpload"
    @dragover.prevent
    @dragenter.prevent
    @drop.prevent="handleDrop"
  >
    <div class="upload-placeholder">
      <div class="upload-icon">📁</div>
      <div class="upload-text">点击或拖动图片到此处上传</div>
      <div class="upload-hint">支持 JPG、PNG、WebP 等格式，可批量上传</div>
    </div>
    <input
      ref="fileInput"
      @change="handleImageChange"
      type="file"
      name="file"
      id="file"
      accept="image/*"
      multiple
      style="display: none"
    />
  </div>
</template>

<script setup>
import { ref } from 'vue'

const emit = defineEmits(['images-selected'])
const fileInput = ref(null)

const triggerUpload = () => {
  // 重置文件输入框，确保可以重复选择同一文件
  fileInput.value.value = ''
  fileInput.value.click()
}

const handleImageChange = (event) => {
  const files = event.target.files
  if (files.length > 0) {
    processFiles(files, event.target)
  }
}

const handleDrop = (event) => {
  const files = event.dataTransfer.files
  if (files.length > 0) {
    processFiles(files)
  }
}

const processFiles = (files, fileInput) => {
  const newImages = []
  const validFiles = Array.from(files).filter(file => file.type.startsWith('image/'))
  
  if (validFiles.length === 0) {
    if (fileInput) fileInput.value = ''
    return
  }
  
  for (let i = 0; i < validFiles.length; i++) {
    const file = validFiles[i]
    const reader = new FileReader()
    reader.onload = (e) => {
      const imageData = {
        file: file,
        url: e.target.result,
        name: file.name
      }
      newImages.push(imageData)
      
      if (newImages.length === validFiles.length) {
        emit('images-selected', newImages)
        // 处理完所有文件后重置，确保下次可以选择同一文件
        if (fileInput) fileInput.value = ''
      }
    }
    reader.readAsDataURL(file)
  }
}
</script>

<style scoped>
.upload-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  border: 2px dashed #e0e0e0;
  border-radius: 12px;
  padding: 40px 20px;
  text-align: center;
  cursor: pointer;
  transition: all 0.3s ease;
  background-color: #fafafa;
  max-width: 100%;
  width: 100%;
}

.upload-container:hover {
  border-color: #4a90e2;
  background-color: #f0f7ff;
  transform: translateY(-2px);
}

.upload-placeholder {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 15px;
  padding: 20px;
}

.upload-icon {
  font-size: 48px;
  color: #999;
}

.upload-text {
  color: #333;
  font-size: 18px;
  font-weight: 500;
}

.upload-hint {
  color: #999;
  font-size: 14px;
  margin-top: 5px;
  max-width: 300px;
}

@media (max-width: 768px) {
  .upload-container {
    padding: 30px 15px;
  }
  
  .upload-text {
    font-size: 16px;
  }
}
</style>