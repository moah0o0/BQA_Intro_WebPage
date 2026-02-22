<template>
  <div class="app-root">
    <!-- 스토리텔링 섹션 -->
    <HeroSection />
    <StorySection @openManifesto="openManifestoFull" />

    <!-- 활동과 계획 -->
    <section class="activities-section">
      <!-- 스티키 탭 네비게이션 -->
      <nav class="sticky-nav" ref="stickyNavRef">
        <div class="sticky-nav-inner">
          <button
            v-for="project in projectList"
            :key="project.year"
            :class="{ active: currentProject.year === project.year }"
            @click="selectProject(project)"
          >
            {{ project.year }}년 {{ project.type === 'PlanProject' ? '활동계획' : '활동보고' }}
          </button>
        </div>
      </nav>

      <div class="activities-content">
        <p v-if="currentProject.type === 'DoneProject'" class="plan-intro">
          12·3 내란 이후, 부산지역 윤석열 퇴진광장에서 성소수자들은
          매주 뒤풀이를 하고, 존재를 당당히 드러내는 깃발을 들고서 '무지개존'을 꾸려가며
          서로에게 든든한 곁이 되어 왔습니다.
          하지만 연대의 광장이 닫힌 이후, 마주해야 하는 것은
          차별과 혐오의 세상과 불화해야 하는 삶이었습니다.
          그래서 2025년 2월 28일, 일상 속에서 서로의 용기가 되어주기로 다짐하며,
          부산퀴어행동을 발족하게 되었습니다.
        </p>

        <p v-if="currentProject.type === 'PlanProject'" class="plan-intro muted">
          퀴어 노동의 문제를 해결하기 위한 조사와 요구, 실천에 본격적으로 나서려면
          <mark>함께할 사람들이 먼저 모여야 합니다.</mark>
          일하고 공부하며 살아가는 퀴어들이
          부담 없이 찾아올 수 있도록 <mark>문턱 낮고 재미있는 모임</mark>을 함께 열어봅시다!
        </p>

        <ProjectViewer v-if="currentProject" :data="currentProject" :key="currentProject.title" />
      </div>
    </section>

    <!-- 함께하기 -->
    <section class="section-wrap join-section">
      <div class="section-inner">
        <span class="join-tag">함께하기</span>
        <h2 class="join-title">부산퀴어행동에<br>함께해주세요</h2>

        <div class="join-tabs">
          <button
            class="join-tab-btn"
            :class="{ active: joinType === 'member' }"
            @click="joinType = 'member'"
          >회원가입</button>
          <button
            class="join-tab-btn"
            :class="{ active: joinType === 'supporter' }"
            @click="joinType = 'supporter'"
          >후원하기</button>
        </div>

        <div class="join-content">
          <Transition name="tab" mode="out-in">
            <div v-if="joinType === 'member' && memberText" key="member" v-html="memberText"></div>
            <div v-else-if="joinType === 'supporter'" key="supporter">
              <div v-if="supporterText" v-html="supporterText"></div>
              <div class="donate-card">
                <p class="donate-label">후원계좌</p>
                <p class="donate-account">우체국 100-0003-05297</p>
                <p class="donate-depositor">예금주 : 김찬(부산퀴어행동)</p>
                <button class="donate-copy-btn" @click="copyAccount">계좌 복사</button>
                <p class="donate-note"><em>단체의 경우, 후원확인증 발급 가능</em></p>
              </div>
            </div>
          </Transition>
        </div>
      </div>
    </section>

    <!-- 푸터 -->
    <footer class="site-footer">
      <div class="footer-inner">
        <div class="footer-social">
          <a
            v-for="platform in socialNetworks"
            :key="platform.name"
            :href="platform.url"
            target="_blank"
            rel="noopener noreferrer"
            class="footer-social-link"
          >
            <svg v-if="platform.icon === 'youtube'" class="social-icon" viewBox="0 0 24 24" fill="currentColor">
              <path d="M23.498 6.186a3.016 3.016 0 0 0-2.122-2.136C19.505 3.545 12 3.545 12 3.545s-7.505 0-9.377.505A3.017 3.017 0 0 0 .502 6.186C0 8.07 0 12 0 12s0 3.93.502 5.814a3.016 3.016 0 0 0 2.122 2.136c1.871.505 9.376.505 9.376.505s7.505 0 9.377-.505a3.015 3.015 0 0 0 2.122-2.136C24 15.93 24 12 24 12s0-3.93-.502-5.814zM9.545 15.568V8.432L15.818 12l-6.273 3.568z"/>
            </svg>
            <svg v-if="platform.icon === 'twitter'" class="social-icon" viewBox="0 0 24 24" fill="currentColor">
              <path d="M18.244 2.25h3.308l-7.227 8.26 8.502 11.24H16.17l-5.214-6.817L4.99 21.75H1.68l7.73-8.835L1.254 2.25H8.08l4.713 6.231zm-1.161 17.52h1.833L7.084 4.126H5.117z"/>
            </svg>
            <span>{{ platform.name }}</span>
          </a>
        </div>

        <div class="footer-info">
          <p class="footer-name">부산퀴어행동</p>
          <p class="footer-contact">문의 : busanqueeract@gmail.com</p>
        </div>
      </div>
    </footer>

    <!-- 후원 플로팅 버튼 -->
    <button class="donate-floating-btn" @click="scrollToDonate" title="후원하기">
      👛
    </button>

    <!-- 다짐 모달 -->
    <BaseModal :is-open="isManifestoModalOpen" :title="manifestoModalTitle" @close="isManifestoModalOpen = false">
      <p v-html="manifestoModalContent"></p>
    </BaseModal>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import ProjectViewer from './components/ProjectViewer.vue';
import BaseModal from './components/BaseModal.vue';
import HeroSection from './components/sections/HeroSection.vue';
import StorySection from './components/sections/StorySection.vue';

// 데이터 상태
const manifestoData = ref({});
const projectList = ref([]);
const memberText = ref('');
const supporterText = ref('');
const socialNetworks = ref([]);

// View 상태 관리
const currentProject = ref({});
const joinType = ref('member');

// 모달 로직
const isManifestoModalOpen = ref(false);
const manifestoModalTitle = ref('');
const manifestoModalContent = ref('');

// refs
const stickyNavRef = ref(null);

// 프로젝트 설정 파일에서 설정 불러오기
const loadProjectConfig = async () => {
  try {
    const configRes = await fetch('/data/project_config.json');
    return await configRes.json();
  } catch (error) {
    console.warn('프로젝트 설정 파일이 없습니다. 기본값을 사용합니다.');
    return { projects: [], defaultYear: null };
  }
};

// 데이터 로드
const loadData = async () => {
  try {
    const config = await loadProjectConfig();

    const projectPromises = config.projects && config.projects.length > 0
      ? config.projects.map(year =>
          fetch(`/data/project_${year}.json`)
            .then(res => res.json())
            .catch(err => {
              console.error(`project_${year}.json 로드 실패:`, err);
              return null;
            })
        )
      : [
          fetch('/data/project_2025.json').then(res => res.json()).catch(() => null),
          fetch('/data/project_2026.json').then(res => res.json()).catch(() => null)
        ];

    const [manifestoRes, memberRes, supporterRes, socialRes, ...projectResults] = await Promise.all([
      fetch('/data/manifesto.json'),
      fetch('/data/member.txt'),
      fetch('/data/supporter.txt'),
      fetch('/data/socialnetwork.json'),
      ...projectPromises
    ]);

    manifestoData.value = await manifestoRes.json();
    memberText.value = await memberRes.text();
    supporterText.value = await supporterRes.text();
    const socialData = await socialRes.json();
    socialNetworks.value = socialData.platforms;

    projectList.value = projectResults
      .filter(project => project !== null)
      .sort((a, b) => b.year - a.year);

    if (projectList.value.length > 0) {
      if (config.defaultYear) {
        const defaultProject = projectList.value.find(p => p.year === config.defaultYear);
        currentProject.value = defaultProject || projectList.value[0];
      } else {
        currentProject.value = projectList.value[0];
      }
    }
  } catch (error) {
    console.error('데이터 로드 실패:', error);
  }
};

onMounted(() => {
  loadData();
});

// 탭 클릭 시 스크롤
const selectProject = (project) => {
  currentProject.value = project;
  if (stickyNavRef.value) {
    stickyNavRef.value.scrollIntoView({ behavior: 'smooth' });
  }
};

// MissionCtaSection에서 emit
const openManifestoFull = () => {
  if (manifestoData.value['다짐']) {
    manifestoModalTitle.value = '다짐';
    manifestoModalContent.value = manifestoData.value['다짐'].full_text;
    isManifestoModalOpen.value = true;
  }
};

const copyAccount = () => {
  navigator.clipboard.writeText("우체국 100-0003-05297");
  alert("계좌번호가 복사되었습니다!");
};

const scrollToDonate = () => {
  joinType.value = 'supporter';
  const joinSection = document.querySelector('.join-section');
  if (joinSection) {
    joinSection.scrollIntoView({ behavior: 'smooth' });
  }
};
</script>

<style>
/* CSS Variables & Global Reset */
:root {
  --bg-color: #ffffff;
  --text-color: #000000;
  --accent-color: #8B25FF;
  --border-color: #e5e5e5;
  --shadow-sm: 0 1px 3px rgba(0, 0, 0, 0.08);
  --shadow-md: 0 4px 12px rgba(0, 0, 0, 0.08);
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --spacing-xs: 8px;
  --spacing-sm: 12px;
  --spacing-md: 16px;
  --spacing-lg: 24px;
  --spacing-xl: 32px;
}

html {
  scroll-behavior: smooth;
  -webkit-text-size-adjust: 100%;
  text-size-adjust: 100%;
}

body {
  margin: 0;
  padding: 0;
  background-color: #ffffff;
  color: var(--text-color);
  font-family: "Wanted Sans Variable", "Wanted Sans", -apple-system, BlinkMacSystemFont, system-ui, "Segoe UI", "Apple SD Gothic Neo", "Noto Sans KR", "Malgun Gothic", "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  line-height: 1.5;
  font-weight: 500;
  font-size: 16px;
  letter-spacing: -0.02em;
  word-break: keep-all;
  overscroll-behavior: none;
  -webkit-tap-highlight-color: transparent;
}

/* 터치 타겟 최소 크기 보장 */
button, a, [role="button"] {
  touch-action: manipulation;
}

.app-root {
  width: 100%;
  min-height: 100vh;
  overflow-x: clip;
}

/* ===== 풀 너비 섹션 래퍼 ===== */
.section-wrap {
  width: 100%;
  padding: 80px 28px 90px;
}

.section-inner {
  max-width: 640px;
  margin: 0 auto;
  overflow: hidden;
}

/* 활동과 계획 섹션 */
.activities-section {
  background: #ffffff;
}

.activities-content {
  max-width: 640px;
  margin: 0 auto;
  padding: 40px 28px 90px;
  overflow: hidden;
}

/* 함께하기 섹션 */
.join-section {
  background: #faf8f6;
}

/* ===== 섹션 타이틀 ===== */
.section-title {
  font-size: 1.75rem;
  font-weight: 700;
  border-bottom: 3px solid var(--accent-color);
  display: inline-block;
  margin-bottom: var(--spacing-xl);
  padding-bottom: var(--spacing-xs);
  letter-spacing: -0.02em;
  line-height: 1.3;
}

/* ===== 계획 소개 문구 ===== */
.plan-intro {
  font-size: 0.9375rem;
  font-weight: 500;
  line-height: 1.9;
  color: #555;
  margin: 0 0 var(--spacing-xl) 0;
  letter-spacing: -0.02em;
  word-break: keep-all;
}

.plan-intro.muted {
  color: #000;
  font-size: 1rem;
  font-weight: 600;
}

.plan-intro.muted mark {
  background: linear-gradient(to top, rgba(139, 37, 255, 0.2) 0%, rgba(139, 37, 255, 0.2) 40%, transparent 40%);
  color: inherit;
  padding: 0 2px;
}

/* ===== 스티키 탭 네비게이션 ===== */
.sticky-nav {
  position: sticky;
  top: 0;
  z-index: 100;
  background: rgba(255, 255, 255, 0.92);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border-bottom: 1px solid rgba(0, 0, 0, 0.06);
  padding-left: env(safe-area-inset-left, 0px);
  padding-right: env(safe-area-inset-right, 0px);
}

.sticky-nav-inner {
  display: flex;
  max-width: 640px;
  margin: 0 auto;
  padding: 0;
}

.sticky-nav-inner button {
  flex: 1;
  padding: 16px 12px;
  background: transparent;
  border: none;
  border-bottom: 3px solid transparent;
  color: #888;
  font-size: 0.9375rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.25s ease;
  letter-spacing: -0.01em;
  font-family: inherit;
}

.sticky-nav-inner button:hover {
  color: var(--accent-color);
}

.sticky-nav-inner button.active {
  color: var(--text-color);
  border-bottom-color: var(--accent-color);
  font-weight: 700;
}

/* ===== 함께하기 ===== */
.join-tag {
  display: inline-block;
  background: var(--accent-color);
  color: #fff;
  font-size: 0.8rem;
  font-weight: 700;
  padding: 6px 16px;
  border-radius: 16px;
  margin-bottom: 20px;
  letter-spacing: -0.01em;
}
.join-title {
  font-size: 1.6rem;
  font-weight: 800;
  line-height: 1.5;
  letter-spacing: -0.03em;
  margin: 0 0 28px 0;
  color: var(--text-color);
}
.join-tabs {
  display: flex;
  gap: 12px;
  margin-bottom: 32px;
}
.join-tab-btn {
  padding: 10px 24px;
  border-radius: 24px;
  font-size: 0.95rem;
  font-weight: 700;
  letter-spacing: -0.02em;
  cursor: pointer;
  transition: all 0.25s ease;
  border: 2px solid var(--accent-color);
  color: var(--accent-color);
  background: transparent;
  font-family: inherit;
}
.join-tab-btn.active {
  background: var(--accent-color);
  color: #fff;
  border-color: var(--accent-color);
}
.join-tab-btn:hover:not(.active) {
  background: rgba(139, 37, 255, 0.08);
}
.join-content {
  line-height: 1.8;
  font-weight: 500;
  font-size: 0.9375rem;
  letter-spacing: -0.02em;
  min-height: 200px;
  overflow-wrap: break-word;
  word-break: keep-all;
}
.join-content h3 {
  font-size: 1.25rem;
  font-weight: 800;
  margin: 0 0 var(--spacing-md) 0;
  letter-spacing: -0.02em;
  color: var(--text-color);
}
.join-content h4 {
  font-size: 1rem;
  font-weight: 700;
  margin: 28px 0 var(--spacing-sm) 0;
  letter-spacing: -0.02em;
  color: var(--text-color);
  padding-bottom: 8px;
  border-bottom: 1px solid var(--border-color);
}
.join-content p {
  margin: var(--spacing-sm) 0;
  color: #444;
}
.join-content ul, .join-content ol {
  margin: var(--spacing-sm) 0;
  padding-left: 20px;
  color: #555;
}
.join-content li {
  margin-bottom: var(--spacing-xs);
  line-height: 1.7;
}
.join-content em {
  font-style: normal;
  font-size: 0.85rem;
  color: #888;
}
.action-btn {
  display: inline;
  margin-top: 28px;
  padding: 2px 4px;
  background: linear-gradient(to top, rgba(139, 37, 255, 0.25) 0%, rgba(139, 37, 255, 0.25) 40%, transparent 40%);
  color: var(--text-color);
  font-size: 1rem;
  font-weight: 800;
  letter-spacing: -0.02em;
  text-decoration: none;
  transition: all 0.25s ease;
  box-decoration-break: clone;
  -webkit-box-decoration-break: clone;
}
.action-btn:hover {
  background: linear-gradient(to top, rgba(139, 37, 255, 0.45) 0%, rgba(139, 37, 255, 0.45) 50%, transparent 50%);
  color: var(--text-color);
}
.action-btn:active {
  background: linear-gradient(to top, var(--accent-color) 0%, var(--accent-color) 45%, transparent 45%);
  color: #fff;
}
/* 후원 카드 */
.donate-card {
  margin-top: 28px;
  background: #f8f6ff;
  border: 1px solid rgba(139, 37, 255, 0.15);
  border-radius: var(--radius-lg);
  padding: var(--spacing-lg);
  text-align: center;
}
.donate-label {
  font-size: 0.75rem;
  font-weight: 700;
  color: var(--accent-color);
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin: 0 0 var(--spacing-xs) 0;
}
.donate-account {
  font-size: 1.25rem;
  font-weight: 800;
  color: var(--text-color);
  margin: var(--spacing-xs) 0;
  letter-spacing: -0.02em;
}
.donate-depositor {
  font-size: 0.875rem;
  font-weight: 500;
  color: #666;
  margin: 4px 0 0 0;
}
.donate-copy-btn {
  margin-top: var(--spacing-sm);
  padding: var(--spacing-xs) var(--spacing-lg);
  background: var(--accent-color);
  color: #fff;
  border: none;
  border-radius: var(--radius-sm);
  font-size: 0.8125rem;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.25s ease;
  font-family: inherit;
  letter-spacing: -0.01em;
}
.donate-copy-btn:hover {
  opacity: 0.85;
}
.donate-copy-btn:active {
  transform: scale(0.98);
}
.donate-note {
  margin: var(--spacing-sm) 0 0 0;
}
.donate-note em {
  font-size: 0.8rem;
  color: #999;
}
/* 탭 전환 애니메이션 */
.tab-enter-active,
.tab-leave-active {
  transition: all 0.25s ease;
}
.tab-enter-from {
  opacity: 0;
  transform: translateY(8px);
}
.tab-leave-to {
  opacity: 0;
  transform: translateY(-8px);
}

/* ===== 다크 푸터 ===== */
.site-footer {
  background: #1a1a1a;
  padding: 60px 28px calc(48px + env(safe-area-inset-bottom, 0px));
  text-align: center;
  color: #ffffff;
}

.footer-inner {
  max-width: 480px;
  margin: 0 auto;
}

.footer-social {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 12px;
  margin: 32px 0;
}
.footer-social-link {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 20px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: var(--radius-md);
  color: rgba(255, 255, 255, 0.8);
  text-decoration: none;
  font-size: 0.875rem;
  font-weight: 600;
  transition: all 0.25s ease;
  letter-spacing: -0.01em;
}
.footer-social-link:hover {
  background: rgba(139, 37, 255, 0.3);
  color: #ffffff;
}
.footer-social-link .social-icon {
  width: 18px;
  height: 18px;
}

.footer-info {
  margin-top: var(--spacing-lg);
  text-align: center;
}
.footer-name {
  font-size: 1rem;
  font-weight: 700;
  color: #ffffff;
  margin: 0 0 var(--spacing-xs) 0;
  letter-spacing: -0.02em;
}
.footer-contact {
  font-size: 0.875rem;
  color: rgba(255, 255, 255, 0.6);
  margin: 0;
  font-weight: 500;
  letter-spacing: -0.01em;
}

/* ===== 후원 플로팅 버튼 ===== */
.donate-floating-btn {
  position: fixed;
  bottom: calc(24px + env(safe-area-inset-bottom, 0px));
  right: calc(24px + env(safe-area-inset-right, 0px));
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background: var(--accent-color);
  border: none;
  font-size: 28px;
  cursor: pointer;
  box-shadow: 0 4px 16px rgba(139, 37, 255, 0.3);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  z-index: 1000;
  display: flex;
  align-items: center;
  justify-content: center;
}
.donate-floating-btn:hover {
  transform: scale(1.1);
  box-shadow: 0 6px 20px rgba(139, 37, 255, 0.4);
}
.donate-floating-btn:active {
  transform: scale(0.95);
}

/* ===== 애니메이션 ===== */
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(16px); }
  to { opacity: 1; transform: translateY(0); }
}

/* ===== 스크롤바 ===== */
::-webkit-scrollbar {
  width: 8px;
}
::-webkit-scrollbar-track {
  background: var(--bg-color);
}
::-webkit-scrollbar-thumb {
  background: var(--border-color);
  border-radius: 4px;
}
::-webkit-scrollbar-thumb:hover {
  background: var(--accent-color);
}

/* ===== 모바일 반응형 ===== */
@media (max-width: 640px) {
  body {
    font-size: 15px;
  }
  .section-wrap {
    padding: 60px 24px 70px;
  }
  .section-title {
    font-size: 1.375rem;
    margin-bottom: var(--spacing-lg);
  }
  .sticky-nav-inner button {
    padding: 14px 8px;
    font-size: 0.875rem;
  }
  .activities-content {
    padding: 32px 24px 70px;
  }
  .join-title {
    font-size: 1.375rem;
  }
  .join-tab-btn {
    padding: 10px 20px;
    font-size: 0.875rem;
  }
  .join-content {
    font-size: 0.875rem;
  }
  .join-content h3 {
    font-size: 1.125rem;
  }
  .join-content h4 {
    font-size: 0.9375rem;
  }
  .site-footer {
    padding: 48px 24px 40px;
  }
  .footer-social-link {
    padding: 10px 16px;
    font-size: 0.8125rem;
  }
  .donate-floating-btn {
    bottom: calc(16px + env(safe-area-inset-bottom, 0px));
    right: calc(16px + env(safe-area-inset-right, 0px));
    width: 56px;
    height: 56px;
    font-size: 26px;
  }
}

/* 작은 모바일 (360px 이하) */
@media (max-width: 360px) {
  .section-title {
    font-size: 1.25rem;
  }
  .sticky-nav-inner button {
    font-size: 0.8125rem;
  }
  .join-tab-btn {
    padding: 8px 16px;
    font-size: 0.8125rem;
  }
}

/* 터치 디바이스 최적화 */
@media (hover: none) and (pointer: coarse) {
  .sticky-nav-inner button:hover,
  .join-tab-btn:hover:not(.active) {
    transform: none;
    background: transparent;
  }
  .sticky-nav-inner button:active,
  .join-tab-btn:active {
    transform: scale(0.98);
  }
  .action-btn:hover {
    background: linear-gradient(to top, rgba(139, 37, 255, 0.25) 0%, rgba(139, 37, 255, 0.25) 40%, transparent 40%);
  }
  .action-btn:active {
    background: linear-gradient(to top, rgba(139, 37, 255, 0.45) 0%, rgba(139, 37, 255, 0.45) 50%, transparent 50%);
  }
  .donate-copy-btn:hover {
    opacity: 1;
  }
  .donate-copy-btn:active {
    transform: scale(0.98);
  }
}
</style>
