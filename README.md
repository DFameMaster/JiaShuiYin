# 加水印-栖桐听雨

一个功能强大、易于使用的图片水印添加工具，支持批量处理和多种格式导出。

## 功能特性

### 🌊 核心功能

- **图片上传**：支持单张和多张图片上传，支持拖动上传
- **水印配置**：
  - 水印文字内容
  - 字体大小、颜色、透明度
  - 水印角度、间距
  - X轴和Y轴偏移
  - 字体选择
  - 水印样式（网格、单个、四角）
  - 字体粗细和样式
- **实时预览**：即时查看水印效果
- **批量操作**：
  - 全选/取消全选
  - 批量下载
  - 自定义下载格式
  - 灵活的命名规则

### 📁 支持的格式

- **输入格式**：支持常见图片格式（JPG、PNG、WebP、BMP、GIF等）
- **输出格式**：
  - PNG
  - JPG
  - WebP
  - BMP
  - GIF

### 🎨 命名选项

- **使用原有名称**：保持原图片名称，只替换扩展名
- **自定义格式**：
  - 自定义基础名称
  - 自定义分隔符（如-、+、X等）
  - 序号格式
  - 实时预览命名效果

## 技术栈

- **前端框架**：Vue 3 + Vite
- **构建工具**：Vite
- **样式**：原生CSS
- **图像处理**：Canvas API
- **状态管理**：Vue 3 Composition API

## 安装和运行

### 环境要求

- Node.js 16.0 或更高版本
- npm 7.0 或更高版本

### 安装步骤

1. **克隆仓库**

   ```bash
   git clone https://github.com/DFameMaster/JiaShuiYin.git
   cd JiaShuiYin
   ```

2. **安装依赖**

   ```bash
   npm install
   ```

3. **运行开发服务器**

   ```bash
   npm run dev
   ```

4. **构建生产版本**
   ```bash
   npm run build
   ```

## 使用说明

### 基本操作

1. **上传图片**：点击上传区域或拖动图片到上传区域
2. **配置水印**：在左侧配置面板中调整水印参数
3. **预览效果**：右侧实时显示水印效果
4. **下载图片**：
   - 单张下载：点击预览区域下方的下载按钮
   - 批量下载：选择多张图片，在配置面板底部选择格式和命名方式，点击批量下载

### 批量操作

1. **选择图片**：勾选图片缩略图下方的复选框
2. **全选/取消全选**：点击配置面板底部的全选按钮
3. **选择格式**：在批量操作区域选择下载格式
4. **配置命名**：
   - 选择命名方式（使用原有名称或自定义格式）
   - 若选择自定义格式，输入基础名称、分隔符和序号格式
   - 查看实时预览效果
5. **批量下载**：点击批量下载按钮

## 项目结构

```
├── src/
│   ├── assets/          # 静态资源
│   ├── components/      # 组件
│   │   ├── ConfigPanel.vue     # 水印配置面板
│   │   ├── ImageSelector.vue   # 图片选择器
│   │   ├── ShowImage.vue       # 图片预览和下载
│   │   ├── ShuiYin.vue         # 主组件
│   │   └── UploadImage.vue     # 图片上传组件
│   ├── config/          # 配置文件
│   │   └── defaultConfig.js    # 默认配置
│   ├── App.vue          # 应用入口
│   ├── main.js          # 主文件
│   └── style.css        # 全局样式
├── public/              # 公共资源
├── index.html           # HTML模板
├── package.json         # 项目配置
├── vite.config.js       # Vite配置
└── README.md            # 项目说明
```

## 开发配置

### 自定义开发端口

编辑 `vite.config.js` 文件，修改 `port` 值：

```javascript
server: {
  port: 3000; // 自定义开发端口
}
```

## 浏览器兼容性

- Chrome 60+
- Firefox 55+
- Safari 12+
- Edge 79+

## 许可证

MIT License

## 贡献

欢迎提交 Issue 和 Pull Request 来改进这个项目！

## 联系

- 项目地址：[https://github.com/DFameMaster/JiaShuiYin](https://github.com/DFameMaster/JiaShuiYin)

---

🌟 如果你觉得这个项目有用，请给它点个星！
