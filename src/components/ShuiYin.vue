<template>
  <div class="shuiyin-app">
    <h1 class="app-title">加水印-qitongtingyu</h1>
    
    <!-- 上传前只显示上传区域 -->
    <div v-if="images.length === 0" class="upload-section">
      <UploadImage @images-selected="handleImagesSelected" />
    </div>
    
    <!-- 上传后显示预览和配置 -->
    <template v-else>
      <div class="main-content">
        <div class="config-section">
          <ConfigPanel 
            v-model="watermarkConfig" 
            :imageSize="currentImageSize"
            :selectedImages="selectedImages"
            :allSelected="allSelected"
            @batch-download="handleBatchDownload"
            @select-all="selectAll"
          />
        </div>
        
        <div class="preview-section">
          <!-- 图片选择器 -->
          <ImageSelector 
            :images="images"
            :activeIndex="currentImageIndex"
            :selectedImages="selectedImages"
            @select-image="selectImage"
            @toggle-select="toggleSelect"
            @delete-image="deleteImage"
            @add-images="addImages"
          />
          
          <!-- 主要预览 -->
          <ShowImage 
            :imageUrl="currentImage?.url" 
            :watermarkConfig="watermarkConfig" 
            :filename="currentImage?.name || 'image.png'"
            @change-image="showUpload" 
            @image-size="handleImageSize" 
          />
        </div>
      </div>
    </template>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import UploadImage from './UploadImage.vue'
import ShowImage from './ShowImage.vue'
import ConfigPanel from './ConfigPanel.vue'
import ImageSelector from './ImageSelector.vue'
import { defaultWatermarkConfig } from '../config/defaultConfig.js'

const images = ref([])
const currentImageIndex = ref(0)
const selectedImages = ref([])
const imageSize = ref({ width: 800, height: 600 })
const watermarkConfig = ref({ ...defaultWatermarkConfig })

const currentImage = computed(() => {
  return images.value[currentImageIndex.value]
})

const currentImageSize = computed(() => {
  return imageSize.value
})

const allSelected = computed(() => {
  return selectedImages.value.length === images.value.length
})

const handleImagesSelected = (newImages) => {
  images.value = newImages
  currentImageIndex.value = 0
  selectedImages.value = []
  if (newImages.length > 0) {
    // 预加载第一张图片以获取尺寸
    const img = new Image()
    img.onload = () => {
      imageSize.value = { width: img.width, height: img.height }
    }
    img.src = newImages[0].url
  }
}

const handleImageSize = (size) => {
  imageSize.value = size
}

const selectImage = (index) => {
  currentImageIndex.value = index
}

const toggleSelect = (index) => {
  const idx = selectedImages.value.indexOf(index)
  if (idx > -1) {
    selectedImages.value.splice(idx, 1)
  } else {
    selectedImages.value.push(index)
  }
}

const selectAll = () => {
  if (allSelected.value) {
    selectedImages.value = []
  } else {
    selectedImages.value = images.value.map((_, index) => index)
  }
}

const deleteImage = (index) => {
  images.value.splice(index, 1)
  // 更新选中状态
  selectedImages.value = selectedImages.value.filter(i => i < index || i > index)
  // 如果删除的是当前选中的图片，调整当前索引
  if (currentImageIndex.value === index) {
    currentImageIndex.value = Math.min(index, images.value.length - 1)
  } else if (currentImageIndex.value > index) {
    currentImageIndex.value--
  }
}

const addImages = (newImages) => {
  images.value = [...images.value, ...newImages]
}

const handleBatchDownload = (options) => {
  if (selectedImages.value.length === 0) return
  
  const { format, namingMode, customName, nameSeparator = '-', numberFormat = '4' } = options
  
  // 为每张选中的图片生成水印并下载
  selectedImages.value.forEach((index, idx) => {
    const image = images.value[index]
    const canvas = document.createElement('canvas')
    const ctx = canvas.getContext('2d', { willReadFrequently: true })
    
    const img = new Image()
    img.onload = () => {
      canvas.width = img.width
      canvas.height = img.height
      
      ctx.clearRect(0, 0, canvas.width, canvas.height)
      ctx.drawImage(img, 0, 0)
      
      // 绘制水印
      const config = watermarkConfig.value
      const text = config.text || '水印文字'
      const fontSize = config.fontSize || 20
      const rotation = config.rotation || -30
      const opacity = (config.opacity || 50) / 100
      const color = config.color || '#000000'
      const spacing = config.spacing || 80
      const offsetX = config.offsetX || 0
      const offsetY = config.offsetY || 0
      const font = config.font || 'Arial'
      const textAlign = config.textAlign || 'center'
      const watermarkStyle = config.watermarkStyle || 'grid'
      const fontWeight = config.fontWeight || 'normal'
      const fontStyle = config.fontStyle || 'normal'

      ctx.save()
      ctx.globalAlpha = opacity
      ctx.fillStyle = color
      ctx.font = `${fontStyle} ${fontWeight} ${fontSize}px ${font}`
      ctx.textAlign = textAlign
      ctx.textBaseline = 'middle'

      const textWidth = ctx.measureText(text).width
      const textHeight = fontSize

      if (watermarkStyle === 'grid') {
        const angle = (rotation * Math.PI) / 180
        const diagonal = Math.sqrt(canvas.width ** 2 + canvas.height ** 2)

        ctx.translate(canvas.width / 2 + offsetX, canvas.height / 2 + offsetY)
        ctx.rotate(angle)

        const stepX = textWidth + spacing
        const stepY = textHeight + spacing

        const cols = Math.ceil(diagonal / stepX) + 4
        const rows = Math.ceil(diagonal / stepY) + 4

        for (let i = -cols / 2; i < cols / 2; i++) {
          for (let j = -rows / 2; j < rows / 2; j++) {
            const x = i * stepX + (j % 2) * (stepX / 2)
            const y = j * stepY
            ctx.fillText(text, x, y)
          }
        }
      } else if (watermarkStyle === 'single') {
        const angle = (rotation * Math.PI) / 180
        ctx.translate(canvas.width / 2 + offsetX, canvas.height / 2 + offsetY)
        ctx.rotate(angle)
        ctx.fillText(text, 0, 0)
      } else if (watermarkStyle === 'corner') {
        const positions = [
          { x: 50, y: 50 },
          { x: canvas.width - 50, y: 50 },
          { x: 50, y: canvas.height - 50 },
          { x: canvas.width - 50, y: canvas.height - 50 }
        ]
        
        positions.forEach(pos => {
          ctx.save()
          ctx.translate(pos.x + offsetX, pos.y + offsetY)
          ctx.rotate((rotation * Math.PI) / 180)
          ctx.fillText(text, 0, 0)
          ctx.restore()
        })
      }

      ctx.restore()
      
      // 根据选择的格式下载图片
      let mimeType = 'image/png'
      let extension = 'png'
      
      if (format === 'jpeg') {
        mimeType = 'image/jpeg'
        extension = 'jpg'
      } else if (format === 'webp') {
        mimeType = 'image/webp'
        extension = 'webp'
      } else if (format === 'bmp') {
        mimeType = 'image/bmp'
        extension = 'bmp'
      } else if (format === 'gif') {
        mimeType = 'image/gif'
        extension = 'gif'
      }
      
      // 生成文件名
      let filename
      if (namingMode === 'original') {
        // 使用原有名称，替换扩展名
        const originalName = image.name.replace(/\.[^/.]+$/, '')
        filename = `${originalName}.${extension}`
      } else {
        // 自定义格式：自定义名称+分隔符+序号
        const paddedIndex = (idx + 1).toString().padStart(parseInt(numberFormat), '0')
        filename = `${customName}${nameSeparator}${paddedIndex}.${extension}`
      }
      
      // 下载图片
      const dataURL = canvas.toDataURL(mimeType, 0.9)
      const link = document.createElement('a')
      link.download = filename
      link.href = dataURL
      
      // 延迟点击以避免浏览器限制
      setTimeout(() => {
        link.click()
      }, idx * 100)
    }
    img.src = image.url
  })
}

const showUpload = () => {
  images.value = []
  currentImageIndex.value = 0
  selectedImages.value = []
  imageSize.value = { width: 800, height: 600 }
}
</script>

<style scoped>
.shuiyin-app {
  padding-top: 20px;
  max-width: 1200px;
  margin: 0 auto;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

.app-title {
  text-align: center;
  color: #333;
  margin-bottom: 40px;
  font-size: 32px;
  font-weight: 700;
  background: linear-gradient(135deg, #4a90e2, #673ab7);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.upload-section {
  margin-bottom: 40px;
  background: white;
  padding: 25px;
  border-radius: 12px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.08);
  transition: box-shadow 0.3s ease;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.main-content {
  display: grid;
  grid-template-columns: 350px 1fr;
  gap: 20px;
}

.preview-section {
  background: white;
  padding: 25px;
  border-radius: 12px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.08);
  transition: box-shadow 0.3s ease;
}

.config-section {
  max-height: 800px;
  overflow-y: auto;
}

.preview-section {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.upload-section:hover,
.preview-section:hover {
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.12);
}

@media (max-width: 992px) {
  .main-content {
    grid-template-columns: 1fr;
  }
  
  .config-section {
    max-height: none;
    overflow-y: visible;
  }
  
  .image-selector {
    justify-content: center;
  }
}

@media (max-width: 768px) {
  .shuiyin-app {
    padding: 15px;
  }
  
  .app-title {
    font-size: 24px;
    margin-bottom: 30px;
  }
  
  .upload-section,
  .preview-section {
    padding: 20px;
    margin-bottom: 30px;
  }
}
</style>