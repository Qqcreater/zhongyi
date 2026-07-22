<template>
  <div class="module">
    <div class="module-head">
      <h2>🍲 食疗药膳专区</h2>
      <p>辨证施膳，按体质与季节挑选适合的养生膳食。</p>
    </div>

    <div class="filters">
      <button
        v-for="c in categories"
        :key="c"
        class="chip"
        :class="{ active: activeCat === c }"
        @click="activeCat = c"
      >
        {{ c }}
      </button>
    </div>

    <div class="grid">
      <article
        v-for="r in filtered"
        :key="r.id"
        class="card"
        @click="open = r.id"
      >
        <div class="card-emoji">{{ r.emoji }}</div>
        <div class="card-body">
          <div class="card-top">
            <h3>{{ r.name }}</h3>
            <span class="card-tag">{{ r.cat }}</span>
          </div>
          <p class="card-desc">{{ r.efficacy }}</p>
          <div class="card-foot">
            <span>⏱ {{ r.time }}</span>
            <span>🔥 {{ r.cal }} kcal</span>
          </div>
        </div>
      </article>
    </div>

    <!-- 详情弹窗 -->
    <div v-if="open !== null" class="modal-mask" @click.self="open = null">
      <div class="modal">
        <button class="modal-close" @click="open = null">✕</button>
        <div class="modal-emoji">{{ current.emoji }}</div>
        <h2>{{ current.name }}</h2>
        <span class="card-tag">{{ current.cat }}</span>
        <p class="modal-eff">{{ current.efficacy }}</p>
        <h4>🧾 食材</h4>
        <p>{{ current.ingredients }}</p>
        <h4>👥 适宜人群</h4>
        <p>{{ current.suitable }}</p>
        <h4>🍳 做法</h4>
        <ol>
          <li v-for="(s, i) in current.steps" :key="i">{{ s }}</li>
        </ol>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'MedicinalCuisine',
  data() {
    return {
      activeCat: '全部',
      open: null,
      categories: ['全部', '健脾', '养肝', '润肺', '补肾', '祛湿', '安神'],
      recipes: [
        {
          id: 1,
          name: '山药薏米粥',
          emoji: '🥣',
          cat: '健脾',
          efficacy: '健脾祛湿，适合脾胃虚弱、易浮肿者。',
          time: '40 分钟',
          cal: 180,
          ingredients: '山药 100g、薏米 50g、粳米 80g、枸杞少许。',
          suitable: '脾胃虚弱、舌苔厚腻、易疲劳人群。',
          steps: ['薏米提前浸泡 2 小时', '粳米与薏米同煮 30 分钟', '加入山药块煮 10 分钟', '撒枸杞焖 5 分钟']
        },
        {
          id: 2,
          name: '枸杞菊花茶',
          emoji: '🍵',
          cat: '养肝',
          efficacy: '清肝明目，缓解用眼过度。',
          time: '10 分钟',
          cal: 25,
          ingredients: '枸杞 10g、菊花 5g、冰糖少许。',
          suitable: '长期用眼、熬夜、肝火旺者。',
          steps: ['温水冲洗材料', '沸水冲泡焖 5 分钟', '可反复续水 2-3 次']
        },
        {
          id: 3,
          name: '冰糖雪梨羹',
          emoji: '🍐',
          cat: '润肺',
          efficacy: '润肺止咳，缓解秋燥咽干。',
          time: '35 分钟',
          cal: 120,
          ingredients: '雪梨 1 个、冰糖 15g、百合 10g。',
          suitable: '干咳无痰、咽干口燥者。',
          steps: ['雪梨去核切块', '与百合同煮 25 分钟', '加冰糖煮至化开']
        },
        {
          id: 4,
          name: '黑豆核桃糊',
          emoji: '🥜',
          cat: '补肾',
          efficacy: '补肾益精，适合腰膝酸软者。',
          time: '30 分钟',
          cal: 220,
          ingredients: '黑豆 60g、核桃 30g、黑芝麻 15g。',
          suitable: '肾虚、脱发、记忆力下降人群。',
          steps: ['黑豆浸泡过夜', '与核桃同煮软烂', '加黑芝麻打成糊']
        },
        {
          id: 5,
          name: '赤小豆冬瓜汤',
          emoji: '🍲',
          cat: '祛湿',
          efficacy: '利水消肿，适合湿热体质。',
          time: '50 分钟',
          cal: 90,
          ingredients: '赤小豆 50g、冬瓜 200g、陈皮 3g。',
          suitable: '水肿、四肢沉重、湿气重者。',
          steps: ['赤小豆先煮 30 分钟', '加陈皮冬瓜煮 15 分钟', '少盐调味']
        },
        {
          id: 6,
          name: '酸枣仁安神饮',
          emoji: '🌿',
          cat: '安神',
          efficacy: '养心安神，改善睡眠质量。',
          time: '15 分钟',
          cal: 40,
          ingredients: '酸枣仁 10g、百合 10g、桂圆 5 颗。',
          suitable: '失眠多梦、心悸易醒者。',
          steps: ['酸枣仁捣碎', '与百合同煮 10 分钟', '加桂圆焖片刻']
        }
      ]
    }
  },
  computed: {
    filtered() {
      if (this.activeCat === '全部') return this.recipes
      return this.recipes.filter((r) => r.cat === this.activeCat)
    },
    current() {
      return this.recipes.find((r) => r.id === this.open) || {}
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
.filters {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  margin-bottom: 20px;
}
.chip {
  padding: 7px 16px;
  border: 1px solid #d8e3dd;
  background: #fff;
  border-radius: 18px;
  cursor: pointer;
  font-size: 13px;
  color: #5a6b62;
}
.chip.active {
  background: #2f5d50;
  color: #fff;
  border-color: #2f5d50;
}
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 16px;
}
.card {
  background: #fff;
  border-radius: 14px;
  padding: 18px;
  display: flex;
  gap: 14px;
  cursor: pointer;
  box-shadow: 0 4px 18px rgba(47, 93, 80, 0.06);
  transition: transform 0.18s, box-shadow 0.18s;
}
.card:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 24px rgba(47, 93, 80, 0.12);
}
.card-emoji {
  font-size: 40px;
  flex-shrink: 0;
}
.card-body {
  flex: 1;
}
.card-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.card-top h3 {
  margin: 0;
  font-size: 16px;
  color: #2f5d50;
}
.card-tag {
  background: #dcefe5;
  color: #2f5d50;
  font-size: 11px;
  padding: 3px 10px;
  border-radius: 12px;
}
.card-desc {
  font-size: 13px;
  color: #7a8a82;
  margin: 8px 0;
  line-height: 1.5;
}
.card-foot {
  display: flex;
  gap: 14px;
  font-size: 12px;
  color: #9aa9a1;
}
.modal-mask {
  position: fixed;
  inset: 0;
  background: rgba(20, 40, 34, 0.45);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 50;
}
.modal {
  background: #fff;
  border-radius: 16px;
  padding: 28px;
  width: 440px;
  max-width: 92vw;
  max-height: 86vh;
  overflow-y: auto;
  position: relative;
}
.modal-close {
  position: absolute;
  top: 14px;
  right: 14px;
  border: none;
  background: #f1f4f1;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  cursor: pointer;
}
.modal-emoji {
  font-size: 52px;
  text-align: center;
}
.modal h2 {
  text-align: center;
  margin: 6px 0;
  color: #2f5d50;
}
.modal-eff {
  text-align: center;
  color: #7a8a82;
  font-size: 14px;
}
.modal h4 {
  color: #2f5d50;
  margin: 16px 0 6px;
}
.modal p {
  font-size: 14px;
  color: #3c4a44;
  line-height: 1.6;
}
.modal ol {
  padding-left: 20px;
  font-size: 14px;
  color: #3c4a44;
  line-height: 1.8;
}
</style>
