<template>
  <div class="app">
    <!-- 顶部标题 -->
    <div class="page-header">
      <h1>Eye Care Timer</h1>
      <p>20-20-20 护眼法则守护者</p>
    </div>

    <el-row :gutter="24" class="main-row">
      <!-- 左侧：计时器卡片 -->
      <el-col :xs="24" :sm="24" :md="14" :lg="16">
        <el-card class="card timer-card" shadow="hover">
          <div class="header">
            <el-icon size="28"><View /></el-icon>
            <div>
              <h2>{{ mode === 'work' ? '专注时间' : '休息时间' }}</h2>
              <p class="sub">{{ mode === 'work' ? '遵循 20-20-20 护眼法则' : '放松眼睛，眺望远方' }}</p>
            </div>
          </div>

          <!-- 模式标签 -->
          <div style="text-align: center; margin: 16px 0;">
            <el-tag :type="mode === 'work' ? 'primary' : 'success'" effect="plain" size="large">
              {{ mode === 'work' ? '🔥 工作模式' : '😌 休息模式' }}
            </el-tag>
          </div>

          <!-- 圆形进度条 -->
          <div class="progress-ring-container">
            <el-progress
                type="circle"
                :percentage="progress"
                :width="220"
                :stroke-width="10"
                :color="mode === 'work' ? '#409EFF' : '#67C23A'"
            >
              <template #default>
                <div class="time">{{ formatTime(currentSeconds) }}</div>
              </template>
            </el-progress>
          </div>

          <div class="actions">
            <el-button
                type="primary"
                size="large"
                @click="toggle"
            >
              {{ isRunning ? '暂停' : '开始' }}
            </el-button>
            <el-button size="large" @click="reset">重置</el-button>
            <el-button
                v-if="alarmPlaying"
                type="danger"
                size="large"
                @click="stopAlarm"
            >
              停止提醒
            </el-button>
          </div>

          <!-- 统计信息 -->
          <el-divider />
          <el-row :gutter="16" class="stats-row">
            <el-col :span="8">
              <div class="stat-item">
                <el-icon style="color: #67C23A;" :size="20"><Select /></el-icon>
                <div>
                  <div class="stat-value">{{ stats.completed }}</div>
                  <div class="stat-label">完成次数</div>
                </div>
              </div>
            </el-col>
            <el-col :span="8">
              <div class="stat-item">
                <el-icon style="color: #409EFF;" :size="20"><Timer /></el-icon>
                <div>
                  <div class="stat-value">{{ stats.totalMinutes }}</div>
                  <div class="stat-label">专注分钟</div>
                </div>
              </div>
            </el-col>
            <el-col :span="8">
              <div class="stat-item">
                <el-icon style="color: #E6A23C;" :size="20"><Coffee /></el-icon>
                <div>
                  <div class="stat-value">{{ stats.restCount }}</div>
                  <div class="stat-label">休息次数</div>
                </div>
              </div>
            </el-col>
          </el-row>

          <!-- GitHub 链接 -->
          <div class="github-section">
            <a href="https://github.com/preciouswxe/eye-care" target="_blank" class="github-link" title="View on GitHub">
              <svg height="24" width="24" viewBox="0 0 16 16" fill="currentColor">
                <path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/>
              </svg>
            </a>
          </div>
        </el-card>
      </el-col>

      <!-- 右侧：设置卡片 -->
      <el-col :xs="24" :sm="24" :md="10" :lg="8">
        <el-card class="card settings-card" shadow="hover">
          <template #header>
            <div class="settings-header">
              <el-icon><Setting /></el-icon>
              <span>设置</span>
            </div>
          </template>
          <div class="settings">
            <el-form label-position="top" size="default">
              <el-form-item label="专注时长（分钟）">
                <el-input-number v-model="workMinutes" :min="1" :max="60" :step="1" style="width: 100%;" />
              </el-form-item>

              <el-form-item label="休息时长（秒）">
                <el-input-number v-model="restSeconds" :min="5" :max="300" :step="5" style="width: 100%;" />
              </el-form-item>

              <el-form-item label="自动循环提醒">
                <el-switch v-model="autoRepeat" active-text="开启" inactive-text="关闭" />
              </el-form-item>

              <el-form-item label="桌面通知">
                <el-switch v-model="enableNotifications" active-text="开启" inactive-text="关闭" @change="handleNotificationToggle" />
              </el-form-item>

              <el-form-item label="提示音量">
                <el-slider v-model="volume" :min="0" :max="100" :show-tooltip="true" :format-tooltip="(val) => val + '%'" @input="updateVolume" />
              </el-form-item>

              <el-form-item>
                <el-button type="primary" @click="applySettings" style="width: 100%;">应用设置</el-button>
              </el-form-item>
            </el-form>

            <!-- 护眼提示 -->
            <el-alert
                title="💡 20-20-20 护眼法则"
                type="info"
                :closable="false"
                description="每使用电子设备 20 分钟，看向 20 英尺（约6米）外的地方，持续 20 秒以上。"
            />
          </div>
        </el-card>
      </el-col>
    </el-row>

    <!-- 全屏休息提醒 -->
    <el-dialog
        v-model="showRestModal"
        fullscreen
        :show-close="false"
        :close-on-click-modal="false"
        :close-on-press-escape="false"
        class="rest-dialog"
    >
      <div class="rest-content">
        <h2>休息时间到！</h2>
        <p>请看向远处，让眼睛放松</p>
        <p class="rest-count">{{ formatTime(restSecondsLeft) }}</p>

        <div class="rest-buttons">
          <el-button
              type="primary"
              size="large"
              @click="startRestCountdown"
              v-if="!restStarted"
          >
            开始休息
          </el-button>
          <el-button
              size="large"
              @click="skipRest"
          >
            跳过休息
          </el-button>
        </div>

        <!-- 护眼小贴士 -->
        <el-card class="tips-card" shadow="never">
          <template #header>
            <div style="display: flex; align-items: center; gap: 8px;">
              <el-icon><InfoFilled /></el-icon>
              <span>休息时可以做这些</span>
            </div>
          </template>
          <ul class="tips-list">
            <li>👁️ 看向窗外的远处景物</li>
            <li>😌 闭眼深呼吸，放松肩颈</li>
            <li>💧 眨眼 10-15 次，湿润眼球</li>
            <li>🔄 转动眼球，活动眼部肌肉</li>
            <li>🪑 调整坐姿，站起来走动</li>
            <li>💦 喝口水，补充水分</li>
          </ul>
        </el-card>
      </div>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, watch } from 'vue'
import { View, Setting, Timer, Select, Coffee, InfoFilled } from '@element-plus/icons-vue'
import { ElMessage, ElNotification } from 'element-plus'

// ========== 配置参数 ==========
const workMinutes = ref(20)
const restSeconds = ref(20)
const autoRepeat = ref(true)
const enableNotifications = ref(true)
const volume = ref(70)

// ========== 状态管理 ==========
const secondsLeft = ref(workMinutes.value * 60)
const restSecondsLeft = ref(restSeconds.value)
const isRunning = ref(false)
const mode = ref('work') // work | rest
const showRestModal = ref(false)
const restStarted = ref(false)
const alarmPlaying = ref(false)

// ========== 统计数据 ==========
const stats = ref({
  completed: 0,
  totalMinutes: 0,
  restCount: 0,
  lastResetDate: new Date().toDateString()
})

// ========== 定时器和音频 ==========
let timer = null
let alarmAudio = null
let alarmInterval = null

// ========== 计算属性 ==========
const currentSeconds = computed(() =>
    mode.value === 'work' ? secondsLeft.value : restSecondsLeft.value
)

const progress = computed(() => {
  const total = mode.value === 'work' ? workMinutes.value * 60 : restSeconds.value
  const current = currentSeconds.value
  return Math.max(0, Math.min(100, ((total - current) / total) * 100))
})

// ========== 工具函数 ==========
function formatTime(sec) {
  const m = Math.floor(sec / 60)
  const s = sec % 60
  return `${String(m).padStart(2, '0')}:${String(s).padStart(2, '0')}`
}

// ========== 核心功能 ==========
function toggle() {
  requestPermission()
  if (isRunning.value) {
    pause()
  } else {
    start()
  }
}

function start() {
  isRunning.value = true
  const endTime = Date.now() + secondsLeft.value * 1000

  timer = setInterval(() => {
    const diff = Math.max(0, Math.round((endTime - Date.now()) / 1000))
    secondsLeft.value = diff

    if (secondsLeft.value <= 0) {
      onWorkComplete()
    }
  }, 100)
}

function pause() {
  isRunning.value = false
  if (timer) {
    clearInterval(timer)
    timer = null
  }
}

function reset() {
  pause()
  stopAlarm()
  mode.value = 'work'
  secondsLeft.value = workMinutes.value * 60
  restSecondsLeft.value = restSeconds.value
  showRestModal.value = false
  restStarted.value = false
}

function onWorkComplete() {
  pause()

  // 更新统计
  stats.value.completed++
  stats.value.totalMinutes += workMinutes.value

  // 显示休息提醒
  showRestModal.value = true
  restStarted.value = false
  startAlarm()
  notify('专注时间完成！', '该休息眼睛了，看向远处 20 秒吧 👀')
}

function startRestCountdown() {
  stopAlarm()
  restStarted.value = true
  restSecondsLeft.value = restSeconds.value
  mode.value = 'rest'

  const endTime = Date.now() + restSecondsLeft.value * 1000

  timer = setInterval(() => {
    const diff = Math.max(0, Math.round((endTime - Date.now()) / 1000))
    restSecondsLeft.value = diff

    if (restSecondsLeft.value <= 0) {
      onRestComplete()
    }
  }, 100)
}

function skipRest() {
  stopAlarm()
  showRestModal.value = false
  restStarted.value = false
  mode.value = 'work'
  secondsLeft.value = workMinutes.value * 60

  ElMessage.info('已跳过休息，记得保护眼睛哦！')

  if (autoRepeat.value) {
    setTimeout(start, 500)
  }
}

function onRestComplete() {
  if (timer) {
    clearInterval(timer)
    timer = null
  }

  stats.value.restCount++

  showRestModal.value = false
  restStarted.value = false
  notify('休息完成！', '继续保持专注吧 💪')
  playBeep()

  mode.value = 'work'
  secondsLeft.value = workMinutes.value * 60

  if (autoRepeat.value) {
    setTimeout(start, 1000)
  }
}

function applySettings() {
  reset()
  saveSettings()

  ElMessage.success({
    message: `设置已保存：专注 ${workMinutes.value} 分钟，休息 ${restSeconds.value} 秒`,
    duration: 3000
  })
}

// ========== 音频控制 ==========
function startAlarm() {
  if (!alarmAudio) return

  alarmPlaying.value = true

  const playSound = () => {
    alarmAudio.currentTime = 0
    alarmAudio.play().catch(() => console.log('需要用户交互才能播放'))
  }

  playSound()
  alarmInterval = setInterval(playSound, 2000)

  // 30秒后自动停止
  setTimeout(() => {
    if (alarmInterval) stopAlarm()
  }, 30000)
}

function stopAlarm() {
  alarmPlaying.value = false

  if (alarmInterval) {
    clearInterval(alarmInterval)
    alarmInterval = null
  }

  if (alarmAudio) {
    alarmAudio.pause()
    alarmAudio.currentTime = 0
  }
}

function playBeep() {
  if (alarmAudio) {
    alarmAudio.currentTime = 0
    alarmAudio.play().catch(() => console.log('播放失败'))
  }
}

function updateVolume(val) {
  if (alarmAudio) {
    alarmAudio.volume = val / 100
  }
}

// ========== 通知功能 ==========
function requestPermission() {
  if (enableNotifications.value && 'Notification' in window && Notification.permission === 'default') {
    Notification.requestPermission()
  }
}

function handleNotificationToggle(enabled) {
  if (enabled && 'Notification' in window && Notification.permission === 'default') {
    Notification.requestPermission().then(permission => {
      if (permission !== 'granted') {
        enableNotifications.value = false
        ElMessage.warning('通知权限未授予')
      }
    })
  }
}

function notify(title, msg) {
  // 桌面通知
  if (enableNotifications.value && 'Notification' in window && Notification.permission === 'granted') {
    new Notification(title, {
      body: msg,
      icon: 'data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><text y=".9em" font-size="90">👁️</text></svg>',
      requireInteraction: true
    })
  }

  // 应用内通知
  ElNotification({
    title,
    message: msg,
    type: 'success',
    duration: 4000
  })
}

// ========== 数据持久化 ==========
function loadSettings() {
  try {
    const saved = localStorage.getItem('eyeCareSettings')
    if (saved) {
      const parsed = JSON.parse(saved)
      workMinutes.value = parsed.workMinutes || 20
      restSeconds.value = parsed.restSeconds || 20
      autoRepeat.value = parsed.autoRepeat !== undefined ? parsed.autoRepeat : true
      enableNotifications.value = parsed.notifications !== undefined ? parsed.notifications : true
      volume.value = parsed.volume !== undefined ? parsed.volume : 70
      secondsLeft.value = workMinutes.value * 60
    }
  } catch (error) {
    console.error('加载设置失败:', error)
  }
}

function saveSettings() {
  try {
    const settings = {
      workMinutes: workMinutes.value,
      restSeconds: restSeconds.value,
      autoRepeat: autoRepeat.value,
      notifications: enableNotifications.value,
      volume: volume.value
    }
    localStorage.setItem('eyeCareSettings', JSON.stringify(settings))
  } catch (error) {
    console.error('保存设置失败:', error)
  }
}

// ========== 生命周期 ==========
onMounted(() => {
  loadSettings()

  // 初始化音频
  alarmAudio = new Audio('https://actions.google.com/sounds/v1/alarms/beep_short.ogg')
  alarmAudio.volume = volume.value / 100

  requestPermission()
})

onUnmounted(() => {
  pause()
  stopAlarm()
})

// 监听设置变化自动保存
watch([workMinutes, restSeconds, autoRepeat, enableNotifications, volume], () => {
  saveSettings()
}, { deep: true })
</script>

<style scoped>
.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  padding: 20px;
}

.page-header {
  text-align: center;
  margin: 30px;
}

.page-header h1 {
  font-size: 32px;
  font-weight: 700;
  margin: 0 0 8px 0;
  background: linear-gradient(135deg, #409EFF, #ccf7ff);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.page-header p {
  font-size: 14px;
  color: var(--el-text-color-secondary);
  margin: 0;
}

.main-row {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
}

.card {
  width: 100%;
  border-radius: 16px;
  height: 100%;
}

.timer-card {
  padding: 8px;
}

.header {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 12px;
}

.header h2 {
  margin: 0;
  font-size: 20px;
  font-weight: 600;
}

.sub {
  font-size: 12px;
  color: var(--el-text-color-secondary);
  margin: 4px 0 0;
}

.progress-ring-container {
  display: flex;
  justify-content: center;
  margin: 20px 0;
}

.time {
  font-size: 30px;
  font-weight: 700;
  letter-spacing: 2px;
  background: linear-gradient(135deg, #409EFF, #8ee1f3);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.actions {
  display: flex;
  justify-content: center;
  gap: 12px;
  margin: 16px 0;
  flex-wrap: wrap;
}

.stats-row {
  margin-top: 50px;
}

.stat-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px;
  background: var(--el-fill-color-light);
  border-radius: 10px;
  transition: all 0.3s;
}

.stat-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
}

.stat-value {
  font-size: 18px;
  font-weight: 700;
  color: var(--el-text-color-primary);
  line-height: 1;
}

.stat-label {
  font-size: 11px;
  color: var(--el-text-color-secondary);
  margin-top: 4px;
}

.settings-card {
  padding: 8px;
}

.settings-header {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 18px;
  font-weight: 600;
}

.settings {
  padding: 4px;
}

.github-section {
  margin-top: 20px;
  text-align: center;
}

.github-link {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 36px;
  height: 36px;
  color: #586069;
  text-decoration: none;
  transition: color 0.2s ease;
}

.github-link:hover {
  color: #24292e;
}

.rest-dialog :deep(.el-dialog) {
  width: 100%;
  height: 100%;
  margin: 0;
}

/* 全屏休息对话框 */
.rest-dialog :deep(.el-dialog__body) {
  padding: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}

.rest-content {
  text-align: center;
  color: white;
  padding: 40px 20px;
  max-width: 600px;
  width: 100%;
  margin: auto;
}

.rest-icon {
  font-size: 100px;
  margin-bottom: 20px;
  animation: bounce 2s ease-in-out infinite;
}

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-15px); }
}

.rest-content h2 {
  font-size: 40px;
  margin-bottom: 12px;
}

.rest-content > p {
  font-size: 18px;
  margin-bottom: 16px;
  opacity: 0.9;
}

.rest-count {
  font-size: 72px !important;
  font-weight: 700;
  color: #67C23A;
  margin: 24px 0 !important;
  text-shadow: 0 0 30px rgba(103, 194, 58, 0.5);
  animation: pulse 1.5s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.05); }
}

.rest-buttons {
  display: flex;
  gap: 12px;
  justify-content: center;
  margin: 32px 0;
  flex-wrap: wrap;
}

.tips-card {
  margin-top: 32px;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
}

.tips-list {
  list-style: none;
  padding: 0;
  margin: 0;
  text-align: left;
}

.tips-list li {
  padding: 10px 0;
  font-size: 15px;
  line-height: 1.6;
  border-bottom: 1px solid var(--el-border-color-lighter);
  color: var(--el-text-color-primary);
}

.tips-list li:last-child {
  border-bottom: none;
}

/* 响应式布局 */
@media (max-width: 992px) {
  .main-row {
    flex-direction: column;
  }
}

@media (max-width: 768px) {
  .app {
    padding: 16px;
  }

  .page-header h1 {
    font-size: 24px;
  }

  .page-header p {
    font-size: 12px;
  }

  .rest-icon {
    font-size: 70px;
  }

  .rest-content h2 {
    font-size: 28px;
  }

  .rest-count {
    font-size: 56px !important;
  }
}
</style>