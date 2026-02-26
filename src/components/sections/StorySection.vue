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


      <h2 class="story-hook">부산퀴어행동은<br>뭘 꿈꾸나요?</h2>

      <!-- 블록: 왜 모이는가 -->
      <div class="block">
        <p class="text">
          차별에 맞서는 힘은 몇몇 사람의 헌신만으로 오래 유지될 수 없습니다.
          <strong>부산에서 살아가는 평범한 퀴어들의 삶과 경험이 모일 때,</strong> 그 힘은 비로소 커집니다.
        </p>
        <p class="text">
          특히 일하고, 공부하고, 생계를 꾸려나가느라
          <mark>세상을 바꾸는 일이 나랑 먼일처럼 느껴지는</mark> 다수의 퀴어가 함께할 수 있어야 합니다.
          그래야 퀴어의 문제가 '일부의 특수한 문제'가 아니라
          우리 사회의 구조적 문제라는 점을 드러낼 수 있고,
          실제로 바꿀 힘도 모을 수 있습니다.
        </p>
        <p class="text">
          그렇다면 어떤 주제로 사람들에게 함께하자고 제안할 것인가?
          부산퀴어행동은 그 중요한 출발점 가운데 하나가 <strong>노동의 문제</strong>라고 생각합니다.
        </p>
        <p class="text">
          극소수를 제외한 대다수 퀴어에게, 먹고사는 문제는 삶의 가장 중요한 현실입니다.
          퀴어는 구직 과정에서부터 차별을 겪습니다.
          "퀴어라서" 배제되기도 하고,
          사회가 요구하는 성별 규범에 맞지 않는다는 이유로
          구직 자체를 포기하거나 미루게 되기도 합니다.
          이렇게 누적된 차별은 많은 사람들을
          근로기준법조차 제대로 지켜지지 않는 열악한 일자리와
          노동권의 사각지대로 내몰기도 합니다.
          특히 <mark>트랜스젠더들은 저임금, 불안정 노동, 해고 위험을 더 크게 떠안고</mark> 살아갑니다.
        </p>
        <p class="text">
          어렵게 들어간 일터에서도 혐오 발언, 따돌림, 성희롱과 괴롭힘을 겪게 됩니다.
          하지만 문제에 대응하려면 <mark>커밍아웃의 위험을 감수</mark>해야 하는 경우가 많아
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
            <span class="stat-label">부당함을 참거나 묵인</span>
            <span class="stat-value">{{ displayStat3 }}<span class="unit">%</span></span>
          </div>
          <div class="stat-card">
            <span class="stat-label">우울증상(vs 일반인구집단)</span>
            <span class="stat-value">{{ displayStat4.toFixed(1) }}<span class="unit">배</span></span>
          </div>
          <div class="stat-card">
            <span class="stat-label">월 평균임금이 200만원 미만인<br>트랜스젠더 비율</span>
            <span class="stat-value">85<span class="unit">%</span></span>
          </div>
          <div class="stat-card">
            <span class="stat-label">부당한 대우, 사직 권유나 해고·재계약 거절 경험이 있는 트랜스젠더 비율</span>
            <span class="stat-value">43.6<span class="unit">%</span></span>
          </div>
        </div>
        <p class="source">
          * 다움, 청년 성소수자 사회적 욕구 및 실태 조사(2021)<br>
          * 한국노동안전보건연구소 등, 성소수자 노동자 노동실태 및 정신건강 연구(2025)<br>
          * 국가인권위원회, 트랜스젠더 혐오차별 실태조사(2020)
        </p>
      </div>

      <!-- 마무리 + CTA -->
      <div class="divider-line"></div>

      <p class="text closing">
        이렇듯 하루의 대부분을 보내는 공간인 일터에서조차 퀴어로서 당당히 서기 어려운데,
        어떻게 단체에 가입하고 집회에 나오고 하는 일이 쉽게 가능하겠습니까?
        그래서 <strong>삶에서 중요한 부분일 수밖에 없는 일터의 문제를 빼놓지 않고 이야기하는 퀴어운동</strong>이 필요합니다.
      </p>
      <p class="text closing">
        그래서 부산퀴어행동은 <mark>퀴어 노동자가 당당하게 일할 수 있는 일터,
        그리고 이와 연결된 모든 노동자의 차별 없이 일할 권리를 실현하는 길</mark>에 함께하고자 합니다.
        또한 이 싸움이 운동에 '올인'할 수 있는 사람만의 것이 되지 않도록,
        일하고 공부하고 생계를 꾸리는 퀴어들도 자기 삶의 자리에서
        세상을 바꾸는 일에 당당하게 나설 수 있는 힘과 조건을 함께 만들고자 합니다.
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

.story-hook {
  font-size: 1.75rem;
  font-weight: 900;
  color: var(--text-color, #000);
  letter-spacing: -0.03em;
  line-height: 1.4;
  margin: 0 0 40px 0;
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
  color: black;
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

.text strong {
  font-weight: 800;
  color: var(--text-color, #000);
}

.text mark {
  background: linear-gradient(to top, rgba(139, 37, 255, 0.2) 0%, rgba(139, 37, 255, 0.2) 40%, transparent 40%);
  color: inherit;
  padding: 0 2px;
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
  color: black;
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
  min-height: 48px;
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
    padding: 60px 24px 70px;
    padding-left: max(24px, env(safe-area-inset-left, 0px));
    padding-right: max(24px, env(safe-area-inset-right, 0px));
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
  .stat-label {
    font-size: 0.75rem;
  }
  .stat-value {
    font-size: 1.75rem;
  }
  .cta-btn {
    width: 100%;
    text-align: center;
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
