<template>
  <div class="module">
    <div class="module-head">
      <h2>🩺 个人健康自测</h2>
      <p>通过语音、文字、图片与实时人脸多维采集，初步评估您的健康状态。</p>
    </div>

    <div class="tabs">
      <button
        v-for="t in tabs"
        :key="t.key"
        class="tab"
        :class="{ active: active === t.key }"
        @click="active = t.key"
      >
        {{ t.label }}
      </button>
    </div>

    <div class="panel">
      <!-- 语音 -->
      <section v-show="active === 'voice'" class="block">
        <h3>语音自述</h3>
        <p class="hint">点击下方按钮，用自然语言描述您近期的身体感受（需浏览器支持语音识别）。</p>
        <div class="voice-row">
          <button class="btn primary" :class="{ recording: recording }" @click="toggleVoice">
            {{ recording ? '⏹ 停止录音' : '🎤 开始录音' }}
          </button>
          <span v-if="recording" class="pulse-dot"></span>
        </div>
        <div v-if="!supportsSpeech" class="warn">
          当前浏览器不支持语音识别，您可直接在下方文本框中输入。
        </div>
        <textarea
          v-model="voiceText"
          class="text-input"
          rows="4"
          placeholder="语音转写内容或出现识别结果后将显示在这里…"
        ></textarea>
        <button class="btn" :disabled="!voiceText" @click="analyzeText(voiceText, 'voice')">
          分析语音自述
        </button>
      </section>

      <!-- 文字 -->
      <section v-show="active === 'text'" class="block">
        <h3>文字描述</h3>
        <p class="hint">描述您的症状、作息或情绪，例如「最近总是失眠、容易疲劳」。</p>
        <textarea
          v-model="textInput"
          class="text-input"
          rows="5"
          placeholder="请输入您的健康困扰…"
        ></textarea>
        <button class="btn primary" :disabled="!textInput" @click="analyzeText(textInput, 'text')">
          生成健康建议
        </button>
      </section>

      <!-- 图片 -->
      <section v-show="active === 'image'" class="block">
        <h3>图片上传</h3>
        <p class="hint">上传舌象、皮肤或饮食照片，系统将给出参考性分析（演示为模拟识别）。</p>
        <label class="uploader">
          <input type="file" accept="image/*" @change="onImage" />
          <span>📷 选择图片</span>
        </label>
        <div v-if="imageUrl" class="img-preview">
          <img :src="imageUrl" alt="预览" />
        </div>
        <button class="btn primary" :disabled="!imageUrl" @click="analyzeImage">分析图片</button>
      </section>

      <!-- 实时人脸 -->
      <section v-show="active === 'face'" class="block">
        <h3>实时人脸识别</h3>
        <p class="hint">调用摄像头，模拟进行面部状态检测（疲劳、面色、专注度等参考指标）。</p>
        <div class="camera-wrap">
          <video ref="video" class="camera" autoplay muted playsinline></video>
          <canvas ref="overlay" class="camera-overlay"></canvas>
          <div v-if="!cameraOn" class="camera-placeholder">摄像头未开启</div>
        </div>
        <div class="voice-row">
          <button class="btn primary" @click="toggleCamera">
            {{ cameraOn ? '⏹ 关闭摄像头' : '📹 开启摄像头' }}
          </button>
          <button class="btn" :disabled="!cameraOn" @click="captureFace">📸 抓拍并分析</button>
        </div>
        <div v-if="!supportsCamera" class="warn">当前环境不支持摄像头访问（需 HTTPS 或 localhost）。</div>
      </section>

      <!-- 分析结果 -->
      <section v-if="result" class="result">
        <div class="result-head">📋 {{ resultTitle }}分析结果</div>
        <div class="result-body">{{ result }}</div>
        <div class="result-tags">
          <span v-for="(t, i) in resultTags" :key="i" class="tag">{{ t }}</span>
        </div>
      </section>
    </div>
  </div>
</template>

<script>
const KEYWORDS = [
  { words: ['失眠', '睡不着', '睡不好'], tag: '睡眠障碍', tip: '建议睡前 1 小时远离电子屏幕，可饮用温热的酸枣仁茶。' },
  { words: ['疲劳', '乏力', '没精神', '累'], tag: '气虚疲劳', tip: '注意规律作息，适当增加红枣、山药等补气食材。' },
  { words: ['头痛', '头晕'], tag: '头部不适', tip: '保证充足饮水与通风，持续头痛请及时就医。' },
  { words: ['胃口', '食欲不振', '吃不下'], tag: '脾胃偏弱', tip: '少食生冷，可用陈皮、茯苓煮水健脾。' },
  { words: ['焦虑', '压力大', '烦躁', '紧张'], tag: '情绪紧张', tip: '尝试每日 10 分钟冥想或八段锦放松练习。' },
  { words: ['咳嗽', '咽干', '喉咙'], tag: '肺燥', tip: '多喝温水，可食用雪梨、百合润肺。' },
  { words: ['怕冷', '手脚凉'], tag: '阳虚', tip: '注意保暖，适量食用羊肉、生姜温补。' }
]

export default {
  name: 'HealthSelfTest',
  data() {
    return {
      active: 'voice',
      tabs: [
        { key: 'voice', label: '🎤 语音' },
        { key: 'text', label: '✍️ 文字' },
        { key: 'image', label: '🖼️ 图片' },
        { key: 'face', label: '📹 人脸' }
      ],
      recording: false,
      voiceText: '',
      textInput: '',
      imageUrl: '',
      result: '',
      resultTitle: '',
      resultTags: [],
      cameraOn: false,
      mediaStream: null,
      rafId: null,
      recognition: null,
      supportsSpeech: 'webkitSpeechRecognition' in window || 'SpeechRecognition' in window,
      supportsCamera: !!(navigator.mediaDevices && navigator.mediaDevices.getUserMedia)
    }
  },
  beforeUnmount() {
    this.stopVoice()
    this.stopCamera()
  },
  methods: {
    /* ---------- 语音 ---------- */
    toggleVoice() {
      if (!this.supportsSpeech) return
      if (this.recording) {
        this.stopVoice()
      } else {
        this.startVoice()
      }
    },
    startVoice() {
      const SR = window.SpeechRecognition || window.webkitSpeechRecognition
      this.recognition = new SR()
      this.recognition.lang = 'zh-CN'
      this.recognition.interimResults = true
      this.recognition.continuous = true
      this.recognition.onresult = (e) => {
        let txt = ''
        for (let i = 0; i < e.results.length; i++) txt += e.results[i][0].transcript
        this.voiceText = txt
      }
      this.recognition.onend = () => {
        this.recording = false
      }
      this.recognition.start()
      this.recording = true
    },
    stopVoice() {
      if (this.recognition) {
        try {
          this.recognition.stop()
        } catch (e) {
          /* noop */
        }
      }
      this.recording = false
    },

    /* ---------- 图片 ---------- */
    onImage(e) {
      const file = e.target.files[0]
      if (!file) return
      this.imageUrl = URL.createObjectURL(file)
    },
    analyzeImage() {
      this.resultTitle = '图片'
      this.result =
        '【模拟识别】已接收图像。参考建议：请结合舌象、面色综合判断；日常保持饮食清淡、作息规律，异常变化建议咨询专业医师。'
      this.resultTags = ['图像已接收', '建议面诊复核']
    },

    /* ---------- 文本分析（语音/文字共用） ---------- */
    analyzeText(text, source) {
      const matched = []
      KEYWORDS.forEach((k) => {
        if (k.words.some((w) => text.includes(w))) matched.push(k)
      })
      if (matched.length === 0) {
        this.resultTitle = source === 'voice' ? '语音' : '文字'
        this.result =
          '未匹配到明显健康信号。请继续保持规律作息、均衡饮食与适度运动，并定期自测。'
        this.resultTags = ['状态平稳']
        return
      }
      this.resultTitle = source === 'voice' ? '语音' : '文字'
      this.result =
        '根据您的描述，识别到以下信号：\n' +
        matched.map((m) => `· ${m.tag}：${m.tip}`).join('\n')
      this.resultTags = matched.map((m) => m.tag)
    },

    /* ---------- 实时人脸 ---------- */
    async toggleCamera() {
      if (this.cameraOn) {
        this.stopCamera()
      } else {
        await this.startCamera()
      }
    },
    async startCamera() {
      try {
        this.mediaStream = await navigator.mediaDevices.getUserMedia({ video: true })
        const video = this.$refs.video
        video.srcObject = this.mediaStream
        this.cameraOn = true
        this.$nextTick(() => this.drawLoop())
      } catch (err) {
        this.supportsCamera = false
        alert('无法访问摄像头：' + err.message)
      }
    },
    stopCamera() {
      if (this.mediaStream) {
        this.mediaStream.getTracks().forEach((t) => t.stop())
        this.mediaStream = null
      }
      if (this.rafId) cancelAnimationFrame(this.rafId)
      this.rafId = null
      this.cameraOn = false
    },
    drawLoop() {
      const video = this.$refs.video
      const canvas = this.$refs.overlay
      if (!video || !canvas) return
      const w = (canvas.width = video.clientWidth)
      const h = (canvas.height = video.clientHeight)
      const ctx = canvas.getContext('2d')
      const loop = () => {
        ctx.clearRect(0, 0, w, h)
        // 模拟人脸检测框（可用 face-api.js 替换）
        const bw = w * 0.42
        const bh = h * 0.6
        const bx = w / 2 - bw / 2 + Math.sin(Date.now() / 600) * 6
        const by = h / 2 - bh / 2
        ctx.strokeStyle = '#7fc8a9'
        ctx.lineWidth = 3
        ctx.strokeRect(bx, by, bw, bh)
        ctx.fillStyle = 'rgba(127,200,169,0.15)'
        ctx.fillRect(bx, by, bw, bh)
        ctx.font = '13px sans-serif'
        ctx.fillStyle = '#7fc8a9'
        ctx.fillText('Face · 检测中…', bx, by - 8)
        this.rafId = requestAnimationFrame(loop)
      }
      loop()
    },
    captureFace() {
      const fatigue = 40 + Math.floor(Math.random() * 45)
      const focus = 55 + Math.floor(Math.random() * 40)
      this.resultTitle = '人脸'
      this.result =
        `【模拟检测】实时抓拍完成。\n` +
        `疲劳指数：${fatigue}/100（${fatigue > 70 ? '偏高，建议休息' : '正常'}）\n` +
        `专注度：${focus}/100\n` +
        `面色参考：红润度良好。请结合自感状态综合判断。`
      this.resultTags = ['人脸已抓拍', `疲劳 ${fatigue}`, `专注 ${focus}`]
    }
  }
}
</script>

<style scoped>
.module-head h2 {
  margin: 0 0 6px;
  color: #2f5d50;
}
.module-head p {
  margin: 0 0 18px;
  color: #7a8a82;
  font-size: 14px;
}
.tabs {
  display: flex;
  gap: 8px;
  margin-bottom: 16px;
  flex-wrap: wrap;
}
.tab {
  padding: 9px 16px;
  border: 1px solid #d8e3dd;
  background: #fff;
  border-radius: 20px;
  cursor: pointer;
  font-size: 14px;
  color: #5a6b62;
  transition: all 0.2s;
}
.tab.active {
  background: #2f5d50;
  color: #fff;
  border-color: #2f5d50;
}
.panel {
  background: #fff;
  border-radius: 14px;
  padding: 22px;
  box-shadow: 0 4px 18px rgba(47, 93, 80, 0.06);
}
.block h3 {
  margin: 0 0 8px;
  color: #2f5d50;
}
.hint {
  color: #8a9b93;
  font-size: 13px;
  margin: 0 0 14px;
}
.text-input {
  width: 100%;
  border: 1px solid #dce5e0;
  border-radius: 10px;
  padding: 12px;
  font-size: 14px;
  resize: vertical;
  outline: none;
  margin-bottom: 12px;
}
.text-input:focus {
  border-color: #7fc8a9;
}
.btn {
  padding: 10px 20px;
  border: 1px solid #2f5d50;
  background: #fff;
  color: #2f5d50;
  border-radius: 10px;
  cursor: pointer;
  font-size: 14px;
}
.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
.btn.primary {
  background: #2f5d50;
  color: #fff;
}
.btn.recording {
  background: #c0392b;
  border-color: #c0392b;
  color: #fff;
}
.voice-row {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 14px;
}
.pulse-dot {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background: #c0392b;
  animation: pulse 1s infinite;
}
@keyframes pulse {
  0% {
    transform: scale(0.8);
    opacity: 0.7;
  }
  100% {
    transform: scale(1.4);
    opacity: 0.2;
  }
}
.uploader {
  display: inline-block;
  padding: 14px 22px;
  border: 1px dashed #b9cabf;
  border-radius: 10px;
  cursor: pointer;
  color: #5a6b62;
  margin-bottom: 14px;
}
.uploader input {
  display: none;
}
.img-preview {
  margin-bottom: 14px;
}
.img-preview img {
  max-width: 260px;
  border-radius: 10px;
  border: 1px solid #e6ebe7;
}
.camera-wrap {
  position: relative;
  width: 100%;
  max-width: 420px;
  aspect-ratio: 4 / 3;
  background: #1f2a26;
  border-radius: 12px;
  overflow: hidden;
  margin-bottom: 14px;
}
.camera {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.camera-overlay {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
}
.camera-placeholder {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #9fb3a9;
}
.result {
  margin-top: 20px;
  background: #f1f7f4;
  border-left: 4px solid #7fc8a9;
  border-radius: 8px;
  padding: 16px;
}
.result-head {
  font-weight: 600;
  color: #2f5d50;
  margin-bottom: 8px;
}
.result-body {
  white-space: pre-line;
  font-size: 14px;
  line-height: 1.7;
  color: #3c4a44;
}
.result-tags {
  margin-top: 10px;
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}
.tag {
  background: #dcefe5;
  color: #2f5d50;
  padding: 4px 12px;
  border-radius: 14px;
  font-size: 12px;
}
.warn {
  background: #fff4e5;
  color: #b9770e;
  padding: 10px 14px;
  border-radius: 8px;
  font-size: 13px;
  margin-bottom: 12px;
}

@media (max-width: 640px) {
  .panel {
    padding: 16px 14px;
  }
  .voice-row {
    flex-wrap: wrap;
  }
  .img-preview img {
    max-width: 100%;
  }
  .result {
    padding: 14px 12px;
  }
}
</style>
