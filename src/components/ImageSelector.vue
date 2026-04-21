<template>
  <div class="image-selector-container">
    <!-- 图片选择器 -->
    <div class="image-selector">
      <div 
        v-for="(image, index) in images" 
        :key="index" 
        class="image-thumbnail"
        :class="{ active: activeIndex === index }"
        @click="selectImage(index)"
      >
        <img :src="image.url" :alt="image.name" class="thumbnail-img" />
        <div class="thumbnail-overlay">
          <input 
            type="checkbox" 
            :checked="selectedImages.includes(index)"
            @change="toggleSelect(index)"
            class="select-checkbox"
          />
          <button class="delete-btn" @click.stop="deleteImage(index)">
            ✕
          </button>
        </div>
      </div>
      
      <!-- 添加更多按钮 -->
      <div class="add-more" @click="triggerAddMore">
        <div class="add-icon">+</div>
        <div class="add-text">添加更多</div>
      </div>
    </div>
    
    <!-- 隐藏的文件输入 -->
    <input
      ref="fileInput"
      @change="handleAddMore"
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

const props = defineProps({
  images: {
    type: Array,
    default: () => []
  },
  activeIndex: {
    type: Number,
    default: 0
  },
  selectedImages: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits([
  'select-image',
  'toggle-select',
  'delete-image',
  'add-images'
])

const fileInput = ref(null)

const selectImage = (index) => {
  emit('select-image', index)
}

const toggleSelect = (index) => {
  emit('toggle-select', index)
}

const deleteImage = (index) => {
  emit('delete-image', index)
}

const triggerAddMore = () => {
  // 重置文件输入框，确保可以重复选择同一文件
  fileInput.value.value = ''
  fileInput.value.click()
}

const handleAddMore = (event) => {
  const files = event.target.files
  if (files.length > 0) {
    processFiles(files, event.target)
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
        emit('add-images', newImages)
        // 处理完所有文件后重置，确保下次可以选择同一文件
        if (fileInput) fileInput.value = ''
      }
    }
    reader.readAsDataURL(file)
  }
}
</script>

<style scoped>
.image-selector-container {
  width: 100%;
}

.image-selector {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  padding: 15px;
  background-color: #f8f9fa;
  border-radius: 8px;
  overflow-x: auto;
  width: 100%;
}

.image-thumbnail {
  position: relative;
  width: 80px;
  height: 80px;
  border-radius: 6px;
  overflow: hidden;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 2px solid transparent;
}

.image-thumbnail.active {
  border-color: #4a90e2;
  box-shadow: 0 0 0 2px rgba(74, 144, 226, 0.2);
}

.thumbnail-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.thumbnail-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.image-thumbnail:hover .thumbnail-overlay {
  opacity: 1;
}

.select-checkbox {
  position: absolute;
  top: 5px;
  left: 5px;
  width: 14px;
  height: 14px;
  cursor: pointer;
  z-index: 2;
}

.delete-btn {
  background: rgba(255, 255, 255, 0.9);
  border: none;
  border-radius: 50%;
  width: 24px;
  height: 24px;
  font-size: 16px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
  color: #333;
}

.delete-btn:hover {
  background: white;
  transform: scale(1.1);
  color: #dc3545;
}

.add-more {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  border: 2px dashed #e0e0e0;
  border-radius: 8px;
  height: 80px;
  width: 80px;
  cursor: pointer;
  transition: all 0.3s ease;
  background-color: #f9f9f9;
}

.add-more:hover {
  border-color: #4a90e2;
  background-color: #f0f7ff;
}

.add-icon {
  font-size: 24px;
  color: #999;
  margin-bottom: 5px;
}

.add-text {
  font-size: 12px;
  color: #666;
}

@media (max-width: 768px) {
  .image-selector {
    justify-content: flex-start;
  }
  
  .image-thumbnail {
    width: 60px;
    height: 60px;
  }
  
  .add-more {
    width: 60px;
    height: 60px;
  }
  
  .add-icon {
    font-size: 20px;
  }
  
  .add-text {
    font-size: 10px;
  }
}
</style>