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


      <h2 class="story-hook">퀴어와 노동</h2>

      <div class="block">
        <p class="text">
          부산퀴어행동은 부산에서 살아가는 퀴어들이 사회운동을 '나와 먼 일'로 느끼지 않도록, 일상 속에서 꾸준히 참여할 수 있는 활동의 공간을 열어내고 싶습니다. 그래서 우리는 <strong>'노동'</strong>에 주목합니다. 우리 대부분은 타인과 관계를 맺고 사회에서 살아가기 위해, 삶의 많은 시간을 일터에서 보내야 하기 때문입니다.
        </p>
        <p class="text">
          그런데 퀴어로 일한다는 건 종종 녹록지 않습니다. 구직 과정에서부터 '퀴어라는 이유로' 불이익을 겪기도 하고, 어렵게 일터에 들어서도 정체성을 드러내기 어려운 순간들이 이어지곤 합니다. 부당한 대우나 차별을 겪어도 대응하려면 <mark>커밍아웃의 위험을 감수</mark>해야 하는 경우가 많아, 참고 넘기게 될 때도 있습니다.
        </p>
        <p class="text">
          또 우리 중 많은 사람에게는, 살아오며 누적된 고립과 배제의 경험이 '일하는 것' 자체를 더 두렵고 불안하게 만들기도 합니다. 특히 청소년기부터 가족·학교·또래집단에서 소외되거나 고립된 경험이 있었던 퀴어들은 <mark>더 낮은 임금, 더 불안정한 고용, 더 큰 해고 위험을 감당해야 하는 자리</mark>로 밀려나기 쉽습니다.
        </p>
        <p class="text">
          그래서 부산퀴어행동은, 노동하며 살아가는 퀴어들, 취업준비·학업 같은 과제에 지쳐 있거나 '뒤처지고 실패했다'고 느끼는 순간을 겪는 퀴어들이 부담 없이 들어와서 들렀다 갈 수 있는 모임이 되고자 합니다. 거기서 답답한 마음을 함께 말하고, 서로 부대끼기도 하고, 시행착오를 겪더라도 이것저것 해보는 경험을 같이 쌓고 싶습니다. 그렇게 해서 <mark>우리 스스로 일터와 일상에서 마주치는 부당함을 조금씩 바꿔낼 힘</mark>을, 그리고 서로를 지지하는 힘을 함께 키워가고 싶습니다.
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
          * 다양성을 향한 지속가능한 움직임, 다움 [청년 성소수자 사회적 욕구 및 실태 조사](2021)<br>
          * ²한국노동안전보건연구소 젠더와노동건강권센터 및 퀴어노동법률지원네트워크 퀴어동네 [성소수자 노동자 노동실태 및 정신건강 연구](2025)<br>
          * 국가인권위원회 [트랜스젠더 혐오차별 실태조사](2020)
        </p>
      </div>

      <!-- CTA -->
      <div class="divider-line"></div>

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
