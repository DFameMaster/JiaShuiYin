<template>
  <div class="show-image-container">
    <div class="canvas-wrapper">
      <canvas ref="canvasRef" class="watermark-canvas"></canvas>
    </div>
    
    <!-- 底部信息栏 -->
    <div class="image-info-bar">
      <span class="filename">{{ filename }}</span>
      <button @click="changeImage" class="change-image-btn">
        重新选择图片
      </button>
    </div>
    
    <!-- 下载按钮组 -->
    <div class="download-section">
      <button @click="downloadImage('png')" class="download-btn">
         下载 PNG
      </button>
      <button @click="downloadImage('jpeg')" class="download-btn">
         下载 JPG
      </button>
      <button @click="downloadImage('webp')" class="download-btn">
         下载 WebP
      </button>
      <button @click="downloadImage('bmp')" class="download-btn">
         下载 BMP
      </button>
      <button @click="downloadImage('gif')" class="download-btn">
         下载 GIF
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted, nextTick } from 'vue'

const props = defineProps({
  imageUrl: {
    type: String,
    required: true
  },
  watermarkConfig: {
    type: Object,
    default: () => ({})
  },
  filename: {
    type: String,
    default: 'image.png'
  }
})

const emit = defineEmits(['change-image', 'image-size'])

const canvasRef = ref(null)
const image = ref(null)

const drawWatermark = () => {
  const canvas = canvasRef.value
  if (!canvas) return

  const ctx = canvas.getContext('2d', { willReadFrequently: true })

  if (!image.value) {
    image.value = new Image()
    image.value.crossOrigin = 'anonymous'
  }

  image.value.onload = () => {
    const img = image.value
    canvas.width = img.width
    canvas.height = img.height

    // 发出图像尺寸信息
    emit('image-size', { width: img.width, height: img.height })

    ctx.clearRect(0, 0, canvas.width, canvas.height)
    ctx.drawImage(img, 0, 0)

    const config = props.watermarkConfig
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
  }

  image.value.src = props.imageUrl
}

const downloadImage = (format) => {
  const canvas = canvasRef.value
  if (!canvas) return

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

  const dataURL = canvas.toDataURL(mimeType, 0.9)
  const link = document.createElement('a')
  link.download = `qitongtingyu_${Date.now()}.${extension}`
  link.href = dataURL
  link.click()
}

const changeImage = () => {
  emit('change-image')
}

const initCanvas = async () => {
  await nextTick()
  drawWatermark()
}

watch(
  () => [props.imageUrl, props.watermarkConfig],
  () => {
    initCanvas()
  },
  { deep: true }
)

onMounted(() => {
  initCanvas()
})
</script>

<style scoped>
.show-image-container {
  width: 100%;
}

.canvas-wrapper {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 500px;
  background: repeating-conic-gradient(#f0f0f0 0% 25%, #ffffff 0% 50%) 50% / 20px 20px;
  border: 1px solid #e0e0e0;
  border-radius: 12px;
  overflow: hidden;
  transition: all 0.3s ease;
}

.watermark-canvas {
  max-width: 100%;
  max-height: 500px;
  border-radius: 8px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease;
}

.watermark-canvas:hover {
  transform: scale(1.01);
}

/* 底部信息栏 */
.image-info-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 15px;
  padding: 12px 16px;
  background-color: #f8f9fa;
  border-radius: 8px;
  border: 1px solid #e9ecef;
}

.filename {
  font-size: 14px;
  color: #495057;
  font-weight: 500;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  max-width: 70%;
}

.change-image-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 16px;
  background-color: #6c757d;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 13px;
  font-weight: 500;
  transition: all 0.2s ease;
}

.change-image-btn:hover {
  background-color: #5a6268;
  transform: translateY(-1px);
}

/* 下载按钮组 */
.download-section {
  display: flex;
  justify-content: center;
  gap: 12px;
  margin-top: 20px;
  padding-top: 20px;
  border-top: 1px solid #e9ecef;
}

.download-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 10px 20px;
  background-color: #4a90e2;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: all 0.2s ease;
}

.download-btn:hover {
  background-color: #357abd;
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(74, 144, 226, 0.3);
}

@media (max-width: 768px) {
  .canvas-wrapper {
    min-height: 400px;
  }
  
  .image-info-bar {
    flex-direction: column;
    gap: 10px;
    text-align: center;
  }
  
  .filename {
    max-width: 100%;
  }
  
  .change-image-btn {
    width: 100%;
    justify-content: center;
  }
  
  .download-section {
    flex-direction: column;
    gap: 8px;
  }
  
  .download-btn {
    width: 100%;
    justify-content: center;
  }
}
</style>