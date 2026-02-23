<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const messages = [
  '👷‍♀️ 일하는 퀴어',
  '🧑‍🎓 취업준비와 학업이 고민인 청년 퀴어',
  '💬️️ 사회가 요구하는 과제들을 따라가기 벅찬 퀴어',
  '🌏 그 밖에 먹고사는 것이 고민인 모든 퀴어',
]

// 배경 슬라이드쇼: 각 사업의 첫 번째 사진
const bgImages = [
  '/images/2025/1.jpeg',
  '/images/2025/5.jpeg',
  '/images/2025/9.jpeg',
  '/images/2025/13.jpeg',
  '/images/2025/17.jpeg',
  '/images/2025/22.jpeg',
  '/images/2025/28.jpeg',
  '/images/2025/31.jpeg',
  '/images/2025/36.jpeg',
  '/images/2025/40.jpeg',
  '/images/2025/45.jpeg',
]
const currentBg = ref(0)
let bgInterval = null

const typedText = ref('')
const showCursor = ref(false)
const preLineShow = ref(false)
const postLineShow = ref(false)
const bottomShow = ref(false)
const originShow = ref(false)
const currentMsg = ref(0)

let alive = true

function sleep(ms) {
  return new Promise(r => setTimeout(r, ms))
}

function getGraphemes(text) {
  const segmenter = new Intl.Segmenter('ko', { granularity: 'grapheme' })
  return [...segmenter.segment(text)].map(s => s.segment)
}

async function typeText(text) {
  showCursor.value = true
  const graphemes = getGraphemes(text)
  for (let i = 0; i < graphemes.length && alive; i++) {
    typedText.value = graphemes.slice(0, i + 1).join('')
    await sleep(60)
  }
}

async function deleteText() {
  const graphemes = getGraphemes(typedText.value)
  for (let i = graphemes.length; i > 0 && alive; i--) {
    typedText.value = graphemes.slice(0, i - 1).join('')
    await sleep(30)
  }
}

async function runCycle() {
  await sleep(400)
  preLineShow.value = true
  await sleep(700)

  while (alive) {
    const msg = messages[currentMsg.value]

    await typeText(msg)
    await sleep(400)

    postLineShow.value = true
    await sleep(300)
    bottomShow.value = true
    showCursor.value = false

    await sleep(2000)
    originShow.value = true
    await sleep(2000)
    if (!alive) break

    postLineShow.value = false
    bottomShow.value = false
    await sleep(500)

    showCursor.value = true
    await deleteText()
    await sleep(300)

    currentMsg.value = (currentMsg.value + 1) % messages.length
  }
}

onMounted(() => {
  runCycle()
  bgInterval = setInterval(() => {
    currentBg.value = (currentBg.value + 1) % bgImages.length
  }, 4000)
})

onUnmounted(() => {
  alive = false
  if (bgInterval) clearInterval(bgInterval)
})
</script>

<template>
  <section class="hero">
    <div class="hero-bg">
      <img
        v-for="(src, i) in bgImages"
        :key="src"
        :src="src"
        :class="{ active: currentBg === i }"
        class="hero-bg-img"
        alt=""
      />
      <div class="hero-bg-overlay"></div>
    </div>
    <div class="hero-content">
      <div class="slogan-area">
        <p class="pre-line" :class="{ show: preLineShow }">부산퀴어행동은</p>
        <h1 class="main-line">
          <span class="typed">{{ typedText }}</span><span v-if="showCursor" class="cursor">|</span><span class="post-text" :class="{ show: postLineShow }">의 힘으로 세상을 바꾸고 싶습니다.</span>
        </h1>
      </div>

      <div class="origin-story" :class="{ show: originShow }">
        <p class="origin-label">부산지역 퀴어들은</p>
        <p class="origin-text">
          12·3 내란에 맞서 열린 부산지역 윤석열 퇴진광장에서 매주 만나 뒤풀이를 하고, 존재를 당당히 드러내는 깃발을 들고서 '무지개존'을 꾸려가며 서로에게 든든한 곁이 되어 왔습니다.
          </p>
        <p class="origin-label">하지만 광장이 닫힌 이후</p>
        <p class="origin-text">
          마주해야 하는 것은
          <mark>변함없는 차별과 혐오의 세상</mark>이었습니다.
          그래서 2025년 2월 28일, 일상 속에서 서로의 용기가 되어주기로 다짐하며,
          <strong>부산퀴어행동을 발족하게 되었습니다.</strong>
        </p>
      </div>

      <div class="scroll-hint" :class="{ show: bottomShow }">
        <svg width="16" height="16" viewBox="0 0 20 20" fill="none">
          <path d="M10 4V16M10 16L5 11M10 16L15 11" stroke="currentColor"
                stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
      </div>
    </div>
  </section>
</template>

<style scoped>
.hero {
  min-height: 100dvh;
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 80px 28px 40px;
  overflow: hidden;
}

.hero-bg {
  position: absolute;
  inset: 0;
  z-index: 0;
}

.hero-bg-img {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  opacity: 0;
  transition: opacity 1.5s ease;
}

.hero-bg-img.active {
  opacity: 1;
}

.hero-bg-overlay {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.82);
}


.hero-content {
  position: relative;
  z-index: 1;
  max-width: 600px;
  margin: 0 auto;
  width: 100%;
  text-shadow: 0 2px 8px rgba(0, 0, 0, 0.7), 0 4px 24px rgba(0, 0, 0, 0.5);
}

.slogan-area {
  padding: 0 4px;
  margin-bottom: 36px;
  width: 100%;
}

.pre-line {
  font-size: 1.6rem;
  font-weight: 800;
  color: rgba(255, 255, 255, 0.9);
  letter-spacing: -0.02em;
  margin-bottom: 12px;
  opacity: 0;
  transform: translateY(12px);
  transition: all 0.6s cubic-bezier(0.22, 1, 0.36, 1);
}

.pre-line.show {
  opacity: 1;
  transform: translateY(0);
}

.main-line {
  font-size: 2.8rem;
  font-weight: 900;
  letter-spacing: -0.04em;
  color: var(--accent-color, #8B25FF);
  line-height: 1.3;
  word-break: keep-all;
}

.typed {
  display: inline;
}

.cursor {
  display: inline;
  font-weight: 300;
  color: var(--accent-color, #8B25FF);
  animation: blink 0.6s step-end infinite;
  will-change: opacity;
}

@keyframes blink {
  0%, 100% { opacity: 1; }
  50% { opacity: 0; }
}

.post-text {
  color: rgba(255, 255, 255, 0.95);
  opacity: 0;
  transition: opacity 0.5s cubic-bezier(0.22, 1, 0.36, 1);
}

.post-text.show {
  opacity: 1;
}

/* 발족 이야기 */
.origin-story {
  margin-top: 40px;
  padding: 24px 0;
  border-top: 1px solid rgba(255, 255, 255, 0.2);
  opacity: 0;
  transform: translateY(16px);
  transition: all 0.8s cubic-bezier(0.22, 1, 0.36, 1);
}

.origin-story.show {
  opacity: 1;
  transform: translateY(0);
}

.origin-label {
  font-size: 0.85rem;
  font-weight: 800;
  color: rgba(180, 140, 255, 1);
  letter-spacing: -0.02em;
  margin-bottom: 12px;
}

.origin-text {
  font-size: 0.9rem;
  font-weight: 700;
  line-height: 1.9;
  color: rgba(255, 255, 255, 0.9);
  letter-spacing: -0.02em;
  word-break: keep-all;
}

.scroll-hint {
  margin-top: 32px;
  display: flex;
  align-items: center;
  gap: 6px;
  color: rgba(255, 255, 255, 0.6);
  opacity: 0;
  transition: opacity 0.6s ease;
}

.scroll-hint.show {
  opacity: 0.4;
}

.scroll-hint svg {
  animation: bounce 2s infinite;
}

mark {
  background: linear-gradient(to top, rgba(180, 140, 255, 0.35) 0%, rgba(180, 140, 255, 0.35) 40%, transparent 40%);
  color: #fff;
  padding: 0 2px;
}
@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(4px); }
}

@media (max-width: 640px) {
  .hero {
    padding: 60px 24px 32px;
    padding-left: max(24px, env(safe-area-inset-left, 0px));
    padding-right: max(24px, env(safe-area-inset-right, 0px));
  }
  .pre-line {
    font-size: 1.4rem;
  }
  .main-line {
    font-size: 2.2rem;
  }
  .origin-story {
    margin-top: 32px;
    padding: 20px 0;
  }
  .origin-label {
    font-size: 0.8rem;
  }
  .origin-text {
    font-size: 0.85rem;
  }
}

@media (max-width: 360px) {
  .pre-line {
    font-size: 1.2rem;
  }
  .main-line {
    font-size: 1.8rem;
  }
}

/* 가로모드 대응 */
@media (max-height: 500px) and (orientation: landscape) {
  .hero {
    min-height: auto;
    padding-top: 40px;
    padding-bottom: 24px;
  }
  .origin-story {
    margin-top: 20px;
  }
}
</style>
