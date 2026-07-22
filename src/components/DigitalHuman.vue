<template>
  <div class="module">
    <div class="module-head">
      <h2>🤖 数字人导诊</h2>
      <p>专属虚拟养生顾问，实时为你讲解与陪伴。</p>
    </div>

    <div class="stage">
      <div class="avatar" :class="{ speaking: speaking }">
        <div class="avatar-face">
          <div class="eye left"></div>
          <div class="eye right"></div>
          <div class="mouth"></div>
        </div>
        <div class="ring"></div>
        <div class="avatar-name">小颐</div>
      </div>

      <div class="dialog">
        <p v-if="!speaking && !lastMsg" class="dialog-idle">
          你好，我是你的数字养生顾问小颐，想了解点什么？
        </p>
        <p v-else class="dialog-msg">{{ lastMsg }}</p>
      </div>
    </div>

    <div class="topics">
      <button
        v-for="t in topics"
        :key="t.q"
        class="topic"
        @click="ask(t)"
      >
        {{ t.q }}
      </button>
    </div>

    <div class="controls">
      <button class="btn" @click="stopSpeak">⏹ 停止播报</button>
      <label class="switch">
        <input type="checkbox" v-model="voiceOn" /> 语音播报
      </label>
    </div>
  </div>
</template>

<script>
const TOPICS = [
  { q: '今天怎么养生？', a: '建议早睡早起，晨起一杯温水，午后可做八段锦舒展筋骨，饮食宜清淡温润。' },
  { q: '失眠怎么办？', a: '睡前一小时远离屏幕，泡脚十五分钟，饮用酸枣仁茶，并保持卧室黑暗安静。' },
  { q: '秋季如何调理？', a: '秋季重在润肺防燥，可食雪梨、百合、银耳，注意增减衣物，避免着凉。' },
  { q: '久坐如何保健？', a: '每小时起身活动，做肩颈拉伸与眼部远眺，日常多饮温水促进代谢。' },
  { q: '怎么改善气色？', a: '保证睡眠与蛋白质摄入，适度有氧运动，多吃红枣、枸杞等补益食材。' }
]

export default {
  name: 'DigitalHuman',
  data() {
    return {
      topics: TOPICS,
      lastMsg: '',
      speaking: false,
      voiceOn: true
    }
  },
  beforeUnmount() {
    this.stopSpeak()
  },
  methods: {
    ask(t) {
      this.lastMsg = t.a
      this.speak(t.a)
    },
    speak(text) {
      this.stopSpeak()
      if (!this.voiceOn || !('speechSynthesis' in window)) return
      const u = new SpeechSynthesisUtterance(text)
      u.lang = 'zh-CN'
      u.onstart = () => (this.speaking = true)
      u.onend = () => (this.speaking = false)
      speechSynthesis.speak(u)
    },
    stopSpeak() {
      if ('speechSynthesis' in window) speechSynthesis.cancel()
      this.speaking = false
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
.stage {
  background: linear-gradient(160deg, #eef6f1, #dcefe5);
  border-radius: 18px;
  padding: 30px;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.avatar {
  position: relative;
  width: 150px;
  height: 150px;
  display: flex;
  align-items: center;
  justify-content: center;
}
.avatar-face {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  background: #fff;
  box-shadow: 0 8px 24px rgba(47, 93, 80, 0.2);
  position: relative;
  z-index: 2;
}
.eye {
  position: absolute;
  top: 44px;
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: #2f5d50;
}
.eye.left {
  left: 34px;
}
.eye.right {
  right: 34px;
}
.mouth {
  position: absolute;
  bottom: 34px;
  left: 50%;
  transform: translateX(-50%);
  width: 36px;
  height: 8px;
  border-radius: 0 0 18px 18px;
  background: #2f5d50;
}
.avatar.speaking .mouth {
  animation: talk 0.35s infinite alternate;
}
@keyframes talk {
  from {
    height: 6px;
  }
  to {
    height: 18px;
  }
}
.ring {
  position: absolute;
  inset: 0;
  border-radius: 50%;
  border: 3px solid #7fc8a9;
  opacity: 0.4;
}
.avatar.speaking .ring {
  animation: ripple 1.2s infinite;
}
@keyframes ripple {
  0% {
    transform: scale(1);
    opacity: 0.4;
  }
  100% {
    transform: scale(1.25);
    opacity: 0;
  }
}
.avatar-name {
  position: absolute;
  bottom: -8px;
  background: #2f5d50;
  color: #fff;
  font-size: 12px;
  padding: 3px 12px;
  border-radius: 12px;
  z-index: 3;
}
.dialog {
  margin-top: 26px;
  background: #fff;
  border-radius: 14px;
  padding: 16px 22px;
  min-height: 60px;
  max-width: 480px;
  text-align: center;
}
.dialog-idle {
  color: #8a9b93;
  margin: 0;
}
.dialog-msg {
  color: #3c4a44;
  margin: 0;
  line-height: 1.7;
  font-size: 15px;
}
.topics {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin: 20px 0;
}
.topic {
  padding: 10px 18px;
  border: 1px solid #d8e3dd;
  background: #fff;
  border-radius: 20px;
  cursor: pointer;
  font-size: 14px;
  color: #2f5d50;
  transition: all 0.2s;
}
.topic:hover {
  background: #2f5d50;
  color: #fff;
}
.controls {
  display: flex;
  align-items: center;
  gap: 18px;
}
.btn {
  padding: 9px 18px;
  border: 1px solid #2f5d50;
  background: #fff;
  color: #2f5d50;
  border-radius: 10px;
  cursor: pointer;
}
.switch {
  font-size: 14px;
  color: #5a6b62;
  display: flex;
  align-items: center;
  gap: 6px;
}
</style>
