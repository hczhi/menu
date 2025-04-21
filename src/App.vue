<script setup>
import { ref, reactive, onMounted } from 'vue'
import { ElMessage } from 'element-plus'
import { Remove, ArrowUp, Crop, Download } from '@element-plus/icons-vue'
import axios from 'axios'

const fileList = ref([])
const previewUrl = ref('')
const loading = ref(false)
const menuResults = ref(null)
const showAnimation = ref(false)
const activeIndex = ref(0) // 当前激活的菜品索引
const dishCards = ref([]) // 菜品卡片的引用

const uploadHeaders = reactive({
  'Content-Type': 'multipart/form-data'
})

// 模拟API地址，实际使用时需要替换为真实的后端API地址
const apiUrl = 'https://api.example.com/menu-recognition'

const handleUploadSuccess = (file) => {
  // 当文件改变时，更新fileList并设置预览URL
  fileList.value = [file]
  if (file.raw) {
    previewUrl.value = URL.createObjectURL(file.raw)
  }
}

const handleExceed = () => {
  ElMessage.warning('只能上传一张菜单图片')
}

const beforeUpload = (file) => {
  const isImage = file.type.startsWith('image/')
  if (!isImage) {
    ElMessage.error('只能上传图片文件!')
    return false
  }
  return true
}

// 不再需要单独的预览处理函数

const handleRemove = () => {
  menuResults.value = null
  previewUrl.value = ''
  fileList.value = []
}

const submitToBackend = async () => {
  if (fileList.value.length === 0) {
    ElMessage.warning('请先上传菜单图片')
    return
  }

  loading.value = true
  try {
    const formData = new FormData()
    formData.append('menuImage', fileList.value[0].raw)

    // 实际项目中应该使用真实的API调用
    // const response = await axios.post(apiUrl, formData, { headers: uploadHeaders })
    // menuResults.value = response.data

    // 模拟API调用
    setTimeout(() => {
      menuResults.value = [
        {
          chineseName: '宫保鸡丁',
          englishName: 'Kung Pao Chicken',
          ingredients: '鸡胸肉、花生、干辣椒、葱姜蒜',
          cookingMethod: '先将鸡肉切丁腌制，爆香花生和干辣椒先将鸡肉切丁腌制，爆香花生和干辣椒先将鸡肉切丁腌制，爆香花生和干辣椒先将鸡肉切丁腌制，爆香花生和干辣椒，加入鸡肉翻炒，最后加入调味料收汁'
        },
        {
          chineseName: '鱼香肉丝',
          englishName: 'Yu-Shiang Shredded Pork',
          ingredients: '猪里脊肉、木耳、胡萝卜、竹笋、葱姜蒜',
          cookingMethod: '将肉丝腌制后炒制，加入配菜翻炒，最后加入鱼香汁收汁'
        },
        {
          chineseName: '水煮鱼',
          englishName: 'Sichuan Boiled Fish',
          ingredients: '草鱼、豆芽、辣椒、花椒、葱姜蒜',
          cookingMethod: '鱼片焯水后与配菜一起放入锅中，浇上热油激发香味，最后加入调味料'
        }
      ]
      loading.value = false
      // 显示动画效果
      showAnimation.value = true
    }, 2000)
  } catch (error) {
    console.error('上传失败:', error)
    ElMessage.error('菜单识别失败，请重试')
    loading.value = false
  }
}
// 添加滚动到指定菜品的方法
const scrollToDish = (index) => {
  activeIndex.value = index
  const element = document.getElementById(`dish-${index}`)
  if (element) {
    element.scrollIntoView({ behavior: 'smooth', block: 'start' })
  }
}

// 添加动画效果
onMounted(() => {
  // 初始化装饰元素动画
  animateDecorations();
});

const animateDecorations = () => {
  const shapes = document.querySelectorAll('.shape');
  shapes.forEach((shape, index) => {
    // 为每个形状添加不同的动画
    const delay = index * 0.2;
    shape.style.animation = `float 8s ease-in-out ${delay}s infinite`;
  });
};

</script>

<template>
  <div class="container">


    <header>
      <div class="logo-container">
        <div class="logo">Menu<span>AI</span></div>
      </div>
      <div class="header-content">
        <h1>中文菜单识别翻译系统</h1>
        <p class="subtitle">上传中文菜单图片，AI将识别菜品并提供英文翻译、原材料和制作方法</p>
      </div>
    </header>

    <main>
      <div class="main-content">
        <div class="upload-section-wrapper">
          <el-card class="upload-section">
            <div class="upload-container" v-if="previewUrl.length === 0">
              <el-upload class="menu-uploader" :action="apiUrl" :auto-upload="false" :show-file-list="false"
                :on-change="handleUploadSuccess" :on-remove="handleRemove" :before-upload="beforeUpload"
                :on-exceed="handleExceed" :limit="1" :headers="uploadHeaders" :file-list="fileList" list-type="picture">
                <div class="upload-area">
                  <el-icon class="upload-icon"><el-icon-upload /></el-icon>
                  <h3 class="upload-title">上传菜单图片</h3>
                  <p class="upload-desc">点击或拖拽文件到此区域</p>
                  <el-button type="primary" class="upload-btn">选择图片</el-button>
                </div>
                <template #tip>
                  <div class="el-upload__tip">
                    <el-icon class="tip-icon"><el-icon-info-filled /></el-icon>
                    <span>请上传清晰的菜单图片，支持jpg/png格式</span>
                  </div>
                </template>
              </el-upload>
            </div>

            <div v-if="previewUrl" class="preview-container">
              <img :src="previewUrl" class="preview-image" alt="菜单预览" />
              <div class="preview-actions">
                <el-button type="danger" class="delete-btn" @click="handleRemove">
                  <el-icon>
                    <Remove />
                  </el-icon>
                  删除图片
                </el-button>
              </div>
            </div>
            <div v-if="previewUrl" class="action-buttons">
              <el-button type="primary" class="submit-btn" @click="submitToBackend" :loading="loading">
                识别菜单
              </el-button>
            </div>
          </el-card>
        </div>



        <!-- 添加使用教程模块 -->
        <div class="tutorial-section" v-if="!menuResults">
          <h2 class="tutorial-title">如何裁剪照片</h2>

          <div class="steps-container">
            <div class="step-item">
              <div class="step-icon">
                <el-icon>
                  <ArrowUp />
                </el-icon>
              </div>
              <div class="step-number">步骤1</div>
              <div class="step-content">
                <p>上传照片，您需要对其进行编辑并完成上传。</p>
              </div>
            </div>

            <div class="step-item">
              <div class="step-icon">
                <el-icon>
                  <Crop />
                </el-icon>
              </div>
              <div class="step-number">第2步</div>
              <div class="step-content">
                <p>在照片中定义要裁剪的区域，以像素为单位指定其尺寸。</p>
              </div>
            </div>

            <div class="step-item">
              <div class="step-icon">
                <el-icon>
                  <Download />
                </el-icon>
              </div>
              <div class="step-number">第三步</div>
              <div class="step-content">
                <p>裁剪后，您可以轻松地将新的照片文件下载到设备上。</p>
              </div>
            </div>
          </div>

          <div class="tutorial-info">
            <h3 class="info-title">为什么您可能需要<span>裁剪</span>一张照片</h3>
            <div class="info-content">

              <p>
                在此页面上，您可以免费在线裁剪照片。无需注册，只需将照片加载到裁剪工具中，然后下载。您只需一个特定的裁剪照片的功能，您可以使用我们的服务来切除照片以外的所有内容。例如边缘边框中的内容，或者当照片中的对象位于边缘时，需要裁剪照片。因此，在线照片编辑器将使您摆脱图像中不必要的内容，只保留照片的有意义部分。
              </p>
            </div>

          </div>
        </div>


        <div v-if="menuResults" class="results-section-wrapper">
          <!-- 添加左侧导航 -->
          <div class="dish-nav" v-if="menuResults.length > 100">

            <div class="dish-nav-items">
              <div v-for="(dish, index) in menuResults" :key="index" class="dish-nav-item"
                :class="{ 'active': activeIndex === index }" @click="scrollToDish(index)">
                <div class="dish-nav-name">{{ dish.chineseName }}
                  <div class="english-name">{{ dish.englishName }}</div>
                </div>
              </div>
            </div>
          </div>
          <div class="results-section" :class="{ 'showAnimation': showAnimation }">
            <!-- <h2 class="section-title">识别结果</h2> -->
            <div class="dish-cards">
              <div v-for="(dish, index) in menuResults" :key="index" class="dish-card"
                :style="{ animationDelay: showAnimation ? `${index * 0.15}s` : '0s' }">
                <div class="leftbox"><img src="/mbg.png"/></div>
                <div class="rightbox">
                  <div class="dish-header">
                  <h3>{{ dish.chineseName }} <span class="english-name">({{ dish.englishName }})</span></h3>
                  <div class="price">49.9</div>
                </div>
                <div class="dish-content">
                  <div class="dish-section dish-ingredients">
                    <p>{{ dish.ingredients }}</p>
                  </div>
                  <div class="dish-section">
                    <p>{{ dish.cookingMethod }}</p>
                  </div>
                </div>
                </div>
                
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>
  <!-- 添加装饰元素 -->
  <div class="decoration-elements">
    <div class="shape-1">
      <!-- <div class="shape-1-light"></div>
        <div class="shape-2-light"></div> -->
    </div>
    <div class="shape-2"></div>
    <!-- <div class="shape shape-3"></div>
      <div class="shape shape-4"></div>
      <div class="shape shape-5"></div> -->
  </div>
  <!-- 添加页脚装饰元素 -->
  <div class="footer-decoration">
    <div class="footer-wave"></div>
    <div class="footer-content">
      <div class="footer-dots">
        <div class="footer-dot dot-1"></div>
        <div class="footer-dot dot-2"></div>
        <div class="footer-dot dot-3"></div>
      </div>
      <div class="copyright">© 2023 MenuAI - 中文菜单识别翻译系统</div>
    </div>
  </div>
</template>

<style scoped lang="scss">
.english-name {
  color: var(--accent-color);
  font-weight: 500;
  font-size: 0.9em;
  opacity: 0.9;
  margin-left: 5px;
}

html,
body {
  height: 100%;
  margin: 0;
  padding: 0;
}

body {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

:root {
  --primary-color: #FF6B35;
  --primary-light: #FFEFE8;
  --accent-color: #2EC4B6;
  --accent-light: #E6F7F5;
  --text-primary: #2D3142;
  --text-secondary: #5C6378;
  --background-color: #FFFFFF;
  --border-color: #EEEEEE;
  --shadow-sm: 0 4px 12px rgba(0, 0, 0, 0.03);
  --shadow-md: 0 8px 24px rgba(0, 0, 0, 0.05);
  --shadow-lg: 0 16px 32px rgba(0, 0, 0, 0.08);
  --radius-sm: 10px;
  --radius-md: 16px;
  --radius-lg: 24px;
}

.container {
  z-index: 10;
  max-width: 1400px;
  width: 70vw;
  margin: 0 auto;
  padding: 2.5rem;
  animation: fadeIn 0.8s ease-out;
  position: relative;
  overflow: hidden;
  flex: 1;
  /* 让容器占据剩余空间 */
}

/* 装饰元素 */
.decoration-elements {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 0;
  overflow-x: hidden;
}

.shape {
  position: absolute;
  border-radius: 50%;
}

.shape-1 {
  width: 100vw;
  height: 50rem;
  background: #f7e8e8;
  position: absolute;
  left: 0;
  top: 0;
  overflow: hidden;
  background: url(/bg3.png) repeat #f7e8e8;
  background-size: 100%;

  &::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(232, 240, 247, 0.6);
    z-index: 1;
  }
}

.shape-2 {
  bottom: 10%;
  left: -80px;
  width: 280px;
  height: 280px;
  background-color: var(--accent-light);
  opacity: 0.7;
  /* 增加不透明度 */
}

.shape-3 {
  top: 40%;
  right: 10%;
  width: 200px;
  height: 200px;
  background-color: var(--primary-light);
  opacity: 0.6;
  /* 增加不透明度 */
  border-radius: 30% 70% 70% 30% / 30% 30% 70% 70%;
}

.shape-4 {
  bottom: 20%;
  right: 20%;
  width: 250px;
  height: 250px;
  background-color: var(--accent-light);
  opacity: 0.5;
  /* 增加不透明度 */
  border-radius: 60% 40% 30% 70% / 60% 30% 70% 40%;
}

.shape-5 {
  top: 60%;
  left: 15%;
  width: 180px;
  height: 180px;
  background-color: var(--primary-light);
  opacity: 0.4;
  border-radius: 30% 70% 50% 50% / 50% 30% 70% 50%;
}

header {
  text-align: left;
  margin-bottom: 3rem;
  position: relative;
  z-index: 1;
  /* display: flex;
  align-items: flex-start;
  gap: 2rem; */
}

.header-content {
  flex: 1;
}

h1 {
  color: var(--primary-color);
  margin-bottom: 1rem;
  font-weight: 800;
  letter-spacing: -0.5px;
  font-size: 2.7rem;
  background: linear-gradient(to right, var(--primary-color), var(--accent-color));
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  display: inline-block;
}

.main-content {
  position: relative;
  z-index: 1;
}

main {
  display: flex;
  flex-direction: column;
  gap: 3rem;
}

.upload-section-wrapper,
.results-section-wrapper {
  position: relative;
}

.upload-section {
  width: 100%;
  overflow: hidden;
  border: none;
  box-shadow: var(--shadow-md);
  border-radius: var(--radius-md);
  background-color: rgba(255, 255, 255, 0.85);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
}

/* .upload-section:hover, .results-section:hover {
  transform: translateY(-5px);
  box-shadow: var(--shadow-lg);
} */

.section-title {
  color: var(--primary-color);
  font-size: 1.8rem;
  margin-bottom: 1.8rem;
  font-weight: 700;
  padding: 2rem 2rem 0;
  position: relative;
  display: inline-block;
}

.section-title::after {
  content: '';
  position: absolute;
  bottom: -8px;
  left: 2rem;
  width: 60px;
  height: 4px;
  background: var(--accent-color);
  border-radius: 2px;
}

.upload-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.5rem;
  padding: 1rem;
}

.menu-uploader {
  width: 95%;
  display: flex;
  justify-content: center;
  padding: 2.5rem;
  margin: 1.5rem;
  background-color: var(--primary-light);
  border-radius: var(--radius-md);
  border: 2px dashed rgb(53 72 255 / 30%);
  transition: all 0.3s cubic-bezier(0.165, 0.84, 0.44, 1);
}

.menu-uploader:hover {
  border-color: var(--primary-color);
  background-color: rgba(53, 110, 255, 0.08);
  transform: scale(1.01);
}

.action-buttons {
  margin-top: 1.5rem;
  display: flex;
  justify-content: center;
  gap: 1rem;
  padding: 0 1.5rem 1.5rem;
}

.submit-btn {
  background: var(--primary-color);
  border: none;
  padding: 0.9em 2.2em;
  font-weight: 600;
  letter-spacing: 0.5px;
  transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  border-radius: 50px;
  font-size: 1.05rem;
  box-shadow: 0 4px 10px rgba(255, 107, 53, 0.2);
}

.submit-btn:hover:not(:disabled) {
  transform: translateY(-3px) scale(1.03);
  box-shadow: 0 8px 20px rgba(255, 107, 53, 0.3);
  background-color: #FF5A20;
}

.submit-btn:disabled {
  background: #E0E0E0;
  opacity: 0.7;
}

.preview-container {
  display: flex;
  justify-content: center;
  /* padding: 1.5rem; */
  animation: fadeIn 0.5s ease;
  position: relative;
  width: 90%;
  margin: 1rem auto;
}

.upload-area {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  justify-content: flex-start;
  padding: 5rem 1rem;
  border-radius: var(--radius-md);
  /* background-color: rgba(255, 125, 77, 0.03); */
  transition: all 0.3s ease;
  width: 100%;
}

.upload-icon {
  font-size: 3rem;
  color: var(--primary-color);
  margin-bottom: 1rem;
}

.upload-title {
  font-size: 1.5rem;
  font-weight: 600;
  color: var(--text-primary);
  margin: 0.5rem 0;
}

.upload-desc {
  color: var(--text-secondary);
  margin-bottom: 1.5rem;
}

.upload-btn {
  background-color: var(--primary-color);
  border: none;
  border-radius: 50px;
  padding: 0.9em 2.2em;
  font-size: 1.05rem;
  font-weight: 600;
  box-shadow: 0 4px 10px rgba(255, 107, 53, 0.2);
  transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

.upload-btn:hover {
  background-color: #FF5A20;
  transform: translateY(-2px) scale(1.03);
  box-shadow: 0 8px 20px rgba(255, 107, 53, 0.3);
}

.el-upload__tip {
  display: flex;
  align-items: center;
  justify-content: flex-start;
  margin-top: 1rem;
  color: var(--text-secondary);
  font-size: 0.9rem;
}

.tip-icon {
  margin-right: 0.5rem;
  color: var(--primary-color);
}

.preview-image {
  width: 100%;
  max-height: 600px;
  object-fit: contain;
  border-radius: var(--radius-md);
  transition: transform 0.4s cubic-bezier(0.165, 0.84, 0.44, 1);
  box-shadow: var(--shadow-sm);
}

.preview-image:hover {
  transform: scale(1.02);
  box-shadow: var(--shadow-md);
}

.preview-actions {
  position: absolute;
  top: 10px;
  right: 10px;
  z-index: 10;
}

.delete-btn {
  background: rgba(220, 53, 69, 0.8);
  border: none;
  color: white;
  padding: 10px 18px;
  border-radius: 50px;
  transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 600;
  box-shadow: 0 4px 8px rgba(220, 53, 69, 0.2);
}

.delete-btn:hover {
  background: rgba(220, 53, 69, 1);
  transform: translateY(-2px) scale(1.03);
  box-shadow: 0 6px 12px rgba(220, 53, 69, 0.3);
}

.results-section {
  padding-bottom: 2rem;
}

.dish-cards {
  display: flex;
  flex-direction: column;
  gap: 2rem;
  padding: 5rem 0rem 1rem;
}

.dish-card {
  transition: all 0.4s cubic-bezier(0.165, 0.84, 0.44, 1);
  overflow: hidden;
  position: relative;
  transform: translateZ(0);
  margin: 0 0.5rem;
  animation: slideInRight 0.5s ease-out both;
  animation-play-state: paused;
  will-change: transform, box-shadow;
  padding-bottom: 2rem;
  display: flex;

  .leftbox {
    width: 100px;
    height: 100px;
    border-radius: 100px;
    overflow: hidden;
    flex-shrink: 0;
    
    img {
      width: 100px;
      height: 100px;
    }
  }

  .rightbox {
    flex: 1;
  }
}

.dish-card:hover {
  transform: translateY(-5px);
}

.showAnimation .dish-card {
  animation-play-state: running;
}

.dish-header {
  text-align: left;
    padding: 0rem 1.8rem 1rem;
    position: relative;
    display: flex;
    align-items: center;
    justify-content: space-between;
  .price{
    color: #FF5A20;
    font-weight: 600;
    font-size: 14px;
  }
}

.dish-header h3 {
  font-size: 1.3rem;
  font-weight: 600;
  margin: 0;
  line-height: 1.4;
  color: var(--text-primary);
}

.dish-content {
  padding: 0rem 0 0 1rem;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.dish-section {
  padding: 0 1rem;
  position: relative;
  text-align: left;
}
.dish-ingredients{
  font-weight: bold;
}

.dish-section h5 {
  margin: 0 0 1rem;
  color: var(--primary-color);
  font-weight: 600;
  font-size: 1.05rem;
  display: flex;
  align-items: center;
  letter-spacing: 0.3px;
}

.dish-section h5::before {
  content: '';
  display: inline-block;
  width: 6px;
  height: 20px;
  background: var(--accent-color);
  margin-right: 0.8rem;
  border-radius: 3px;
  transform: translateY(0px);
}

.dish-section p {
  margin: 0;
  color: var(--text-secondary);
  line-height: 1.7;
  font-size: 1rem;
  padding: 0 0 0 0rem;
  border-left: 2px solid var(--primary-light);
  letter-spacing: 0.2px;
}

/* 响应式调整 */
@media (max-width: 992px) {
  .results-section-wrapper {
    flex-direction: column;
  }

  .dish-nav {
    position: relative;
    top: 0;
    width: 100%;
    margin-bottom: 1rem;
  }

  .dish-nav-items {
    flex-direction: row;
    flex-wrap: wrap;
    padding: 0.5rem 0.5rem;
    gap: 0.8rem;
  }

  .dish-nav-item {
    padding: 0.5rem 0.5rem;
    border-radius: var(--radius-sm);
    background-color: rgba(255, 255, 255, 0.7);
  }

  .dish-nav-item.active::before {
    top: 0.9rem;
  }
}

@media (max-width: 768px) {
  .container {
    padding: 2rem 0rem;
    width: 80vw;
  }

  .el-upload__tip {
    display: none;
  }

  .upload-area {
    text-align: center;
    display: flex;
    align-items: center;
  }

  header {
    margin-bottom: 2rem;
    flex-direction: column;
    gap: 1rem;
  }

  h1 {
    font-size: 2rem;
  }

  .dish-content {
    flex-direction: column;
    gap: 0.8rem;
    padding: 0rem 0 0 0rem;
  }

  .upload-container {
    padding: 0.5rem;
  }

  .menu-uploader {
    padding: 1.5rem 1rem;
  }

  .logo {
    font-size: 1.8rem;
  }

  .subtitle {
    font-size: 0.95rem;
  }

  .dish-header {
    padding: 1rem 0rem;
  }

  .dish-card {
    margin: 0rem;
  }

  .dish-section {
    padding: 0 0rem;
  }

  .dish-nav {
    display: none;
  }

  .footer-wave {
    height: 40px;
    background: none;

    &::after {
      height: 40px;
    }
  }

  .footer-content {
    padding: 0.5rem 0 1.5rem;
    background: none;
  }

  .tutorial-section {
    padding: 1.5rem 1rem;
  }

  .steps-container {
    flex-direction: column;
    gap: 3rem;
  }

  .info-content {
    flex-direction: column;
  }

  .tutorial-image {
    width: 100%;
    margin-bottom: 1.5rem;
  }
}

/* 添加一些动画效果 */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes float {
  0% {
    transform: translateY(0px);
  }

  50% {
    transform: translateY(-15px);
  }

  100% {
    transform: translateY(0px);
  }
}

@keyframes slideInRight {
  from {
    opacity: 0;
    transform: translateX(30px);
  }

  to {
    opacity: 1;
    transform: translateX(0);
  }
}

@keyframes pulse {
  0% {
    transform: scale(1);
  }

  50% {
    transform: scale(1.05);
  }

  100% {
    transform: scale(1);
  }
}

.logo-container {
  margin-bottom: 1rem;
}

.logo {
  font-size: 2.4rem;
  font-weight: 800;
  color: var(--primary-color);
  letter-spacing: -1px;
  position: relative;
  display: inline-block;
  text-shadow: 2px 2px 0px rgba(255, 107, 53, 0.1);
}

.logo span {
  color: var(--accent-color);
  position: relative;
}

.logo::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 4px;
  background: linear-gradient(to right, var(--primary-color), var(--accent-color));
  border-radius: 2px;
  transform: scaleX(0.7) translateY(5px);
  opacity: 0.7;
}

.subtitle {
  color: var(--text-secondary);
  font-size: 1.15rem;
  max-width: 650px;
  margin: 0;
  line-height: 1.6;
  letter-spacing: 0.2px;
}


/* 添加导航样式 */
.results-section-wrapper {
  display: flex;
  gap: 1rem;
  position: relative;
}

.dish-nav {
  position: sticky;
  top: 3rem;
  height: fit-content;
  width: 250px;
  flex-shrink: 0;
  overflow: hidden;
  padding: 5rem 0 0 0rem;
  transition: all 0.3s ease;
}

.dish-nav-title {
  font-weight: 700;
  color: var(--text-primary);
  padding: 0 1.5rem 1rem;
  margin-bottom: 0.5rem;
  border-bottom: 1px solid rgba(240, 240, 240, 0.6);
  font-size: 1.1rem;
}

.dish-nav-items {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  padding: 0.5rem 0;
}

.dish-nav-item {
  display: flex;
  align-items: center;
  gap: 0.8rem;
  padding: 0.8rem 0.5rem 0rem 1.5rem;
  cursor: pointer;
  transition: all 0.2s ease;
  position: relative;
}

.dish-nav-item:hover {
  // background-color: rgba(255, 107, 53, 0.08);
}

.dish-nav-item.active {
  // background-color: rgba(255, 107, 53, 0.12);
}

.dish-nav-item.active::before {
  content: '';
  position: absolute;
  left: 0;
  top: 1.2rem;
  height: 20%;
  width: 4px;
  background-color: var(--primary-color);
  border-radius: 5rem 10rem 10rem 5rem;
}

.dish-nav-index {
  width: 24px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: var(--accent-color);
  color: white;
  border-radius: 50%;
  font-size: 0.8rem;
  font-weight: 600;
}

.dish-nav-name {
  font-size: 0.95rem;
  color: var(--text-primary);
  font-weight: 500;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  text-align: left;

  .english-name {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    padding-top: 0.2rem;
  }
}

.results-section {
  flex: 1;
  min-width: 0;
  /* 确保弹性项可以缩小到小于其内容大小 */
}

/* 响应式调整 */
@media (max-width: 992px) {
  .results-section-wrapper {
    flex-direction: column;
  }

  .dish-nav {
    position: relative;
    top: 0;
    width: 100%;
    margin-bottom: 1rem;
  }

  .dish-nav-items {
    flex-direction: row;
    flex-wrap: wrap;
    padding: 0.5rem 0.5rem;
    gap: 0.8rem;
  }

  .dish-nav-item {
    padding: 0.5rem 0.5rem;
    border-radius: var(--radius-sm);
    background-color: rgba(255, 255, 255, 0.7);
  }

  .dish-nav-item.active::before {
    top: 0.9rem;
  }
}

@media (max-width: 768px) {
  .container {
    padding: 2rem 0rem;
    width: 80vw;
  }

  .el-upload__tip {
    display: none;
  }

  .upload-area {
    text-align: center;
    display: flex;
    align-items: center;
  }

  header {
    margin-bottom: 2rem;
    flex-direction: column;
    gap: 1rem;
  }

  h1 {
    font-size: 2rem;
  }

  .dish-content {
    flex-direction: column;
    gap: 0.8rem;
    padding: 0rem 0 0 0rem;
  }

  .upload-container {
    padding: 0.5rem;
  }

  .menu-uploader {
    padding: 1.5rem 1rem;
  }

  .logo {
    font-size: 1.8rem;
  }

  .subtitle {
    font-size: 0.95rem;
  }

  .dish-header {
    padding: 1rem 0rem;
  }

  .dish-card {
    margin: 0rem;
  }

  .dish-section {
    padding: 0 0rem;
  }

  .dish-nav {
    display: none;
  }

  .footer-wave {
    height: 40px;
    background: none;

    &::after {
      height: 40px;
    }
  }

  .footer-content {
    padding: 0.5rem 0 1.5rem;
    background: none;
  }

  .tutorial-section {
    padding: 1.5rem 1rem;
  }

  .steps-container {
    flex-direction: column;
    gap: 3rem;
  }

  .info-content {
    flex-direction: column;
  }

  .tutorial-image {
    width: 100%;
    margin-bottom: 1.5rem;
  }
}

/* 页脚装饰元素样式 */
.footer-decoration {
  position: relative;
  /* 改为相对定位 */
  width: 100%;
  overflow: hidden;
  z-index: 1;
  margin-top: auto;
  /* 自动调整上边距 */
}

.footer-wave {
  position: relative;
  height: 70px;
  background: linear-gradient(to right, #f7e8e8, #f0f8ff);

  &::before {
    content: "";
    position: absolute;
    left: 0;
    right: 0;
    bottom: 0;
    height: 10px;
    background: linear-gradient(45deg, var(--primary-light) 45%, var(--accent-light) 55%);
    opacity: 0.6;
  }

  &::after {
    content: "";
    position: absolute;
    left: 0;
    right: 0;
    bottom: 0;
    height: 70px;
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 1200 120' preserveAspectRatio='none'%3E%3Cpath d='M321.39,56.44c58-10.79,114.16-30.13,172-41.86,82.39-16.72,168.19-17.73,250.45-.39C823.78,31,906.67,72,985.66,92.83c70.05,18.48,146.53,26.09,214.34,3V0H0V27.35A600.21,600.21,0,0,0,321.39,56.44Z' fill='%23ffffff' opacity='1'/%3E%3C/svg%3E");
    background-size: cover;
  }
}

.footer-content {
  background: linear-gradient(to right, #f7e8e8, #f0f8ff);
  padding: 1rem 0 2rem;
  text-align: center;
  position: relative;
}

.copyright {
  color: var(--text-secondary);
  font-size: 0.9rem;
  opacity: 0.8;
  letter-spacing: 0.5px;
}

/* 添加使用教程模块的样式 */
.tutorial-section {
  margin: 4rem 0;
  padding: 2rem;
}

.tutorial-title {
  text-align: center;
  font-size: 2rem;
  font-weight: 700;
  color: var(--text-primary);
  margin-bottom: 3rem;
}

.steps-container {
  display: flex;
  justify-content: space-between;
  gap: 2rem;
  margin-bottom: 4rem;
}

.step-item {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
}

.step-icon {
  width: 80px;
  height: 80px;
  background-color: var(--primary-light);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 1rem;
  font-size: 2rem;
  color: var(--primary-color);
}

.step-number {
  font-weight: 700;
  font-size: 1.2rem;
  color: var(--primary-color);
  margin-bottom: 0.5rem;
}

.step-content {
  color: var(--text-secondary);
  line-height: 1.6;
}

.tutorial-info {
  // background-color: #f9f9f9;
  border-radius: var(--radius-md);
  padding: 2rem;
  position: relative;
}

.info-title {
  font-size: 1.5rem;
  font-weight: 600;
  color: var(--text-primary);
  margin-bottom: 1.5rem;
}

.info-title span {
  color: var(--primary-color);
}

.info-content {
  display: flex;
  gap: 2rem;
  align-items: flex-start;
}

.tutorial-image {
  width: 300px;
  border-radius: var(--radius-sm);
  box-shadow: var(--shadow-sm);
}

.info-content p {
  flex: 1;
  color: var(--text-secondary);
  line-height: 1.8;
}

.tutorial-btn {
  margin-top: 1.5rem;
  padding: 0.8em 2em;
  font-weight: 600;
}

/* 响应式调整 */
@media (max-width: 768px) {
  .tutorial-section {
    padding: 1.5rem 1rem;
  }

  .steps-container {
    flex-direction: column;
    gap: 3rem;
  }

  .info-content {
    flex-direction: column;
  }

  .tutorial-image {
    width: 100%;
    margin-bottom: 1.5rem;
  }

  .shape-1 {
    background: url(/mbg.png) repeat #f7e8e8;
    background-size: 100%;
  }
}
</style>
