<template>
  <div class="module">
    <div class="module-head">
      <h2>✨ AI 养生助手</h2>
      <p>随时提问，获取个性化养生建议。</p>
    </div>

    <div class="chat">
      <div class="messages" ref="box">
        <div
          v-for="(m, i) in messages"
          :key="i"
          class="msg"
          :class="m.role"
        >
          <span class="msg-avatar">{{ m.role === 'user' ? '🧑' : '🤖' }}</span>
          <div class="bubble">{{ m.text }}</div>
        </div>
      </div>

      <div class="quick">
        <button
          v-for="q in quick"
          :key="q"
          class="quick-btn"
          @click="send(q)"
        >
          {{ q }}
        </button>
      </div>

      <div class="input-row">
        <input
          v-model="input"
          type="text"
          placeholder="输入你的问题，如：熬夜后如何恢复？"
          @keyup.enter="send()"
        />
        <button class="btn primary" :disabled="!input.trim()" @click="send()">发送</button>
      </div>
    </div>
  </div>
</template>

<script>
const KB = [
  { keys: ['失眠', '睡不着', '睡眠'], ans: '改善睡眠可从三方面入手：固定作息、睡前温水泡脚、卧室保持黑暗安静；必要时可饮用酸枣仁茶辅助。' },
  { keys: ['熬夜', '晚睡'], ans: '熬夜后建议次日午休 20 分钟、多补充维 C 与水分、清淡饮食，并尽快恢复规律作息，避免连续熬夜。' },
  { keys: ['减肥', '瘦身', '胖'], ans: '健康减重重在热量缺口与代谢：增加优质蛋白与蔬菜、规律有氧运动、保证睡眠，切忌极端节食。' },
  { keys: ['脾胃', '消化', '胃口'], ans: '养脾胃需少食生冷、细嚼慢咽、可常食山药薏米粥；饭后散步助消化，避免暴饮暴食。' },
  { keys: ['降压', '高血压'], ans: '日常控压建议低盐饮食、规律有氧运动、保持情绪平稳；请遵医嘱用药，本建议不能替代医疗。' },
  { keys: ['上火', '口腔溃疡'], ans: '“上火”可多饮水、食苦瓜绿豆等清润食物、保证睡眠；反复口腔溃疡建议补充维生素 B 族。' },
  { keys: ['感冒', '咳嗽'], ans: '外感初期多休息多饮温水，咳嗽可饮冰糖雪梨羹润肺；高热或持续症状请及时就医。' },
  { keys: ['运动', '锻炼'], ans: '推荐每日 30 分钟中等强度运动如快走、八段锦或太极，循序渐进，避免久坐。' },
  { keys: ['饮食', '吃什么'], ans: '均衡膳食遵循“五谷为养、五果为助、五畜为益、五菜为充”，少油少盐、多蔬果全谷。' }
]

export default {
  name: 'AiAssistant',
  data() {
    return {
      input: '',
      quick: ['失眠怎么办？', '熬夜后如何恢复？', '怎么养脾胃？', '适合的运动？'],
      messages: [
        { role: 'ai', text: '你好，我是 AI 养生助手。关于睡眠、饮食、运动或调理的问题都可以问我～' }
      ]
    }
  },
  computed: {
    box() {
      return this.$refs.box
    }
  },
  methods: {
    send(text) {
      const content = (text !== undefined ? text : this.input).trim()
      if (!content) return
      this.messages.push({ role: 'user', text: content })
      this.input = ''
      this.scroll()
      setTimeout(() => {
        this.messages.push({ role: 'ai', text: this.reply(content) })
        this.scroll()
      }, 450)
    },
    reply(q) {
      const hit = KB.find((k) => k.keys.some((w) => q.includes(w)))
      if (hit) return hit.ans
      return '这个问题我还在学习中～你可以试试问我关于失眠、脾胃、运动、饮食或熬夜恢复的话题，也可以前往「健康自测」做初步评估。'
    },
    scroll() {
      this.$nextTick(() => {
        if (this.box) this.box.scrollTop = this.box.scrollHeight
      })
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
.chat {
  background: #fff;
  border-radius: 14px;
  padding: 18px;
  box-shadow: 0 4px 18px rgba(47, 93, 80, 0.06);
  display: flex;
  flex-direction: column;
  height: 560px;
}
.messages {
  flex: 1;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 14px;
  padding-right: 6px;
}
.msg {
  display: flex;
  gap: 10px;
  max-width: 80%;
}
.msg.user {
  align-self: flex-end;
  flex-direction: row-reverse;
}
.msg-avatar {
  font-size: 26px;
  flex-shrink: 0;
}
.bubble {
  padding: 11px 15px;
  border-radius: 14px;
  font-size: 14px;
  line-height: 1.6;
}
.msg.ai .bubble {
  background: #eef6f1;
  color: #3c4a44;
  border-top-left-radius: 4px;
}
.msg.user .bubble {
  background: #2f5d50;
  color: #fff;
  border-top-right-radius: 4px;
}
.quick {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin: 14px 0;
}
.quick-btn {
  padding: 7px 14px;
  border: 1px solid #d8e3dd;
  background: #f7faf8;
  border-radius: 16px;
  cursor: pointer;
  font-size: 13px;
  color: #2f5d50;
}
.quick-btn:hover {
  background: #eef6f1;
}
.input-row {
  display: flex;
  gap: 10px;
}
.input-row input {
  flex: 1;
  border: 1px solid #dce5e0;
  border-radius: 10px;
  padding: 11px 14px;
  font-size: 14px;
  outline: none;
}
.input-row input:focus {
  border-color: #7fc8a9;
}
.btn {
  padding: 11px 22px;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  font-size: 14px;
}
.btn.primary {
  background: #2f5d50;
  color: #fff;
}
.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>
