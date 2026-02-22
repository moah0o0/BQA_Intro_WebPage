<template>
  <div class="project-viewer">
    <!-- 2025년 결과 (DoneProject) - 인스타 피드 -->
    <div v-if="data.type === 'DoneProject'" class="view-done">
      <div v-if="loadingPosts" class="loading-indicator">불러오는 중...</div>
      <div v-else class="feed">
        <article
          v-for="post in posts"
          :key="post.id"
          class="feed-card"
          @click="openPost(post)"
        >
          <div class="feed-header">
            <div class="feed-avatar">{{ post.order }}</div>
            <div class="feed-header-text">
              <span class="feed-username">{{ post.subtitle }}</span>
              <span class="feed-label">부산퀴어행동 · 2025</span>
            </div>
          </div>
          <div v-if="post.photos && post.photos.length" class="feed-cover">
            <img :src="post.photos[0]" :alt="post.subtitle" />
            <span v-if="post.photos.length > 1" class="feed-photo-count">{{ post.photos.length }}장</span>
          </div>
          <div class="feed-caption">
            <p class="feed-desc"><strong>{{ post.subtitle }}</strong> {{ post.description }}</p>
            <span class="feed-more">더보기</span>
          </div>
        </article>
      </div>
    </div>

    <!-- 2026년 계획 (PlanProject) - 카드 UI -->
    <div v-if="data.type === 'PlanProject'" class="view-plan">
      <div v-if="loadingCards" class="loading-indicator">불러오는 중...</div>
      <template v-if="data.cards && data.cards.length">
        <!-- featured 카드 -->
        <div class="plan-cards">
          <div
            v-for="card in data.cards.filter(c => c.featured)"
            :key="card.id"
            class="plan-card featured"
            @click="openCardModal(card)"
          >
            <span class="featured-badge">주요활동</span>
            <div class="plan-card-icon">{{ card.icon }}</div>
            <h3 class="plan-card-title">{{ card.title }}</h3>
            <p v-if="card.subtitle" class="plan-card-subtitle">{{ card.subtitle }}</p>
            <p class="plan-card-body">{{ card.body }}</p>
            <span class="plan-card-cta">{{ card.cta }}</span>
          </div>
        </div>
        <p class="featured-note">
          세상을 바꾸기 위한 <mark>사회운동에 재미있는 방식으로 함께</mark>하고,
          우리 스스로의 고민과 기획을 담아 행동으로 옮겨봅시다.
          이를 통해 <mark>세상을 바꾸는 우리 퀴어들의 힘</mark>을 키워봐요!
        </p>
        <!-- 나머지 카드 -->
        <div class="plan-cards">
          <div
            v-for="card in data.cards.filter(c => !c.featured)"
            :key="card.id"
            class="plan-card"
            :class="{ 'half': card.layout === 'half' }"
            @click="openCardModal(card)"
          >
            <div class="plan-card-icon">{{ card.icon }}</div>
            <h3 class="plan-card-title">{{ card.title }}</h3>
            <p v-if="card.subtitle" class="plan-card-subtitle">{{ card.subtitle }}</p>
            <p class="plan-card-body">{{ card.body }}</p>
            <span class="plan-card-cta">{{ card.cta }}</span>
          </div>
        </div>
      </template>
      <!-- 회원이 여는 소모임 -->
      <div v-if="data.clubs && data.clubs.length" class="clubs-section">
        <h4 class="extras-heading">회원이 여는 소모임</h4>
        <div
          v-for="club in data.clubs"
          :key="club.id"
          class="extra-item"
          :class="{ open: openExtras[club.id] }"
        >
          <button class="extra-title" @click.stop="toggleExtra(club.id)">
            <span><strong>{{ club.name }}</strong> <span class="club-label">{{ club.label }}</span></span>
            <span class="extra-arrow">&#8250;</span>
          </button>
          <div v-if="openExtras[club.id]" class="extra-body">
            <p>{{ club.description }}</p>
          </div>
        </div>
      </div>

      <!-- 부가 활동 아코디언 -->
      <div v-if="data.extras && data.extras.length" class="extras-section">
        <h4 class="extras-heading">함께하는 활동</h4>
        <div
          v-for="extra in data.extras"
          :key="extra.id"
          class="extra-item"
          :class="{ open: openExtras[extra.id] }"
        >
          <button class="extra-title" @click.stop="toggleExtra(extra.id)">
            <span>{{ extra.title }}</span>
            <span class="extra-arrow">&#8250;</span>
          </button>
          <div v-if="openExtras[extra.id]" class="extra-body">
            <p>{{ extra.description }}</p>
          </div>
        </div>
      </div>

      <!-- 임원진 다짐 -->
      <div v-if="data.leaders && data.leaders.length" class="leaders-section">
        <h4 class="extras-heading">2026 부산퀴어행동을 이끌 사람들</h4>
        <div class="leaders-grid">
          <div v-for="leader in data.leaders" :key="leader.id" class="leader-card">
            <img :src="leader.photo" :alt="leader.name" class="leader-photo" />
            <div class="leader-info">
              <span class="leader-name">{{ leader.name }}</span>
              <span class="leader-role">{{ leader.role }}</span>
            </div>
            <p class="leader-message">{{ leader.message }}</p>
          </div>
        </div>
      </div>
    </div>

    <!-- PlanProject 카드 모달 -->
    <BaseModal :is-open="showCardModal" :title="cardModalData.title" @close="showCardModal = false">
      <div v-if="cardModalData.htmlContent" class="post-body" v-html="cardModalData.htmlContent"></div>
    </BaseModal>

    <!-- DoneProject 게시글 모달 -->
    <BaseModal :is-open="showPostModal" :title="postModalData.subtitle" @close="showPostModal = false">
      <p class="post-description">{{ postModalData.description }}</p>

      <div v-if="postModalData.photos && postModalData.photos.length" class="post-gallery">
        <div class="post-gallery-main">
          <img :src="postModalData.photos[currentPostPhoto]" alt="활동 사진" />
          <button v-if="postModalData.photos.length > 1" class="photo-nav prev" @click="prevPostPhoto">&lt;</button>
          <button v-if="postModalData.photos.length > 1" class="photo-nav next" @click="nextPostPhoto">&gt;</button>
        </div>
        <div v-if="postModalData.photos.length > 1" class="post-gallery-thumbs">
          <img
            v-for="(photo, pIdx) in postModalData.photos"
            :key="pIdx"
            :src="photo"
            :class="{ active: currentPostPhoto === pIdx }"
            @click="currentPostPhoto = pIdx"
            alt="썸네일"
          />
        </div>
      </div>

      <div v-if="postModalData.htmlContent" class="post-body" v-html="postModalData.htmlContent"></div>

      <div v-if="postModalData.videos && postModalData.videos.length" class="videos-section">
        <h4 class="videos-title">영상</h4>
        <div v-for="(video, vIdx) in postModalData.videos" :key="vIdx" class="video-wrapper">
          <iframe
            :src="video"
            frameborder="0"
            allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
            allowfullscreen
          ></iframe>
        </div>
      </div>
    </BaseModal>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue';
import { marked } from 'marked';
import BaseModal from './BaseModal.vue';

const props = defineProps({
  data: Object
});

// PlanProject 카드 모달
const showCardModal = ref(false);
const cardModalData = ref({ title: '', htmlContent: '' });
const cardMarkdownCache = ref({});
const loadingCards = ref(false);

// PlanProject 부가활동 아코디언
const openExtras = ref({});
const toggleExtra = (id) => {
  openExtras.value[id] = !openExtras.value[id];
};

// DoneProject 게시판
const posts = ref([]);
const loadingPosts = ref(false);

// DoneProject 게시글 모달
const showPostModal = ref(false);
const postModalData = ref({ subtitle: '', description: '', photos: [], videos: [], htmlContent: '' });
const currentPostPhoto = ref(0);

// frontmatter 파싱
const parseFrontmatter = (raw) => {
  const match = raw.match(/^---\n([\s\S]*?)\n---\n?([\s\S]*)$/);
  if (!match) return { meta: {}, content: raw };

  const frontmatter = match[1];
  const content = match[2];
  const meta = {};

  let currentKey = null;
  let currentList = null;

  for (const line of frontmatter.split('\n')) {
    if (line.match(/^\s+-\s+/)) {
      const val = line.replace(/^\s+-\s+/, '').trim();
      if (currentList && currentKey) {
        currentList.push(val);
      }
      continue;
    }

    const kvMatch = line.match(/^(\w+):\s*(.*)$/);
    if (kvMatch) {
      const key = kvMatch[1];
      const value = kvMatch[2].trim();

      if (value === '') {
        currentKey = key;
        currentList = [];
        meta[key] = currentList;
      } else {
        meta[key] = value.replace(/^['"]|['"]$/g, '');
        currentKey = null;
        currentList = null;
      }
    }
  }

  return { meta, content };
};

// 마크다운 파일들 로드
const loadPosts = async () => {
  if (props.data.type !== 'DoneProject' || !props.data.posts) return;

  loadingPosts.value = true;
  const postsDir = props.data.postsDir || '/data/posts/' + props.data.year + '/';

  try {
    const results = await Promise.all(
      props.data.posts.map(async (filename) => {
        const res = await fetch(postsDir + filename);
        const raw = await res.text();
        const { meta, content } = parseFrontmatter(raw);
        return {
          ...meta,
          order: parseInt(meta.order) || 0,
          photos: meta.photos || [],
          videos: meta.videos || [],
          markdownContent: content.trim()
        };
      })
    );
    posts.value = results.sort((a, b) => a.order - b.order);
  } catch (error) {
    console.error('게시글 로드 실패:', error);
  } finally {
    loadingPosts.value = false;
  }
};

// 게시글 열기
const openPost = (post) => {
  currentPostPhoto.value = 0;
  postModalData.value = {
    subtitle: post.subtitle || '',
    description: post.description || '',
    photos: post.photos || [],
    videos: post.videos || [],
    htmlContent: post.markdownContent ? marked(post.markdownContent) : ''
  };
  showPostModal.value = true;
};

// 사진 네비게이션
const nextPostPhoto = () => {
  const total = postModalData.value.photos.length;
  currentPostPhoto.value = (currentPostPhoto.value + 1) % total;
};

const prevPostPhoto = () => {
  const total = postModalData.value.photos.length;
  currentPostPhoto.value = currentPostPhoto.value === 0 ? total - 1 : currentPostPhoto.value - 1;
};

// PlanProject 카드 모달
const openCardModal = async (card) => {
  if (!card.detailFile) return;

  cardModalData.value = { title: card.title, htmlContent: '' };
  showCardModal.value = true;

  // 캐시에 있으면 바로 사용
  if (cardMarkdownCache.value[card.id]) {
    cardModalData.value.htmlContent = cardMarkdownCache.value[card.id];
    return;
  }

  try {
    const postsDir = props.data.postsDir || '/data/posts/' + props.data.year + '/';
    const res = await fetch(postsDir + card.detailFile);
    const raw = await res.text();
    const html = marked(raw);
    cardMarkdownCache.value[card.id] = html;
    cardModalData.value.htmlContent = html;
  } catch (error) {
    console.error('카드 상세 로드 실패:', error);
    cardModalData.value.htmlContent = '<p>내용을 불러올 수 없습니다.</p>';
  }
};

watch(() => props.data, () => {
  if (props.data.type === 'DoneProject') {
    loadPosts();
  }
}, { immediate: true });
</script>

<style scoped>
.project-viewer {
  animation: fadeIn 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

/* ===== 인스타 피드 스타일 ===== */
.feed {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.feed-card {
  background: var(--bg-color, #ffffff);
  border: 1px solid var(--border-color, #e5e5e5);
  border-radius: 8px;
  overflow: hidden;
  cursor: pointer;
  transition: all 0.2s ease;
}

.feed-card:hover {
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.08);
}

.feed-card:active {
  transform: scale(0.99);
}

/* 프로필 헤더 */
.feed-header {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px 14px;
}

.feed-avatar {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--accent-color, #8B25FF), #c084fc);
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.8125rem;
  font-weight: 700;
  flex-shrink: 0;
}

.feed-header-text {
  display: flex;
  flex-direction: column;
  min-width: 0;
}

.feed-username {
  font-size: 0.9375rem;
  font-weight: 700;
  color: var(--text-color, #000);
  letter-spacing: -0.02em;
  line-height: 1.3;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.feed-label {
  font-size: 0.8125rem;
  color: #999;
  font-weight: 400;
  letter-spacing: -0.01em;
}

/* 커버 이미지 */
.feed-cover {
  position: relative;
  width: 100%;
  aspect-ratio: 2 / 1;
  overflow: hidden;
  background: #f0f0f0;
}

.feed-cover img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.feed-photo-count {
  position: absolute;
  top: 12px;
  right: 12px;
  background: rgba(0, 0, 0, 0.65);
  color: white;
  font-size: 0.75rem;
  font-weight: 600;
  padding: 3px 8px;
  border-radius: 12px;
  backdrop-filter: blur(4px);
}

/* 캡션 */
.feed-caption {
  padding: 12px 14px 16px;
}

.feed-desc {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  margin: 0;
  font-size: 0.9375rem;
  color: var(--text-color, #000);
  line-height: 1.6;
  font-weight: 400;
  letter-spacing: -0.02em;
  word-break: keep-all;
}

.feed-desc strong {
  font-weight: 700;
  margin-right: 4px;
}

.feed-more {
  display: inline-block;
  margin-top: 4px;
  font-size: 0.9375rem;
  color: #999;
  font-weight: 400;
}

.loading-indicator {
  text-align: center;
  padding: var(--spacing-xl, 32px);
  color: #888;
  font-size: 0.875rem;
}

/* ===== 게시글 모달 내부 스타일 ===== */
.post-description {
  margin: 0 0 var(--spacing-lg, 24px) 0;
  font-size: 0.9375rem;
  color: var(--text-color, #000);
  line-height: 1.7;
  font-weight: 500;
  letter-spacing: -0.02em;
  white-space: pre-line;
}

.post-gallery {
  margin-bottom: var(--spacing-lg, 24px);
}

.post-gallery-main {
  position: relative;
  background: #f0f0f0;
  border-radius: var(--radius-md, 8px);
  overflow: hidden;
}

.post-gallery-main img {
  width: 100%;
  height: auto;
  display: block;
  max-height: 400px;
  object-fit: contain;
}

.post-gallery-thumbs {
  display: flex;
  gap: 6px;
  margin-top: 8px;
  overflow-x: auto;
  padding-bottom: 4px;
}

.post-gallery-thumbs img {
  width: 56px;
  height: 56px;
  object-fit: cover;
  border-radius: 6px;
  cursor: pointer;
  border: 2px solid transparent;
  opacity: 0.6;
  transition: all 0.2s ease;
  flex-shrink: 0;
}

.post-gallery-thumbs img.active {
  border-color: var(--accent-color, #8B25FF);
  opacity: 1;
}

.post-gallery-thumbs img:hover {
  opacity: 0.9;
}

.post-body {
  line-height: 1.8;
  font-size: 1rem;
  color: var(--text-color, #000);
  font-weight: 500;
  letter-spacing: -0.02em;
  white-space: normal;
  word-break: keep-all;
  overflow-wrap: break-word;
}

.post-body :deep(h2) {
  font-size: 1.1875rem;
  font-weight: 700;
  color: var(--text-color, #000);
  margin: var(--spacing-xl, 28px) 0 var(--spacing-sm, 12px) 0;
  padding-bottom: 8px;
  border-bottom: 1px solid var(--border-color, #e5e5e5);
  letter-spacing: -0.02em;
}

.post-body :deep(h2:first-child) {
  margin-top: 0;
}

.post-body :deep(p) {
  margin: 0 0 var(--spacing-sm, 12px) 0;
  line-height: 1.8;
}

.post-body :deep(strong) {
  font-weight: 700;
  color: var(--text-color, #000);
}

.post-body :deep(blockquote) {
  margin: var(--spacing-md, 16px) 0;
  padding: var(--spacing-md, 16px) var(--spacing-lg, 24px);
  background: #f8f8f8;
  border-left: 4px solid var(--accent-color, #8B25FF);
  border-radius: 4px;
  font-style: italic;
  color: #333;
  font-size: 0.9375rem;
  line-height: 1.8;
}

.post-body :deep(blockquote p) {
  margin: 0;
}

.post-body :deep(ul),
.post-body :deep(ol) {
  margin: var(--spacing-sm, 12px) 0;
  padding-left: 24px;
}

.post-body :deep(li) {
  margin-bottom: 8px;
  line-height: 1.8;
}

.post-body :deep(ul li)::marker {
  color: var(--accent-color, #8B25FF);
}

.post-body :deep(a) {
  color: var(--accent-color, #8B25FF);
  text-decoration: none;
  font-weight: 600;
  transition: all 0.2s ease;
}

.post-body :deep(a:hover) {
  text-decoration: underline;
  opacity: 0.8;
}

/* ===== 2026 카드 스타일 ===== */
.plan-cards {
  display: flex;
  flex-wrap: wrap;
  gap: var(--spacing-md, 16px);
}

.plan-card {
  width: 100%;
  box-sizing: border-box;
  background: var(--bg-color, #ffffff);
  border: 1px solid var(--border-color, #e5e5e5);
  border-radius: var(--radius-lg, 12px);
  padding: var(--spacing-lg, 24px);
  cursor: pointer;
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  display: flex;
  flex-direction: column;
}

.plan-card.half {
  width: calc(50% - 8px);
}

.plan-card.featured {
  background: linear-gradient(135deg, #f5edff 0%, #ede4ff 100%);
  border-color: var(--accent-color, #8B25FF);
  border-width: 2px;
}

.featured-badge {
  display: inline-block;
  font-size: 0.75rem;
  font-weight: 800;
  color: var(--accent-color, #8B25FF);
  letter-spacing: 0.02em;
  margin-bottom: 8px;
}

.featured-note {
  margin: 32px 0 32px;
  font-size: 1rem;
  font-weight: 600;
  color: #000;
  line-height: 1.8;
  letter-spacing: -0.02em;
  word-break: keep-all;
}

.featured-note mark {
  background: linear-gradient(to top, rgba(139, 37, 255, 0.2) 0%, rgba(139, 37, 255, 0.2) 40%, transparent 40%);
  color: inherit;
  padding: 0 2px;
}

.plan-card.featured .plan-card-title {
  color: var(--accent-color, #8B25FF);
}

.plan-card:hover {
  border-color: var(--accent-color, #8B25FF);
  transform: translateY(-2px);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.08);
}

.plan-card:active {
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
}

.plan-card-icon {
  font-size: 2.5rem;
  line-height: 1;
  margin-bottom: var(--spacing-md, 16px);
}

.plan-card-title {
  font-size: 1.25rem;
  font-weight: 700;
  color: var(--text-color, #000);
  margin: 0 0 6px 0;
  letter-spacing: -0.02em;
  line-height: 1.3;
}

.plan-card-subtitle {
  font-size: 0.875rem;
  font-weight: 600;
  color: #888;
  margin: 0 0 var(--spacing-md, 16px) 0;
  letter-spacing: -0.01em;
}

.plan-card-body {
  font-size: 1rem;
  color: var(--text-color, #000);
  line-height: 1.7;
  font-weight: 500;
  letter-spacing: -0.02em;
  margin: 0;
  flex: 1;
  word-break: keep-all;
  overflow-wrap: break-word;
  white-space: pre-line;
}

.plan-card-cta {
  display: block;
  margin-top: var(--spacing-md, 16px);
  padding-top: var(--spacing-sm, 12px);
  border-top: 1px solid var(--border-color, #e5e5e5);
  color: var(--accent-color, #8B25FF);
  font-size: 0.9375rem;
  font-weight: 700;
  letter-spacing: -0.02em;
  transition: opacity 0.2s ease;
}

.plan-card:hover .plan-card-cta {
  opacity: 0.8;
}

/* ===== 임원진 다짐 ===== */
.leaders-section {
  margin-top: var(--spacing-lg, 24px);
  padding-top: var(--spacing-lg, 24px);
  border-top: 1px solid var(--border-color, #e5e5e5);
}

.leaders-grid {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.leader-card {
  display: flex;
  flex-wrap: wrap;
  align-items: flex-start;
  gap: 12px;
  padding: 16px;
  background: #faf8f6;
  border-radius: var(--radius-lg, 12px);
}

.leader-photo {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  object-fit: cover;
  flex-shrink: 0;
}

.leader-info {
  display: flex;
  flex-direction: column;
  gap: 2px;
  padding-top: 4px;
}

.leader-name {
  font-size: 0.9375rem;
  font-weight: 700;
  color: var(--text-color, #000);
  letter-spacing: -0.02em;
}

.leader-role {
  font-size: 0.75rem;
  font-weight: 600;
  color: var(--accent-color, #8B25FF);
  letter-spacing: -0.01em;
}

.leader-message {
  width: 100%;
  margin: 0;
  font-size: 0.875rem;
  color: #555;
  line-height: 1.7;
  font-weight: 400;
  letter-spacing: -0.02em;
  word-break: keep-all;
}

/* ===== 소모임 ===== */
.clubs-section {
  margin-top: var(--spacing-lg, 24px);
  padding-top: var(--spacing-lg, 24px);
  border-top: 1px solid var(--border-color, #e5e5e5);
}

.club-label {
  font-weight: 400;
  color: #888;
  font-size: 0.875rem;
}

/* ===== 부가 활동 아코디언 ===== */
.extras-section {
  margin-top: var(--spacing-lg, 24px);
  padding-top: var(--spacing-lg, 24px);
  border-top: 1px solid var(--border-color, #e5e5e5);
}

.extras-heading {
  margin: 0 0 var(--spacing-sm, 12px) 0;
  font-size: 0.9375rem;
  font-weight: 700;
  color: #999;
  letter-spacing: -0.01em;
}

.extra-item {
  border: 1px solid var(--border-color, #e5e5e5);
  border-radius: 8px;
  margin-bottom: 8px;
  overflow: hidden;
  transition: border-color 0.2s ease;
}

.extra-item:last-child {
  margin-bottom: 0;
}

.extra-item.open {
  border-color: var(--accent-color, #8B25FF);
}

.extra-title {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  width: 100%;
  padding: 14px 16px;
  background: none;
  border: none;
  cursor: pointer;
  font-size: 1rem;
  font-weight: 600;
  color: var(--text-color, #000);
  letter-spacing: -0.02em;
  line-height: 1.4;
  text-align: left;
  transition: background 0.15s ease;
}

.extra-title:hover {
  background: rgba(139, 37, 255, 0.03);
}

.extra-arrow {
  flex-shrink: 0;
  font-size: 1.125rem;
  color: #999;
  transition: transform 0.2s ease;
  line-height: 1;
}

.extra-item.open .extra-arrow {
  transform: rotate(90deg);
  color: var(--accent-color, #8B25FF);
}

.extra-body {
  padding: 0 16px 16px;
  animation: extraFadeIn 0.2s ease;
}

.extra-body p {
  margin: 0;
  font-size: 0.9375rem;
  color: #555;
  line-height: 1.7;
  font-weight: 400;
  letter-spacing: -0.02em;
  word-break: keep-all;
  white-space: pre-line;
}

@keyframes extraFadeIn {
  from { opacity: 0; transform: translateY(-4px); }
  to { opacity: 1; transform: translateY(0); }
}

/* ===== 공통 ===== */
.photo-nav {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  background: rgba(0, 0, 0, 0.6);
  color: white;
  border: none;
  width: 36px;
  height: 36px;
  cursor: pointer;
  font-size: 18px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.25s ease;
  backdrop-filter: blur(4px);
}
.photo-nav:hover {
  background: rgba(139, 37, 255, 0.8);
  transform: translateY(-50%) scale(1.1);
}
.photo-nav:active {
  transform: translateY(-50%) scale(0.95);
}
.photo-nav.prev { left: 12px; }
.photo-nav.next { right: 12px; }

.videos-section {
  margin-top: var(--spacing-lg, 24px);
  padding-top: var(--spacing-lg, 24px);
  border-top: 1px solid var(--border-color, #e5e5e5);
}
.videos-title {
  margin: 0 0 var(--spacing-md, 16px) 0;
  font-size: 1rem;
  font-weight: 700;
  color: var(--accent-color, #8B25FF);
  letter-spacing: -0.02em;
}
.video-wrapper {
  position: relative;
  width: 100%;
  padding-bottom: 56.25%;
  margin-bottom: var(--spacing-md, 16px);
  background: #000;
  border-radius: var(--radius-md, 12px);
  overflow: hidden;
}
.video-wrapper:last-child { margin-bottom: 0; }
.video-wrapper iframe {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(16px); }
  to { opacity: 1; transform: translateY(0); }
}

/* 모바일 반응형 */
@media (max-width: 640px) {
  /* 피드 */
  .feed { gap: 20px; }
  .feed-card { border-radius: 0; border-left: none; border-right: none; }
  .feed-header { padding: 10px 12px; }
  .feed-caption { padding: 10px 12px 14px; }
  .feed-desc { font-size: 0.875rem; }

  /* 모달 내 마크다운 */
  .post-body { font-size: 0.9375rem; }
  .post-body :deep(h2) { font-size: 1.0625rem; }
  .post-body :deep(blockquote) {
    padding: 12px 14px;
    margin: 12px 0;
    font-size: 0.875rem;
  }
  .post-body :deep(ul),
  .post-body :deep(ol) { padding-left: 20px; }
  .post-description { font-size: 0.9375rem; }

  /* 사진 갤러리 */
  .post-gallery-main img { max-height: 300px; }
  .post-gallery-thumbs img { width: 48px; height: 48px; }
  .photo-nav { width: 40px; height: 40px; font-size: 18px; }
  .photo-nav.prev { left: 8px; }
  .photo-nav.next { right: 8px; }

  /* 카드 */
  .plan-cards { gap: 12px; }
  .plan-card { padding: var(--spacing-md, 16px); }
  .plan-card-icon { font-size: 2rem; margin-bottom: 12px; }
  .plan-card-title { font-size: 1.0625rem; }
  .plan-card-subtitle { font-size: 0.75rem; margin-bottom: 12px; }
  .plan-card-body { font-size: 0.875rem; line-height: 1.6; }
  .plan-card-cta { font-size: 0.8125rem; margin-top: 12px; padding-top: 10px; }

  /* half 카드 모바일 최적화 */
  .plan-card.half { width: calc(50% - 6px); }
  .plan-card.half .plan-card-icon { font-size: 1.75rem; margin-bottom: 10px; }
  .plan-card.half .plan-card-title { font-size: 0.875rem; }
  .plan-card.half .plan-card-subtitle { font-size: 0.625rem; margin-bottom: 10px; }
  .plan-card.half .plan-card-body { font-size: 0.75rem; line-height: 1.5; }
  .plan-card.half .plan-card-cta { font-size: 0.6875rem; }

  /* 아코디언 */
  .extra-title { padding: 12px 14px; font-size: 0.875rem; }
  .extra-body { padding: 0 14px 14px; }
  .extra-body p { font-size: 0.875rem; }

  /* 임원진 */
  .leader-card { padding: 14px; }
  .leader-photo { width: 44px; height: 44px; }
  .leader-message { font-size: 0.8125rem; }
}

/* 작은 모바일 (360px 이하) */
@media (max-width: 360px) {
  .plan-card.half { width: 100%; }
  .plan-card-body { font-size: 0.8125rem; }
  .post-body { font-size: 0.875rem; }
  .post-body :deep(h2) { font-size: 1rem; }
}

/* 터치 디바이스 최적화 */
@media (hover: none) and (pointer: coarse) {
  .feed-card:hover { box-shadow: none; }
  .feed-card:active { transform: scale(0.99); }
  .plan-card:hover {
    transform: none;
    box-shadow: none;
    border-color: var(--border-color, #e5e5e5);
  }
  .plan-card:active {
    transform: scale(0.98);
    border-color: var(--accent-color, #8B25FF);
  }
  .photo-nav:hover { transform: translateY(-50%); background: rgba(0, 0, 0, 0.6); }
  .photo-nav:active { transform: translateY(-50%) scale(0.95); background: rgba(139, 37, 255, 0.8); }
  .extra-title:hover { background: transparent; }
  .extra-title:active { background: rgba(139, 37, 255, 0.03); }
  .post-gallery-thumbs img:hover { opacity: 0.6; }
  .post-gallery-thumbs img.active:hover { opacity: 1; }
}
</style>
