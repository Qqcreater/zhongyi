<template>
  <div class="module">
    <div class="module-head">
      <h2>🧘 养生课程内容</h2>
      <p>跟练名师课程，科学管理你的学习进度。</p>
    </div>

    <div class="stats">
      <div class="stat">
        <div class="stat-num">{{ enrolledCount }}</div>
        <div class="stat-label">已报名</div>
      </div>
      <div class="stat">
        <div class="stat-num">{{ totalMinutes }}</div>
        <div class="stat-label">学习分钟</div>
      </div>
      <div class="stat">
        <div class="stat-num">{{ avgProgress }}%</div>
        <div class="stat-label">平均进度</div>
      </div>
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

    <div class="list">
      <div v-for="c in filtered" :key="c.id" class="course">
        <div class="course-cover">{{ c.emoji }}</div>
        <div class="course-main">
          <div class="course-top">
            <h3>{{ c.title }}</h3>
            <span class="level" :class="c.level">{{ c.level }}</span>
          </div>
          <p class="course-sub">{{ c.teacher }} · {{ c.duration }} 分钟 · {{ c.cat }}</p>
          <div class="progress-bar">
            <div class="progress-fill" :style="{ width: c.progress + '%' }"></div>
          </div>
          <div class="course-foot">
            <span class="progress-text">{{ c.progress }}%</span>
            <button
              class="btn"
              :class="{ enrolled: c.enrolled }"
              @click="toggleEnroll(c)"
            >
              {{ c.enrolled ? '继续学习' : '报名学习' }}
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'HealthCourses',
  data() {
    return {
      activeCat: '全部',
      categories: ['全部', '太极', '瑜伽', '冥想', '八段锦', '营养'],
      courses: [
        { id: 1, title: '八段锦入门十二式', emoji: '🧎', cat: '八段锦', teacher: '李道长', duration: 25, level: '入门', progress: 0, enrolled: false },
        { id: 2, title: '清晨太极云手练习', emoji: '🌅', cat: '太极', teacher: '王师傅', duration: 30, level: '进阶', progress: 0, enrolled: false },
        { id: 3, title: '睡前正念冥想 10 分钟', emoji: '🌙', cat: '冥想', teacher: '周老师', duration: 10, level: '入门', progress: 0, enrolled: false },
        { id: 4, title: '办公室肩颈舒缓瑜伽', emoji: '🧘‍♀️', cat: '瑜伽', teacher: '林教练', duration: 20, level: '入门', progress: 0, enrolled: false },
        { id: 5, title: '中医四季养生营养课', emoji: '🥗', cat: '营养', teacher: '陈医师', duration: 45, level: '进阶', progress: 0, enrolled: false },
        { id: 6, title: '五行经络拍打操', emoji: '✋', cat: '八段锦', teacher: '李道长', duration: 15, level: '入门', progress: 0, enrolled: false }
      ]
    }
  },
  computed: {
    filtered() {
      if (this.activeCat === '全部') return this.courses
      return this.courses.filter((c) => c.cat === this.activeCat)
    },
    enrolledCount() {
      return this.courses.filter((c) => c.enrolled).length
    },
    totalMinutes() {
      return this.courses.reduce((s, c) => s + Math.round((c.progress / 100) * c.duration), 0)
    },
    avgProgress() {
      if (this.courses.length === 0) return 0
      return Math.round(this.courses.reduce((s, c) => s + c.progress, 0) / this.courses.length)
    }
  },
  methods: {
    toggleEnroll(c) {
      if (!c.enrolled) {
        c.enrolled = true
        c.progress = 5
      } else {
        // 模拟继续学习，进度增加
        c.progress = Math.min(100, c.progress + 15)
      }
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
.stats {
  display: flex;
  gap: 14px;
  margin-bottom: 20px;
}
.stat {
  flex: 1;
  background: #fff;
  border-radius: 12px;
  padding: 16px;
  text-align: center;
  box-shadow: 0 4px 18px rgba(47, 93, 80, 0.06);
}
.stat-num {
  font-size: 26px;
  font-weight: 700;
  color: #2f5d50;
}
.stat-label {
  font-size: 12px;
  color: #8a9b93;
  margin-top: 4px;
}
.filters {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  margin-bottom: 18px;
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
.list {
  display: flex;
  flex-direction: column;
  gap: 14px;
}
.course {
  background: #fff;
  border-radius: 14px;
  padding: 16px;
  display: flex;
  gap: 16px;
  box-shadow: 0 4px 18px rgba(47, 93, 80, 0.06);
}
.course-cover {
  width: 76px;
  height: 76px;
  border-radius: 12px;
  background: #eef6f1;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 38px;
  flex-shrink: 0;
}
.course-main {
  flex: 1;
}
.course-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.course-top h3 {
  margin: 0;
  font-size: 16px;
  color: #2f5d50;
}
.level {
  font-size: 11px;
  padding: 3px 10px;
  border-radius: 12px;
}
.level.入门 {
  background: #e3f1ea;
  color: #2f5d50;
}
.level.进阶 {
  background: #fbe9d0;
  color: #b9770e;
}
.course-sub {
  font-size: 13px;
  color: #8a9b93;
  margin: 6px 0 10px;
}
.progress-bar {
  height: 8px;
  background: #eef2ef;
  border-radius: 6px;
  overflow: hidden;
}
.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #7fc8a9, #2f5d50);
  transition: width 0.3s;
}
.course-foot {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 10px;
}
.progress-text {
  font-size: 13px;
  color: #5a6b62;
}
.btn {
  padding: 8px 18px;
  border: 1px solid #2f5d50;
  background: #fff;
  color: #2f5d50;
  border-radius: 10px;
  cursor: pointer;
  font-size: 13px;
}
.btn.enrolled {
  background: #2f5d50;
  color: #fff;
}
</style>
