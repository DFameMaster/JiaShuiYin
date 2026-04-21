<template>
  <div class="config-panel">
    <div class="panel-header">
      <h3 class="panel-title">水印配置</h3>
      <button @click="resetConfig" class="reset-btn">
        <span class="btn-icon">🔄</span> 重置
      </button>
    </div>

    <div class="config-grid">
      <div class="config-item">
        <label class="config-label">水印内容</label>
        <input
          v-model="config.text"
          @input="updateConfig"
          type="text"
          placeholder="请输入水印文字"
          class="config-input"
        />
      </div>

      <div class="config-item">
        <label class="config-label">水印样式</label>
        <select v-model="config.watermarkStyle" @change="updateConfig" class="config-select">
          <option v-for="style in watermarkStyleOptions" :key="style.value" :value="style.value">
            {{ style.label }}
          </option>
        </select>
      </div>
      <div class="config-item">
        <label class="config-label">字体选择</label>
        <select v-model="config.font" @change="updateConfig" class="config-select">
          <option v-for="font in fontOptions" :key="font.value" :value="font.value">
            {{ font.label }}
          </option>
        </select>
      </div>

      <div class="config-item">
        <label class="config-label">字体大小: {{ config.fontSize }}px</label>
        <input
          v-model.number="config.fontSize"
          @input="updateConfig"
          type="range"
          min="12"
          max="72"
          class="config-slider"
        />
      </div>

      <div class="config-item">
        <label class="config-label">文字对齐</label>
        <select v-model="config.textAlign" @change="updateConfig" class="config-select">
          <option v-for="align in textAlignOptions" :key="align.value" :value="align.value">
            {{ align.label }}
          </option>
        </select>
      </div>

      <div class="config-item">
        <label class="config-label">水印角度: {{ config.rotation }}°</label>
        <input
          v-model.number="config.rotation"
          @input="updateConfig"
          type="range"
          min="-180"
          max="180"
          class="config-slider"
        />
      </div>

      <div class="config-item">
        <label class="config-label">水印透明度: {{ config.opacity }}%</label>
        <input
          v-model.number="config.opacity"
          @input="updateConfig"
          type="range"
          min="1"
          max="100"
          class="config-slider"
        />
      </div>

      <div class="config-item">
        <label class="config-label">水印颜色</label>
        <div class="color-input-group">
          <input
            v-model="config.color"
            @input="updateConfig"
            type="color"
            class="config-color"
          />
          <input
            v-model="config.color"
            @input="updateConfig"
            type="text"
            placeholder="#000000"
            class="color-value-input"
          />
        </div>
      </div>

      <div class="config-item">
        <label class="config-label">水印间距: {{ config.spacing }}px</label>
        <input
          v-model.number="config.spacing"
          @input="updateConfig"
          type="range"
          :min="20"
          :max="maxSpacing"
          class="config-slider"
        />
      </div>

      <div class="config-item">
        <label class="config-label">X轴偏移: {{ config.offsetX }}px</label>
        <input
          v-model.number="config.offsetX"
          @input="updateConfig"
          type="range"
          :min="-maxOffset"
          :max="maxOffset"
          class="config-slider"
        />
      </div>

      <div class="config-item">
        <label class="config-label">Y轴偏移: {{ config.offsetY }}px</label>
        <input
          v-model.number="config.offsetY"
          @input="updateConfig"
          type="range"
          :min="-maxOffset"
          :max="maxOffset"
          class="config-slider"
        />
      </div>

      <div class="config-item">
        <label class="config-label">字体粗细</label>
        <select v-model="config.fontWeight" @change="updateConfig" class="config-select">
          <option v-for="weight in fontWeightOptions" :key="weight.value" :value="weight.value">
            {{ weight.label }}
          </option>
        </select>
      </div>

      <div class="config-item">
        <label class="config-label">文字样式</label>
        <select v-model="config.fontStyle" @change="updateConfig" class="config-select">
          <option v-for="style in fontStyleOptions" :key="style.value" :value="style.value">
            {{ style.label }}
          </option>
        </select>
      </div>
    </div>
    
    <!-- 批量操作 -->
    <div class="batch-download-section">
      <h4 class="batch-title">批量操作</h4>
      
      <div class="batch-format-select">
        <label class="format-label">下载格式：</label>
        <select v-model="batchFormat" class="format-select">
          <option value="png">PNG</option>
          <option value="jpeg">JPG</option>
          <option value="webp">WebP</option>
          <option value="bmp">BMP</option>
          <option value="gif">GIF</option>
        </select>
      </div>
      
      <div class="batch-naming-select">
        <label class="format-label">命名方式：</label>
        <select v-model="namingMode" class="format-select">
          <option value="original">使用原有名称</option>
          <option value="custom">自定义格式</option>
        </select>
      </div>
      
      <div v-if="namingMode === 'custom'" class="batch-naming-input">
        <div class="batch-naming-select">
          <label class="format-label">自定义名称：</label>
          <input 
            v-model="customName" 
            type="text" 
            placeholder="输入基础名称" 
            class="config-input"
          />
        </div>
        
        <div class="batch-separator-select">
          <label class="format-label">分隔符：</label>
          <input 
            v-model="nameSeparator" 
            type="text" 
            placeholder="输入分隔符" 
            class="config-input separator-input"
            maxlength="2"
          />
        </div>
        
        <div class="batch-number-format-select">
        <label class="format-label">序号格式：</label>
        <select v-model="numberFormat" class="format-select">
          <option value="5">00001 (5位)</option>
          <option value="4">0001 (4位)</option>
          <option value="3">001 (3位)</option>
          <option value="2">01 (2位)</option>
          <option value="1">1 (1位)</option>
        </select>
      </div>
        
        <div class="naming-preview">
          预览：{{ customName }}{{ nameSeparator }}{{ padNumber(1, numberFormat) }}.{{ batchFormat }}
        </div>
      </div>
      
      <button 
        @click="batchDownload" 
        class="batch-download-btn"
        :disabled="selectedImages.length === 0"
      >
        <span class="btn-icon">📥</span> 批量下载 ({{ selectedImages.length }})
      </button>
      <button @click="selectAll" class="select-all-btn">
        {{ allSelected ? '取消全选' : '全选' }}
      </button>
    </div>
  </div>
</template>

<script setup>
import { reactive, watch, computed, ref } from 'vue'
import { defaultWatermarkConfig } from '../config/defaultConfig.js'

const props = defineProps({
  modelValue: {
    type: Object,
    default: () => ({})
  },
  imageSize: {
    type: Object,
    default: () => ({ width: 800, height: 600 })
  },
  selectedImages: {
    type: Array,
    default: () => []
  },
  allSelected: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['update:modelValue', 'batch-download', 'select-all'])

const watermarkStyleOptions = [
  { value: 'grid', label: '网格铺满' },
  { value: 'single', label: '单个水印' },
  { value: 'corner', label: '四角水印' }
]

const fontOptions = [
  { value: 'Arial', label: 'Arial' },
  { value: 'SimSun', label: '宋体' },
  { value: 'SimHei', label: '黑体' },
  { value: 'Microsoft YaHei', label: '微软雅黑' },
  { value: 'Times New Roman', label: 'Times New Roman' },
  { value: 'Courier New', label: 'Courier New' }
]

const textAlignOptions = [
  { value: 'center', label: '居中' },
  { value: 'left', label: '左对齐' },
  { value: 'right', label: '右对齐' }
]

const fontWeightOptions = [
  { value: 'normal', label: '正常' },
  { value: 'bold', label: '粗体' },
  { value: 'lighter', label: '细体' }
]

const fontStyleOptions = [
  { value: 'normal', label: '正常' },
  { value: 'italic', label: '斜体' }
]

const config = reactive({ ...defaultWatermarkConfig })
const batchFormat = ref('png')
const namingMode = ref('original')
const customName = ref('栖桐听雨')
const nameSeparator = ref('-')
const numberFormat = ref('3')

const maxSpacing = computed(() => {
  return Math.max(200, Math.min(props.imageSize.width, props.imageSize.height) / 2)
})

const maxOffset = computed(() => {
  return Math.max(100, Math.max(props.imageSize.width, props.imageSize.height) / 2)
})

watch(
  () => props.modelValue,
  (newVal) => {
    if (newVal && Object.keys(newVal).length > 0) {
      Object.assign(config, newVal)
    }
  },
  { immediate: true, deep: true }
)

const updateConfig = () => {
  emit('update:modelValue', { ...config })
}

const resetConfig = () => {
  Object.assign(config, defaultWatermarkConfig)
  updateConfig()
}

const padNumber = (num, length) => {
  return num.toString().padStart(parseInt(length), '0')
}

const batchDownload = () => {
  emit('batch-download', {
    format: batchFormat.value,
    namingMode: namingMode.value,
    customName: customName.value,
    nameSeparator: nameSeparator.value,
    numberFormat: numberFormat.value
  })
}

const selectAll = () => {
  emit('select-all')
}
</script>

<style scoped>
.config-panel {
  width: 100%;
  background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
  border-radius: 12px;
  padding: 25px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.08);
}

.panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 25px;
  padding-bottom: 15px;
  border-bottom: 2px solid #dee2e6;
}

.panel-title {
  margin: 0;
  font-size: 20px;
  color: #333;
  font-weight: 600;
  background: linear-gradient(135deg, #4a90e2, #673ab7);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.reset-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 16px;
  background-color: #6c757d;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: all 0.2s ease;
}

.reset-btn:hover {
  background-color: #5a6268;
  transform: translateY(-1px);
}

.btn-icon {
  font-size: 14px;
}

.config-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 20px;
}

.config-item {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.config-label {
  font-size: 14px;
  font-weight: 500;
  color: #495057;
  margin-bottom: 4px;
}

.config-input {
  padding: 10px 12px;
  border: 1px solid #ced4da;
  border-radius: 6px;
  font-size: 14px;
  transition: all 0.2s ease;
  background-color: white;
}

.config-select {
  padding: 10px 12px;
  border: 1px solid #ced4da;
  border-radius: 6px;
  font-size: 14px;
  transition: all 0.2s ease;
  background-color: white;
  cursor: pointer;
}

.config-input:focus,
.config-select:focus {
  outline: none;
  border-color: #4a90e2;
  box-shadow: 0 0 0 3px rgba(74, 144, 226, 0.1);
}

.config-slider {
  width: 100%;
  height: 8px;
  border-radius: 4px;
  background: #e9ecef;
  appearance: none;
  cursor: pointer;
  transition: background 0.2s ease;
}

.config-slider:hover {
  background: #dee2e6;
}

.config-slider::-webkit-slider-thumb {
  appearance: none;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #4a90e2;
  cursor: pointer;
  box-shadow: 0 2px 6px rgba(74, 144, 226, 0.3);
  transition: all 0.2s ease;
}

.config-slider::-webkit-slider-thumb:hover {
  transform: scale(1.1);
  box-shadow: 0 3px 8px rgba(74, 144, 226, 0.4);
}

.color-input-group {
  display: flex;
  align-items: center;
  gap: 12px;
}

.config-color {
  width: 50px;
  height: 40px;
  border: 1px solid #ced4da;
  border-radius: 6px;
  cursor: pointer;
  transition: transform 0.2s ease;
}

.config-color:hover {
  transform: scale(1.05);
}

.color-value-input {
  font-size: 14px;
  color: #6c757d;
  font-family: 'Courier New', monospace;
  background-color: white;
  padding: 8px 12px;
  border-radius: 4px;
  border: 1px solid #e9ecef;
  flex: 1;
  transition: all 0.2s ease;
}

.color-value-input:focus {
  outline: none;
  border-color: #4a90e2;
  box-shadow: 0 0 0 3px rgba(74, 144, 226, 0.1);
}

@media (max-width: 768px) {
  .config-panel {
    padding: 20px;
  }
  
  .panel-title {
    font-size: 18px;
  }
  
  .panel-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 10px;
  }
  
  .reset-btn {
    width: 100%;
    justify-content: center;
  }
  
  .config-item {
    gap: 6px;
  }
  
  .color-input-group {
    flex-direction: column;
    align-items: flex-start;
  }
  
  .color-value-input {
    width: 100%;
  }
}

.batch-download-section {
  margin-top: 30px;
  padding-top: 20px;
  border-top: 2px solid #dee2e6;
}

.batch-title {
  margin: 0 0 15px 0;
  font-size: 16px;
  color: #333;
  font-weight: 600;
}

.batch-format-select,
.batch-naming-select,
.batch-separator-select,
.batch-number-format-select {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 15px;
}

.batch-naming-input {
  margin-bottom: 15px;
}

.naming-preview {
  margin-top: 8px;
  padding: 8px 12px;
  background-color: #f8f9fa;
  border-radius: 6px;
  font-size: 14px;
  color: #6c757d;
  font-style: italic;
}

.format-label {
  font-size: 14px;
  color: #495057;
  font-weight: 500;
  width: 80px;
  display: inline-block;
  white-space: nowrap;
}

.format-select {
  padding: 8px 12px;
  border: 1px solid #ced4da;
  border-radius: 6px;
  font-size: 14px;
  transition: all 0.2s ease;
  background-color: white;
  cursor: pointer;
  flex: 1;
}

.format-select:focus {
  outline: none;
  border-color: #4a90e2;
  box-shadow: 0 0 0 3px rgba(74, 144, 226, 0.1);
}

.batch-download-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  width: 100%;
  padding: 12px 20px;
  background: linear-gradient(135deg, #4a90e2, #673ab7);
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 15px;
  font-weight: 600;
  transition: all 0.3s ease;
  margin-bottom: 10px;
}

.batch-download-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(74, 144, 226, 0.4);
}

.batch-download-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.select-all-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  padding: 10px 20px;
  background-color: #6c757d;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: all 0.2s ease;
}

.select-all-btn:hover {
  background-color: #5a6268;
  transform: translateY(-1px);
}

@media (max-width: 768px) {
  .batch-format-select,
  .batch-naming-select,
  .batch-separator-select,
  .batch-number-format-select {
    flex-direction: column;
    align-items: flex-start;
    gap: 5px;
  }
  
  .format-select {
    width: 100%;
  }
  
  .batch-naming-input {
    width: 100%;
  }
  
  .config-input {
    width: 100%;
  }
  
  .separator-input {
    width: 100%;
  }
}
</style>