<template>
  <div class="app-container">
    <header class="main-header">
      <img src="./assets/logo.png" alt="부산퀴어행동 로고" class="logo" />
    </header>

    <main>
     <section class="section social-section">
        <div class="social-buttons">
          <a
            v-for="platform in socialNetworks"
            :key="platform.name"
            :href="platform.url"
            target="_blank"
            rel="noopener noreferrer"
            class="social-btn"
            :style="{ '--platform-color': platform.color }"
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
      </section>
      <section class="section manifesto-section">
        <div class="intro-text">
          <p>
            12·3 내란 이후, 부산지역 윤석열 퇴진광장에서 성소수자들은 매주 뒤풀이를 하고, 존재를 당당히 드러내는 깃발을 들고서 '무지개 깃발존'을 꾸려가며 내란청산을 향한 서로의 든든한 곁이 되어 왔습니다. 하지만 연대의 광장이 닫힌 이후, 마주해야 하는 것은 차별과 혐오의 세상과 불화해야 하는 삶이었습니다. 그래서 일상 속에서 <strong>흩어지지 말고 함께 연대하고 실천하자</strong>며 부산퀴어행동을 발족하게 되었습니다.
          </p>
        </div>

        <div class="manifesto-list" v-if="Object.keys(manifestoData).length > 0">
          <div class="mf-item" @click="openManifestoModal('다짐', manifestoData['다짐'])">
            <span class="mf-label">2026년~2028년 부산퀴어행동의 다짐</span>
            <p>{{ manifestoData['다짐'].summary }}</p>
            <span class="mf-arrow">→</span>
          </div>
        </div>
      </section>

      <hr class="divider">

      <section class="section project-section">
        <h2 class="section-title">활동과 계획</h2>
        
        <div class="tab-buttons">
          <button
            v-for="project in projectList"
            :key="project.year"
            :class="{ active: currentProject.year === project.year }"
            @click="currentProject = project"
          >
            {{ project.year }}년 {{ project.type === 'PlanProject' ? '계획' : '결과' }}
          </button>
        </div>

        <ProjectViewer v-if="currentProject" :data="currentProject" :key="currentProject.title" />
      </section>
      
      <hr class="divider"></hr>

      <section class="section join-section">
        <h2 class="section-title">함께하기</h2>
        
        <div class="join-tabs">
          <div class="join-option" :class="{ active: joinType === 'member' }" @click="joinType = 'member'">
            <span>회원으로</span>
          </div>
          <div class="join-option" :class="{ active: joinType === 'supporter' }" @click="joinType = 'supporter'">
            <span>후원자로</span>
          </div>
        </div>

        <div class="join-content">
          <div v-if="joinType === 'member' && memberText" class="fade-in" v-html="memberText"></div>
          <div v-if="joinType === 'supporter' && supporterText" class="fade-in" v-html="supporterText"></div>
        </div>
      </section>

 
      <footer class="footer">
        <div class="bank-info" ref="bankInfoSection">
          <p class="bank-label">후원계좌</p>
          <p class="bank-number">
            우체국 100-0003-05297<br>
            <span class="depositor">예금주 : 김찬(부산퀴어행동)</span>
          </p>
          <button class="copy-btn" @click="copyAccount">계좌 복사</button>
        </div>
        <div class="footer-info">
          <p class="footer-name">부산퀴어행동</p>
          <p class="footer-contact">문의 : busanqueeract@gmail.com</p>
        </div>
      </footer>
    </main>

    <!-- 후원 플로팅 버튼 -->
    <button class="donate-floating-btn" @click="scrollToDonate" title="후원하기">
      👛
    </button>

    <BaseModal :is-open="isManifestoModalOpen" :title="manifestoModalTitle" @close="isManifestoModalOpen = false">
      <p v-html="manifestoModalContent"></p>
    </BaseModal>

  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import ProjectViewer from './components/ProjectViewer.vue';
import BaseModal from './components/BaseModal.vue';

// 데이터 상태
const manifestoData = ref({});
const projectList = ref([]); // 모든 프로젝트 데이터
const memberText = ref('');
const supporterText = ref('');
const socialNetworks = ref([]);

// View 상태 관리
const currentProject = ref({}); // 기본값: 로드 후 설정
const joinType = ref('member'); // member or supporter

// 모달 로직
const isManifestoModalOpen = ref(false);
const manifestoModalTitle = ref('');
const manifestoModalContent = ref('');

// 후원 섹션 ref
const bankInfoSection = ref(null);

// 프로젝트 설정 파일에서 설정 불러오기
const loadProjectConfig = async () => {
  try {
    const configRes = await fetch('/data/project_config.json');
    return await configRes.json();
  } catch (error) {
    console.warn('프로젝트 설정 파일이 없습니다. 기본값을 사용합니다.');
    // 기본값: 가장 최신 연도의 PlanProject를 디폴트로 설정
    return { projects: [], defaultYear: null };
  }
};

// 데이터 로드
const loadData = async () => {
  try {
    // 프로젝트 설정 불러오기
    const config = await loadProjectConfig();

    // 프로젝트 파일 동적 로드
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
          // 기본값: 2025, 2026 프로젝트 로드
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

    // 프로젝트 데이터 정리 (null 제거 및 연도순 정렬)
    projectList.value = projectResults
      .filter(project => project !== null)
      .sort((a, b) => b.year - a.year); // 최신 연도가 먼저 오도록 정렬

    // 기본 프로젝트 설정
    if (projectList.value.length > 0) {
      if (config.defaultYear) {
        // 설정된 디폴트 연도가 있으면 해당 프로젝트 찾기
        const defaultProject = projectList.value.find(p => p.year === config.defaultYear);
        currentProject.value = defaultProject || projectList.value[0];
      } else {
        // 디폴트 설정이 없으면 가장 최신 연도 선택
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

const openManifestoModal = (title, item) => {
  manifestoModalTitle.value = title;
  manifestoModalContent.value = item.full_text;
  isManifestoModalOpen.value = true;
};

const copyAccount = () => {
  navigator.clipboard.writeText("우체국 100-0003-05297");
  alert("계좌번호가 복사되었습니다!");
};

const scrollToDonate = () => {
  if (bankInfoSection.value) {
    bankInfoSection.value.scrollIntoView({ behavior: 'smooth', block: 'center' });
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

body {
  margin: 0;
  padding: 0;
  background-color: #f5f5f5;
  color: var(--text-color);
  font-family: "Wanted Sans Variable", "Wanted Sans", -apple-system, BlinkMacSystemFont, system-ui, "Segoe UI", "Apple SD Gothic Neo", "Noto Sans KR", "Malgun Gothic", "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  line-height: 1.5;
  font-weight: 500;
  font-size: 16px;
  letter-spacing: -0.02em;
}

.app-container {
  max-width: 640px;
  margin: 0 auto;
  min-height: 100vh;
  padding: var(--spacing-xl) var(--spacing-lg);
  background: var(--bg-color);
  box-shadow: 0 0 40px rgba(0, 0, 0, 0.08);
}

@media (max-width: 640px) {
  body {
    background-color: var(--bg-color);
    font-size: 15px;
  }
  .app-container {
    box-shadow: none;
    padding: var(--spacing-md);
    padding-bottom: 80px; /* 플로팅 버튼 공간 확보 */
  }
  .logo {
    width: 140px;
    height: 140px;
  }
  .main-header {
    padding: var(--spacing-lg) 0 var(--spacing-xl) 0;
  }
  .section {
    margin-bottom: 48px;
  }
  .section-title {
    font-size: 1.375rem;
    margin-bottom: var(--spacing-lg);
  }
  .divider {
    margin: 40px 0;
  }
  /* 소셜 버튼 모바일 최적화 */
  .social-buttons {
    flex-direction: column;
    gap: var(--spacing-sm);
  }
  .social-btn {
    min-width: 100%;
    padding: var(--spacing-md);
    min-height: 48px; /* 터치 타겟 최소 크기 */
  }
  .social-icon {
    width: 22px;
    height: 22px;
  }
  /* 다짐 카드 모바일 */
  .mf-item {
    padding: var(--spacing-md);
  }
  .mf-item p {
    font-size: 0.875rem;
  }
  .intro-text {
    font-size: 0.875rem;
    line-height: 1.8;
  }
  /* 탭 버튼 모바일 최적화 */
  .tab-buttons {
    gap: var(--spacing-xs);
  }
  .tab-buttons button {
    padding: var(--spacing-sm) var(--spacing-xs);
    font-size: 0.8125rem;
    min-height: 44px; /* 터치 타겟 */
  }
  /* 함께하기 탭 모바일 */
  .join-option {
    padding: var(--spacing-md) var(--spacing-sm);
    min-height: 48px;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .join-content {
    padding: var(--spacing-lg);
    font-size: 0.875rem;
  }
  .join-content h3 {
    font-size: 1.125rem;
  }
  .join-content h4 {
    font-size: 1rem;
  }
  /* 푸터 모바일 */
  .footer {
    padding: 32px 0;
  }
  .bank-info {
    padding: var(--spacing-md);
  }
  .bank-number {
    font-size: 1rem;
  }
  .copy-btn {
    min-height: 44px;
    padding: var(--spacing-sm) var(--spacing-lg);
  }
}

/* Headers */
.main-header {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: var(--spacing-xl) 0 48px 0;
}
.logo {
  width: 180px;
  height: 180px;
  object-fit: contain;
  border-radius: var(--radius-md);
  box-shadow: var(--shadow-sm);
}
.main-header h1 {
  font-size: 2rem;
  line-height: 1.3;
  margin: 0 0 var(--spacing-sm) 0;
  font-weight: 700;
  letter-spacing: -0.03em;
}
.highlight { color: var(--accent-color); }
.slogan {
  color: var(--text-color);
  font-size: 0.875rem;
  margin: 0;
  font-weight: 500;
  letter-spacing: -0.01em;
}

/* Section Common */
.section { margin-bottom: 80px; }
.join-section { margin-bottom: 24px; }
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
.divider {
  border: 0;
  height: 1px;
  background: var(--border-color);
  margin: 64px 0;
  opacity: 0.8;
}

/* Manifesto */
.intro-text {
  line-height: 1.7;
  color: var(--text-color);
  margin-bottom: var(--spacing-xl);
  font-size: 0.9375rem;
  letter-spacing: -0.02em;
  font-weight: 500;
}
.intro-text strong {
  color: var(--text-color);
  font-weight: 700;
  text-decoration: underline;
  text-decoration-color: var(--accent-color);
  text-decoration-thickness: 2px;
  text-underline-offset: 3px;
}

.mf-item {
  background: var(--bg-color);
  padding: var(--spacing-md) var(--spacing-lg);
  border-radius: var(--radius-md);
  margin-bottom: var(--spacing-md);
  cursor: pointer;
  position: relative;
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  border: 1px solid var(--border-color);
}
.mf-item:hover {
  transform: translateY(-2px);
  border-color: var(--accent-color);
}
.mf-item:active {
  transform: translateY(-1px);
  box-shadow: var(--shadow-sm);
}
.mf-label {
  display: block;
  font-size: 0.6875rem;
  color: var(--accent-color);
  font-weight: 700;
  margin-bottom: var(--spacing-xs);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}
.mf-item p {
  margin: 0;
  font-size: 0.9375rem;
  color: var(--text-color);
  padding-right: 28px;
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
  line-height: 1.5;
  font-weight: 500;
  letter-spacing: -0.02em;
}
.mf-arrow {
  position: absolute;
  right: var(--spacing-lg);
  top: 50%;
  transform: translateY(-50%);
  color: var(--text-color);
  font-size: 1.2rem;
  transition: all 0.25s ease;
  font-weight: 300;
}
.mf-item:hover .mf-arrow {
  color: var(--accent-color);
  transform: translateY(-50%) translateX(2px);
}

/* Project Tabs */
.tab-buttons {
  display: flex;
  gap: var(--spacing-sm);
  margin-bottom: var(--spacing-xl);
}
.tab-buttons button {
  flex: 1;
  padding: var(--spacing-md);
  background: transparent;
  border: 1.5px solid var(--border-color);
  color: var(--text-color);
  border-radius: 30px;
  font-size: 0.875rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  letter-spacing: -0.01em;
}
.tab-buttons button:hover {
  border-color: var(--accent-color);
  color: var(--accent-color);
}
.tab-buttons button.active {
  background: var(--accent-color);
  color: #fff;
  border-color: var(--accent-color);
  font-weight: 700;
}

/* Social Network Section */
.social-section {
  margin-bottom: var(--spacing-xl);
}

.social-buttons {
  display: flex;
  gap: var(--spacing-md);
  flex-wrap: wrap;
}
.social-btn {
  flex: 1;
  min-width: 140px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: var(--spacing-sm);
  padding: var(--spacing-md) var(--spacing-lg);
  background: var(--bg-color);
  border: 1.5px solid var(--border-color);
  border-radius: var(--radius-md);
  color: var(--text-color);
  text-decoration: none;
  font-weight: 600;
  font-size: 0.9375rem;
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  letter-spacing: -0.01em;
}
.social-btn:hover {
  border-color: var(--platform-color);
  color: var(--platform-color);
  transform: translateY(-2px);
  box-shadow: var(--shadow-md);
}
.social-btn:active {
  transform: translateY(-1px);
}
.social-icon {
  width: 20px;
  height: 20px;
  transition: all 0.25s ease;
}
.social-btn:hover .social-icon {
  color: var(--platform-color);
}

/* Join Section */
.join-tabs {
  display: flex;
  border-bottom: 2px solid var(--border-color);
  margin-bottom: var(--spacing-lg);
}
.join-option {
  flex: 1;
  text-align: center;
  padding: var(--spacing-md);
  cursor: pointer;
  color: var(--text-color);
  position: relative;
  transition: all 0.25s ease;
  font-weight: 400;
}
.join-option:hover {
  color: var(--accent-color);
}
.join-option.active {
  color: var(--text-color);
  font-weight: 700;
}
.join-option.active::after {
  content: '';
  position: absolute;
  bottom: -2px;
  left: 0;
  width: 100%;
  height: 3px;
  background: var(--accent-color);
  border-radius: 2px 2px 0 0;
}
.join-content {
  background: var(--bg-color);
  padding: var(--spacing-xl);
  border-radius: var(--radius-md);
  border: 1px solid var(--border-color);
  line-height: 1.7;
  font-weight: 500;
  font-size: 0.9375rem;
  letter-spacing: -0.02em;
}
.join-content h3 {
  font-size: 1.25rem;
  font-weight: 700;
  margin: 0 0 var(--spacing-md) 0;
  letter-spacing: -0.02em;
  color: var(--text-color);
}
.join-content h4 {
  font-size: 1.0625rem;
  font-weight: 700;
  margin: var(--spacing-lg) 0 var(--spacing-sm) 0;
  letter-spacing: -0.02em;
  color: var(--text-color);
}
.join-content p {
  margin: var(--spacing-sm) 0;
  color: var(--text-color);
}
.join-content ul, .join-content ol {
  margin: var(--spacing-sm) 0;
  padding-left: 24px;
  color: var(--text-color);
}
.join-content li {
  margin-bottom: var(--spacing-xs);
}
.join-content strong {
  font-weight: 700;
  color: var(--text-color);
}
.join-content em {
  font-style: italic;
  color: var(--text-color);
  opacity: 0.8;
}
.action-btn {
  display: inline;
  margin-top: var(--spacing-lg);
  color: var(--text-color);
  text-decoration: none;
  font-weight: 700;
  font-size: 1rem;
  letter-spacing: -0.01em;
  position: relative;
  background: linear-gradient(to top, rgba(139, 37, 255, 0.3) 0%, rgba(139, 37, 255, 0.3) 40%, transparent 40%);
  transition: all 0.25s ease;
  padding: 2px 4px;
  box-decoration-break: clone;
  -webkit-box-decoration-break: clone;
}
.action-btn:hover {
  background: linear-gradient(to top, rgba(139, 37, 255, 0.5) 0%, rgba(139, 37, 255, 0.5) 50%, transparent 50%);
  color: var(--text-color);
}
.action-btn:active {
  background: linear-gradient(to top, var(--accent-color) 0%, var(--accent-color) 45%, transparent 45%);
}
.fade-in {
  animation: fadeIn 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

/* Footer */
.footer {
  text-align: center;
  padding: 48px 0;
  color: var(--text-color);
  font-size: 0.9rem;
  font-weight: 400;
}
.bank-info {
  background: var(--bg-color);
  padding: var(--spacing-lg);
  border-radius: var(--radius-md);
  display: inline-block;
  margin-bottom: var(--spacing-lg);
  border: 1px solid var(--border-color);
  width: 100%;
  box-sizing: border-box;
}
.bank-label {
  color: var(--accent-color);
  font-size: 0.6875rem;
  font-weight: 700;
  margin-bottom: var(--spacing-xs);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}
.bank-number {
  font-size: 1.125rem;
  color: var(--text-color);
  font-weight: 700;
  margin: var(--spacing-sm) 0;
  letter-spacing: -0.02em;
}
.depositor {
  display: block;
  font-size: 0.875rem;
  font-weight: 500;
  color: var(--text-color);
  margin-top: var(--spacing-xs);
  letter-spacing: -0.01em;
}
.copy-btn {
  background: transparent;
  border: 1.5px solid var(--border-color);
  color: var(--text-color);
  padding: var(--spacing-xs) var(--spacing-md);
  border-radius: var(--radius-sm);
  font-size: 0.8125rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.25s ease;
  margin-top: var(--spacing-sm);
  letter-spacing: -0.01em;
}
.copy-btn:hover {
  border-color: var(--accent-color);
  color: var(--accent-color);
}
.copy-btn:active {
  transform: scale(0.98);
}
.footer-info {
  margin-top: var(--spacing-lg);
  text-align: center;
}
.footer-name {
  font-size: 1rem;
  font-weight: 700;
  color: var(--text-color);
  margin: 0 0 var(--spacing-xs) 0;
  letter-spacing: -0.02em;
}
.footer-contact {
  font-size: 0.875rem;
  color: var(--text-color);
  margin: 0;
  font-weight: 500;
  letter-spacing: -0.01em;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(16px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* Smooth Scrollbar */
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

/* 후원 플로팅 버튼 */
.donate-floating-btn {
  position: fixed;
  bottom: 24px;
  right: 24px;
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

@media (max-width: 640px) {
  .donate-floating-btn {
    bottom: 16px;
    right: 16px;
    width: 56px;
    height: 56px;
    font-size: 26px;
  }
}

/* 작은 모바일 화면 추가 최적화 (360px 이하) */
@media (max-width: 360px) {
  .app-container {
    padding: var(--spacing-sm);
    padding-bottom: 80px;
  }
  .logo {
    width: 120px;
    height: 120px;
  }
  .section-title {
    font-size: 1.25rem;
  }
  .tab-buttons {
    flex-wrap: wrap;
  }
  .tab-buttons button {
    flex: 1 1 45%;
    font-size: 0.75rem;
  }
  .mf-label {
    font-size: 0.625rem;
  }
  .bank-number {
    font-size: 0.9375rem;
  }
  .join-content {
    padding: var(--spacing-md);
  }
}

/* 터치 디바이스 최적화 */
@media (hover: none) and (pointer: coarse) {
  .mf-item:hover,
  .social-btn:hover,
  .tab-buttons button:hover,
  .join-option:hover,
  .copy-btn:hover {
    transform: none;
  }
  .mf-item:active {
    transform: scale(0.98);
    background: rgba(139, 37, 255, 0.03);
  }
  .social-btn:active {
    transform: scale(0.98);
    background: rgba(139, 37, 255, 0.05);
  }
  .tab-buttons button:active,
  .join-option:active {
    transform: scale(0.98);
  }
  .copy-btn:active {
    transform: scale(0.95);
    background: rgba(139, 37, 255, 0.1);
  }
}
</style>