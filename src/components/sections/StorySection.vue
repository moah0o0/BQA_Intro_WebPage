<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'

defineEmits(['openManifesto'])

const sectionRef = ref(null)
const statsRef = ref(null)
const isVisible = ref(false)
const statsVisible = ref(false)
let observer = null
let statsObserver = null

// 통계 카운터
const displayStat1 = ref(0)
const displayStat2 = ref(0)
const displayStat3 = ref(0)
const displayStat4 = ref(0)

function animateValue(setter, target, duration, decimals = 0) {
  const start = performance.now()
  function update(currentTime) {
    const elapsed = currentTime - start
    const progress = Math.min(elapsed / duration, 1)
    const eased = 1 - Math.pow(1 - progress, 4)
    const value = target * eased
    setter(decimals > 0 ? parseFloat(value.toFixed(decimals)) : Math.round(value))
    if (progress < 1) requestAnimationFrame(update)
  }
  requestAnimationFrame(update)
}

watch(statsVisible, (val) => {
  if (val) {
    animateValue((v) => { displayStat1.value = v }, 22.6, 2000, 1)
    animateValue((v) => { displayStat2.value = v }, 34, 2000)
    animateValue((v) => { displayStat3.value = v }, 74, 2000)
    animateValue((v) => { displayStat4.value = v }, 4.3, 2000, 1)
  }
})

onMounted(() => {
  observer = new IntersectionObserver(
    (entries) => {
      if (entries[0] && entries[0].isIntersecting) isVisible.value = true
    },
    { threshold: 0.1 }
  )
  statsObserver = new IntersectionObserver(
    (entries) => {
      if (entries[0] && entries[0].isIntersecting) statsVisible.value = true
    },
    { threshold: 0.2 }
  )

  if (sectionRef.value) observer.observe(sectionRef.value)
  if (statsRef.value) statsObserver.observe(statsRef.value)
})

onUnmounted(() => {
  if (observer) observer.disconnect()
  if (statsObserver) statsObserver.disconnect()
})
</script>

<template>
  <section class="story" ref="sectionRef">
    <div class="content" :class="{ visible: isVisible }">


      <!-- 블록 2: 왜 모이는가 -->
      <div class="block">
        <p class="text">
          차별에 맞서는 힘은 몇몇 사람의 헌신만으로 오래 유지될 수 없고,
          부산에서 살아가는 평범한 퀴어들의 삶과 경험이 모일 때 비로소 커집니다.
          특히 일하고 공부하고 생계를 꾸려야 하는 다수의 퀴어가 함께할 수 있어야,
          퀴어의 문제가 '일부의 특수한 문제'가 아니라
          우리 사회의 구조적 문제라는 점을 드러내고,
          실제로 바꿀 힘도 만들 수 있습니다.
        </p>
      </div>

      <!-- 블록 3: 왜 노동인가 + 실태 + 통계 -->
      <div class="block">
        <p class="text">
          그렇다면 우리는 어떤 의제로 사람들을 모을 것인가.
          극소수를 제외한 대다수 퀴어에게,
          먹고사는 문제—곧 노동의 문제—는
          삶의 가장 중요한 현실 가운데 하나입니다.
        </p>
        <p class="text">
          퀴어는 구직 과정에서부터 차별을 겪습니다.
          "퀴어라서" 배제되기도 하고,
          사회가 정한 성별 규범에 맞지 않는다는 이유로
          구직 자체를 포기하거나 미루게 되기도 합니다.
          누적된 차별은 결국 많은 사람들을
          근로기준법조차 제대로 지켜지지 않는 열악한 일자리,
          노동권의 사각지대로 내몰기도 합니다.
        </p>
        <p class="text">
          어렵게 들어간 일터에서도 혐오 발언, 따돌림, 성희롱과 괴롭힘은 이어집니다.
          그러나 문제에 대응하려면 커밍아웃의 위험을 감수해야 하는 경우가 많아,
          많은 이들이 부당함을 참고 넘기게 됩니다.
          그 결과 일터는 생계를 위한 공간인 동시에,
          자신을 지키기 위해 끊임없이 긴장해야 하는 공간이 되기도 합니다.
        </p>
      </div>

      <!-- 통계 -->
      <div class="stats-area" ref="statsRef">
        <div class="stats-grid">
          <div class="stat-card">
            <span class="stat-label">구직 과정 차별</span>
            <span class="stat-value">{{ displayStat1.toFixed(1) }}<span class="unit">%</span></span>
          </div>
          <div class="stat-card">
            <span class="stat-label">근무 중 차별</span>
            <span class="stat-value">{{ displayStat2 }}<span class="unit">%</span></span>
          </div>
          <div class="stat-card">
            <span class="stat-label">부당함을 묵인</span>
            <span class="stat-value">{{ displayStat3 }}<span class="unit">%</span></span>
          </div>
          <div class="stat-card">
            <span class="stat-label">우울증상(vs 일반인)</span>
            <span class="stat-value">{{ displayStat4.toFixed(1) }}<span class="unit">배</span></span>
          </div>
        </div>
        <p class="source">
          * 성소수자 노동자 노동실태 및 정신건강 연구(2025)<br>
          * 청년 성소수자 사회적 욕구 및 실태 조사(2020)
        </p>
      </div>

      <!-- 마무리 + CTA -->
      <div class="divider-line"></div>

      <p class="text closing">
        하루의 대부분을 보내는 일터에서조차 퀴어로서 당당히 서기 어렵다면,
        차별에 맞서는 싸움이 뒷전으로 밀리는 것은 이상한 일이 아닙니다.
        그래서 부산퀴어행동은 퀴어 노동자가 당당하게 일할 수 있는 일터,
        그리고 모든 노동자가 인간다운 생활을 보장받는 세상을 향해 나아가고자 합니다.
      </p>

      <button class="cta-btn" @click="$emit('openManifesto')">
        부산퀴어행동의 다짐 전문 보기
      </button>
    </div>
  </section>
</template>

<style scoped>
.story {
  background: #faf8f6;
  padding: 80px 28px 90px;
}

.content {
  max-width: 640px;
  margin: 0 auto;
  opacity: 0;
  transform: translateY(30px);
  transition: all 1s ease;
}

.content.visible {
  opacity: 1;
  transform: translateY(0);
}

.block {
  margin-bottom: 48px;
}

.sub-heading {
  font-size: 1.25rem;
  font-weight: 800;
  color: var(--text-color, #000);
  margin: 0 0 16px 0;
  letter-spacing: -0.03em;
  line-height: 1.4;
}

.text {
  font-size: 1rem;
  font-weight: 500;
  line-height: 1.9;
  color: #444;
  margin: 0 0 20px 0;
  letter-spacing: -0.02em;
  word-break: keep-all;
}

.text:last-child {
  margin-bottom: 0;
}

.text.closing {
  margin-bottom: 32px;
}

/* 통계 */
.stats-area {
  margin: 12px 0 36px;
}

.stats-heading {
  font-size: 0.85rem;
  font-weight: 700;
  color: var(--accent-color, #8B25FF);
  margin: 0 0 16px 0;
  letter-spacing: -0.01em;
}

.stats-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
  margin-bottom: 16px;
}

.stat-card {
  background: #fff;
  border: 1px solid var(--border-color, #e5e5e5);
  border-radius: var(--radius-lg, 12px);
  padding: 20px 14px;
  text-align: center;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.03);
}

.stat-label {
  display: block;
  font-size: 0.8rem;
  font-weight: 600;
  color: #888;
  margin-bottom: 8px;
  letter-spacing: -0.01em;
  line-height: 1.3;
}

.stat-value {
  font-size: 2rem;
  font-weight: 800;
  letter-spacing: -0.03em;
  color: var(--accent-color, #8B25FF);
  line-height: 1;
}

.unit {
  font-size: 0.875rem;
  font-weight: 600;
  margin-left: 2px;
}

.source {
  font-size: 0.7rem;
  color: #aaa;
  line-height: 1.5;
  letter-spacing: -0.01em;
}

/* 구분선 */
.divider-line {
  width: 48px;
  height: 2px;
  background: var(--accent-color, #8B25FF);
  margin: 0 0 36px 0;
}

/* CTA */
.cta-btn {
  display: inline-block;
  padding: 14px 28px;
  background: var(--accent-color, #8B25FF);
  color: #fff;
  border: none;
  border-radius: 30px;
  font-size: 0.95rem;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.2s ease;
  font-family: inherit;
  letter-spacing: -0.02em;
}

.cta-btn:hover {
  opacity: 0.85;
  transform: translateY(-2px);
}

.cta-btn:active {
  transform: translateY(0);
  opacity: 0.9;
}

@media (max-width: 640px) {
  .story {
    padding: 60px 20px 70px;
  }
  .block {
    margin-bottom: 40px;
  }
  .sub-heading {
    font-size: 1.125rem;
  }
  .text {
    font-size: 0.9375rem;
    line-height: 1.85;
  }
  .stat-card {
    padding: 16px 10px;
  }
  .stat-value {
    font-size: 1.75rem;
  }
}

@media (max-width: 360px) {
  .stat-value {
    font-size: 1.5rem;
  }
  .stats-grid {
    gap: 8px;
  }
}

@media (hover: none) and (pointer: coarse) {
  .cta-btn:hover {
    opacity: 1;
    transform: none;
  }
  .cta-btn:active {
    opacity: 0.85;
    transform: scale(0.98);
  }
}
</style>
