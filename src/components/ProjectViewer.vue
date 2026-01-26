<template>
  <div class="project-viewer">
    <!-- 2025년 결과 (DoneProject) -->
    <div v-if="data.type === 'DoneProject'" class="view-done">
      <div class="items-list">
        <div v-for="(item, idx) in data.items" :key="idx" class="card"
             :class="{ 'clickable': item.is_modal }"
             @click="item.is_modal ? openModalWithContent(item) : null">
          <div v-if="item.photos && item.photos.length" class="card-img-wrapper"
               :class="{ 'square': imageAspectRatios[idx] === 'square' }"
               @click.stop>
            <img :src="item.photos[currentPhotoIndex[idx] || 0]" alt="활동 사진"
                 @load="checkImageRatio($event, idx)" />
            <button v-if="item.photos.length > 1" class="photo-nav prev" @click="prevPhoto(idx, item.photos.length)">&lt;</button>
            <button v-if="item.photos.length > 1" class="photo-nav next" @click="nextPhoto(idx, item.photos.length)">&gt;</button>
            <div v-if="item.photos.length > 1" class="photo-indicators">
              <span v-for="(photo, pIdx) in item.photos" :key="pIdx"
                    :class="{ active: (currentPhotoIndex[idx] || 0) === pIdx }"
                    @click="setPhoto(idx, pIdx)"></span>
            </div>
          </div>
          <div class="card-body">
            <div class="card-header-row">
              <div class="card-title-wrapper">
                <h4>{{ item.subtitle }}</h4>
                <span v-if="item.date" class="card-date">{{ item.date }}</span>
              </div>
              <span v-if="item.is_modal" class="more-icon">+</span>
            </div>
            <div v-if="!item.is_modal" class="card-content">
              <p v-html="item.description"></p>
              <ul v-if="item.details && item.details.length" class="details-list">
                <li v-for="(detail, dIdx) in item.details" :key="dIdx" v-html="detail"></li>
              </ul>
            </div>
            <div v-else class="card-summary">
              <p v-html="item.description"></p>
              <small class="click-hint">자세히 보기</small>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 2026년 계획 (PlanProject) - 새 구조 -->
    <div v-if="data.type === 'PlanProject'" class="view-plan">

      <!-- 배경 섹션 -->
      <div v-if="data.background && data.background.length" class="background-section">
        <h3 class="section-title">배경</h3>
        <ul class="background-list">
          <li v-for="(item, idx) in data.background" :key="idx">{{ item }}</li>
        </ul>
      </div>

      <!-- 정기모임 개요 섹션 -->
      <div v-if="data.overview" class="overview-section">
        <h3 class="section-title">정기모임</h3>
        <div class="overview-card">
          <div class="overview-item">
            <span class="overview-label">시기</span>
            <span class="overview-value">{{ data.overview.period }}</span>
          </div>
          <div class="overview-item">
            <span class="overview-label">일시</span>
            <span class="overview-value">{{ data.overview.schedule }}</span>
          </div>
          <div class="overview-item">
            <span class="overview-label">대상</span>
            <span class="overview-value">{{ data.overview.target }}</span>
          </div>
        </div>
      </div>

      <!-- 목적 및 목표 섹션 -->
      <div v-if="data.goals && data.goals.length" class="goals-section">
        <h3 class="section-title">정기모임 목적 및 목표</h3>
        <ul class="goals-list">
          <li v-for="(goal, idx) in data.goals" :key="idx">{{ goal }}</li>
        </ul>
      </div>

      <!-- 정기모임 내용 -->
      <div v-if="data.programs && data.programs.length" class="programs-section">
        <h3 class="section-title">정기모임 내용</h3>
        <div class="programs-list">
          <div
            v-for="(program, idx) in data.programs"
            :key="program.id"
            class="program-card"
            :class="{ 'clickable': program.is_modal }"
            @click="program.is_modal ? openProgramModal(program) : null"
          >
            <div class="program-header">
              <span class="program-number">{{ idx + 1 }}</span>
              <div class="program-info">
                <h4>{{ program.title }}</h4>
                <span class="program-schedule">{{ program.schedule }}</span>
              </div>
              <span v-if="program.is_modal" class="more-icon">+</span>
            </div>
            <p class="program-summary">{{ program.summary }}</p>
            <template v-if="!program.is_modal && program.details && program.details.length">
              <ul class="program-details">
                <li v-for="(detail, dIdx) in program.details" :key="dIdx" v-html="detail"></li>
              </ul>
            </template>
            <small v-if="program.is_modal" class="click-hint">자세히 보기</small>
          </div>
        </div>
      </div>

      <!-- 정기모임 연계활동 -->
      <div v-if="data.linked_activities && data.linked_activities.length" class="activities-section">
        <h3 class="section-title">정기모임 연계활동</h3>
        <div class="activities-list">
          <div v-for="activity in data.linked_activities" :key="activity.id" class="activity-item">
            <span class="activity-date">{{ activity.date }}</span>
            <div class="activity-content">
              <h4>{{ activity.title }}</h4>
              <p v-if="activity.description">{{ activity.description }}</p>
            </div>
          </div>
        </div>
      </div>

      <!-- 연대사업 -->
      <div v-if="data.solidarity_activities && data.solidarity_activities.length" class="solidarity-section">
        <h3 class="section-title">연대사업</h3>
        <div class="solidarity-list">
          <div v-for="item in data.solidarity_activities" :key="item.id" class="solidarity-card">
            <span class="solidarity-date">{{ item.date }}</span>
            <h4>{{ item.title }}</h4>
            <p>{{ item.description }}</p>
          </div>
        </div>
      </div>

      <!-- MT / 캠프 -->
      <div v-if="data.camp" class="camp-section">
        <h3 class="section-title">{{ data.camp.title }}</h3>
        <div class="camp-card">
          <div class="camp-header">
            <span class="camp-schedule">{{ data.camp.schedule }}</span>
          </div>
          <p class="camp-desc">{{ data.camp.description }}</p>
          <ul v-if="data.camp.details && data.camp.details.length" class="camp-details">
            <li v-for="(detail, idx) in data.camp.details" :key="idx">{{ detail }}</li>
          </ul>
        </div>
      </div>

      <!-- 열린 회의 / 교육 -->
      <div v-if="data.open_meetings && data.open_meetings.length" class="open-meetings-section">
        <h3 class="section-title">열린 회의 및 교육</h3>
        <div class="open-meetings-list">
          <div
            v-for="meeting in data.open_meetings"
            :key="meeting.id"
            class="open-meeting-card"
            :class="{ 'clickable': meeting.is_modal }"
            @click="meeting.is_modal ? openMeetingModal(meeting) : null"
          >
            <div class="open-meeting-header">
              <div class="open-meeting-title-wrapper">
                <h4>{{ meeting.title }}</h4>
                <span class="open-meeting-schedule">{{ meeting.schedule }}</span>
              </div>
              <span v-if="meeting.is_modal" class="more-icon">+</span>
            </div>
            <p class="open-meeting-desc">{{ meeting.description }}</p>
            <template v-if="!meeting.is_modal && meeting.details && meeting.details.length">
              <ul class="open-meeting-details">
                <li v-for="(detail, idx) in meeting.details" :key="idx">{{ detail }}</li>
              </ul>
            </template>
            <small v-if="meeting.is_modal" class="click-hint" style="padding-left: 0;">자세히 보기</small>
          </div>
        </div>
      </div>

      <!-- 회원이 여는 소모임 -->
      <div v-if="data.sub_clubs && data.sub_clubs.length" class="clubs-section">
        <h3 class="section-title">회원이 여는 소모임</h3>
        <div class="clubs-list">
          <div v-for="club in data.sub_clubs" :key="club.id" class="club-card">
            <h4>{{ club.name }}</h4>
            <p>{{ club.description }}</p>
          </div>
        </div>
      </div>

    </div>

    <!-- 모달 -->
    <BaseModal :is-open="showModal" :title="modalContent.title" @close="showModal = false">
      <div v-if="modalContent.details">
        <p v-for="(p, i) in modalContent.details" :key="i" style="margin-bottom:10px;" v-html="p"></p>
      </div>
      <div v-if="modalContent.videos && modalContent.videos.length" class="videos-section">
        <h4 class="videos-title">영상</h4>
        <div v-for="(video, vIdx) in modalContent.videos" :key="vIdx" class="video-wrapper">
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
import { ref } from 'vue';
import BaseModal from './BaseModal.vue';

const props = defineProps({
  data: Object
});

const showModal = ref(false);
const modalContent = ref({ title: '', details: [], videos: [] });
const currentPhotoIndex = ref({});
const imageAspectRatios = ref({});

// 프로그램 모달 열기
const openProgramModal = (program) => {
  if (!program?.is_modal) return;

  modalContent.value = {
    title: program.title,
    details: program.details,
    videos: program.videos || []
  };
  showModal.value = true;
};

// 열린 회의/교육 모달 열기
const openMeetingModal = (meeting) => {
  if (!meeting?.is_modal) return;

  modalContent.value = {
    title: meeting.title,
    details: meeting.details,
    videos: []
  };
  showModal.value = true;
};

// 기존 DoneProject용 모달
const openModalWithContent = (item) => {
  if (!item?.is_modal) return;

  modalContent.value = {
    title: item.subtitle,
    details: item.details,
    videos: item.videos || []
  };
  showModal.value = true;
};

const nextPhoto = (itemIdx, total) => {
  const current = currentPhotoIndex.value[itemIdx] || 0;
  currentPhotoIndex.value[itemIdx] = (current + 1) % total;
};

const prevPhoto = (itemIdx, total) => {
  const current = currentPhotoIndex.value[itemIdx] || 0;
  currentPhotoIndex.value[itemIdx] = current === 0 ? total - 1 : current - 1;
};

const setPhoto = (itemIdx, photoIdx) => {
  currentPhotoIndex.value[itemIdx] = photoIdx;
};

const checkImageRatio = (event, idx) => {
  const img = event.target;
  const ratio = img.naturalWidth / img.naturalHeight;
  if (ratio >= 0.95 && ratio <= 1.05) {
    imageAspectRatios.value[idx] = 'square';
  } else {
    imageAspectRatios.value[idx] = 'auto';
  }
};
</script>

<style scoped>
.project-viewer {
  animation: fadeIn 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

/* ===== 2026 계획 스타일 ===== */
.view-plan {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-xl, 32px);
}

/* 배경 섹션 */
.background-section {
  margin-bottom: var(--spacing-md, 16px);
}

.background-list {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  flex-direction: column;
  gap: var(--spacing-sm, 12px);
}

.background-list li {
  position: relative;
  padding-left: 24px;
  font-size: 0.9375rem;
  color: var(--text-color, #000);
  line-height: 1.6;
  font-weight: 500;
  letter-spacing: -0.02em;
}

.background-list li::before {
  content: '•';
  position: absolute;
  left: 0;
  color: var(--accent-color, #8B25FF);
  font-weight: 700;
}

/* 개요 섹션 */
.overview-section {
  margin-bottom: var(--spacing-md, 16px);
}

.overview-card {
  background: linear-gradient(135deg, rgba(139, 37, 255, 0.05), rgba(139, 37, 255, 0.02));
  border: 1px solid rgba(139, 37, 255, 0.15);
  border-radius: var(--radius-md, 12px);
  padding: var(--spacing-lg, 24px);
  display: flex;
  flex-direction: column;
  gap: var(--spacing-md, 16px);
}

.overview-item {
  display: flex;
  gap: var(--spacing-md, 16px);
  align-items: flex-start;
}

.overview-label {
  min-width: 48px;
  font-size: 0.75rem;
  font-weight: 700;
  color: var(--accent-color, #8B25FF);
  text-transform: uppercase;
  letter-spacing: 0.03em;
}

.overview-value {
  font-size: 0.9375rem;
  color: var(--text-color, #000);
  line-height: 1.5;
  font-weight: 500;
  letter-spacing: -0.02em;
}

/* 섹션 타이틀 */
.section-title {
  font-size: 1.25rem;
  font-weight: 700;
  color: var(--text-color, #000);
  margin: 0 0 var(--spacing-md, 16px) 0;
  padding-bottom: var(--spacing-xs, 8px);
  border-bottom: 2px solid var(--accent-color, #8B25FF);
  letter-spacing: -0.02em;
}

/* 목표 섹션 */
.goals-section {
  margin-bottom: var(--spacing-md, 16px);
}

.goals-list {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  flex-direction: column;
  gap: var(--spacing-sm, 12px);
}

.goals-list li {
  position: relative;
  padding-left: 24px;
  font-size: 0.9375rem;
  color: var(--text-color, #000);
  line-height: 1.6;
  font-weight: 500;
  letter-spacing: -0.02em;
}

.goals-list li::before {
  content: '✓';
  position: absolute;
  left: 0;
  color: var(--accent-color, #8B25FF);
  font-weight: 700;
}

/* 프로그램 섹션 */
.programs-section {
  margin-bottom: var(--spacing-md, 16px);
}

.programs-list {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-md, 16px);
}

.program-card {
  background: var(--bg-color, #ffffff);
  border: 1px solid var(--border-color, #e5e5e5);
  border-radius: var(--radius-md, 12px);
  padding: var(--spacing-lg, 24px);
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
}

.program-card:hover {
  border-color: var(--accent-color, #8B25FF);
}

.program-card.clickable {
  cursor: pointer;
}

.program-card.clickable:active {
  transform: scale(0.99);
}

.program-header {
  display: flex;
  align-items: flex-start;
  gap: var(--spacing-md, 16px);
  margin-bottom: var(--spacing-sm, 12px);
}

.program-number {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  background: var(--accent-color, #8B25FF);
  color: white;
  font-size: 0.875rem;
  font-weight: 700;
  border-radius: 50%;
  flex-shrink: 0;
}

.program-info {
  flex: 1;
}

.program-info h4 {
  margin: 0 0 4px 0;
  font-size: 1.0625rem;
  font-weight: 700;
  color: var(--text-color, #000);
  letter-spacing: -0.02em;
  line-height: 1.4;
}

.program-schedule {
  font-size: 0.8125rem;
  color: var(--accent-color, #8B25FF);
  font-weight: 600;
  letter-spacing: -0.01em;
}

.program-summary {
  margin: 0;
  font-size: 0.9375rem;
  color: var(--text-color, #000);
  line-height: 1.6;
  font-weight: 500;
  letter-spacing: -0.02em;
  padding-left: 48px;
}

.program-details {
  margin: var(--spacing-md, 16px) 0 0 0;
  padding-left: 68px;
  list-style: none;
}

.program-details li {
  position: relative;
  padding-left: 16px;
  margin-bottom: var(--spacing-xs, 8px);
  font-size: 0.875rem;
  color: var(--text-color, #000);
  line-height: 1.6;
  font-weight: 500;
  letter-spacing: -0.02em;
}

.program-details li::before {
  content: '•';
  position: absolute;
  left: 0;
  color: var(--accent-color, #8B25FF);
}

.program-details li:last-child {
  margin-bottom: 0;
}

.program-card .click-hint {
  padding-left: 48px;
}

.more-icon {
  color: var(--accent-color, #8B25FF);
  font-weight: 700;
  font-size: 1.5rem;
  line-height: 1;
  transition: transform 0.25s ease;
  flex-shrink: 0;
}

.program-card.clickable:hover .more-icon {
  transform: rotate(90deg);
}

.click-hint {
  display: block;
  margin-top: var(--spacing-xs, 8px);
  color: var(--accent-color, #8B25FF);
  font-size: 0.75rem;
  font-weight: 600;
  letter-spacing: -0.02em;
}

/* 정기모임 연계활동 */
.activities-section {
  margin-bottom: var(--spacing-md, 16px);
}

.activities-list {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-sm, 12px);
}

.activity-item {
  display: flex;
  gap: var(--spacing-md, 16px);
  padding: var(--spacing-md, 16px);
  background: var(--bg-color, #ffffff);
  border: 1px solid var(--border-color, #e5e5e5);
  border-radius: var(--radius-md, 12px);
}

.activity-date {
  min-width: 100px;
  font-size: 0.8125rem;
  font-weight: 600;
  color: var(--accent-color, #8B25FF);
  letter-spacing: -0.01em;
}

.activity-content h4 {
  margin: 0 0 4px 0;
  font-size: 0.9375rem;
  font-weight: 600;
  color: var(--text-color, #000);
  letter-spacing: -0.02em;
}

.activity-content p {
  margin: 0;
  font-size: 0.875rem;
  color: #666;
  line-height: 1.5;
  font-weight: 500;
  letter-spacing: -0.02em;
}

/* 연대사업 */
.solidarity-section {
  margin-bottom: var(--spacing-md, 16px);
}

.solidarity-list {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-md, 16px);
}

.solidarity-card {
  background: var(--bg-color, #ffffff);
  border: 1px solid var(--border-color, #e5e5e5);
  border-radius: var(--radius-md, 12px);
  padding: var(--spacing-lg, 24px);
}

.solidarity-date {
  display: inline-block;
  font-size: 0.75rem;
  font-weight: 600;
  color: var(--accent-color, #8B25FF);
  margin-bottom: var(--spacing-xs, 8px);
  letter-spacing: -0.01em;
}

.solidarity-card h4 {
  margin: 0 0 var(--spacing-sm, 12px) 0;
  font-size: 1rem;
  font-weight: 700;
  color: var(--text-color, #000);
  letter-spacing: -0.02em;
  line-height: 1.4;
}

.solidarity-card p {
  margin: 0;
  font-size: 0.9375rem;
  color: var(--text-color, #000);
  line-height: 1.6;
  font-weight: 500;
  letter-spacing: -0.02em;
}

/* MT / 캠프 섹션 */
.camp-section {
  margin-bottom: var(--spacing-md, 16px);
}

.camp-card {
  background: linear-gradient(135deg, rgba(255, 107, 107, 0.08), rgba(255, 159, 67, 0.05));
  border: 1px solid rgba(255, 107, 107, 0.25);
  border-radius: var(--radius-md, 12px);
  padding: var(--spacing-lg, 24px);
}

.camp-header {
  margin-bottom: var(--spacing-sm, 12px);
}

.camp-schedule {
  display: inline-block;
  font-size: 0.875rem;
  font-weight: 700;
  color: #ff6b6b;
  letter-spacing: -0.01em;
}

.camp-desc {
  margin: 0 0 var(--spacing-md, 16px) 0;
  font-size: 1.0625rem;
  font-weight: 600;
  color: var(--text-color, #000);
  line-height: 1.5;
  letter-spacing: -0.02em;
}

.camp-details {
  margin: 0;
  padding-left: 20px;
  list-style: none;
}

.camp-details li {
  position: relative;
  padding-left: 16px;
  margin-bottom: var(--spacing-xs, 8px);
  font-size: 0.9375rem;
  color: var(--text-color, #000);
  line-height: 1.5;
  font-weight: 500;
  letter-spacing: -0.02em;
}

.camp-details li::before {
  content: '•';
  position: absolute;
  left: 0;
  color: #ff6b6b;
}

/* 열린 회의 및 교육 */
.open-meetings-section {
  margin-bottom: var(--spacing-md, 16px);
}

.open-meetings-list {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-md, 16px);
}

.open-meeting-card {
  background: linear-gradient(135deg, rgba(139, 37, 255, 0.03), rgba(139, 37, 255, 0.01));
  border: 1px solid rgba(139, 37, 255, 0.15);
  border-radius: var(--radius-md, 12px);
  padding: var(--spacing-lg, 24px);
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
}

.open-meeting-card:hover {
  border-color: var(--accent-color, #8B25FF);
}

.open-meeting-card.clickable {
  cursor: pointer;
}

.open-meeting-card.clickable:active {
  transform: scale(0.99);
}

.open-meeting-card.clickable:hover .more-icon {
  transform: rotate(90deg);
}

.open-meeting-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: var(--spacing-md, 16px);
  margin-bottom: var(--spacing-sm, 12px);
}

.open-meeting-title-wrapper {
  flex: 1;
}

.open-meeting-card h4 {
  margin: 0 0 4px 0;
  font-size: 1.0625rem;
  font-weight: 700;
  color: var(--text-color, #000);
  letter-spacing: -0.02em;
}

.open-meeting-schedule {
  font-size: 0.8125rem;
  font-weight: 600;
  color: var(--accent-color, #8B25FF);
  letter-spacing: -0.01em;
  white-space: nowrap;
}

.open-meeting-desc {
  margin: 0 0 var(--spacing-sm, 12px) 0;
  font-size: 0.9375rem;
  color: var(--text-color, #000);
  line-height: 1.6;
  font-weight: 500;
  letter-spacing: -0.02em;
}

.open-meeting-details {
  margin: 0;
  padding-left: 20px;
  list-style: none;
}

.open-meeting-details li {
  position: relative;
  padding-left: 16px;
  margin-bottom: var(--spacing-xs, 8px);
  font-size: 0.875rem;
  color: var(--text-color, #000);
  line-height: 1.5;
  font-weight: 500;
  letter-spacing: -0.02em;
}

.open-meeting-details li::before {
  content: '•';
  position: absolute;
  left: 0;
  color: var(--accent-color, #8B25FF);
}

/* 소모임 섹션 */
.clubs-section {
  margin-bottom: var(--spacing-md, 16px);
}

.clubs-list {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: var(--spacing-md, 16px);
}

.club-card {
  background: var(--bg-color, #ffffff);
  border: 1px solid var(--border-color, #e5e5e5);
  border-radius: var(--radius-md, 12px);
  padding: var(--spacing-md, 16px);
}

.club-card h4 {
  margin: 0 0 var(--spacing-xs, 8px) 0;
  font-size: 1rem;
  font-weight: 700;
  color: var(--text-color, #000);
  letter-spacing: -0.02em;
}

.club-card p {
  margin: 0;
  font-size: 0.875rem;
  color: #666;
  line-height: 1.5;
  font-weight: 500;
  letter-spacing: -0.02em;
}

/* ===== 2025년 결과 스타일 (기존 유지) ===== */
.card {
  background: var(--bg-color, #ffffff);
  border-radius: var(--radius-md, 12px);
  overflow: hidden;
  margin-bottom: var(--spacing-lg, 24px);
  border: 1px solid var(--border-color, #e5e5e5);
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
}
.card:hover {
  transform: translateY(-2px);
  border-color: var(--accent-color, #8B25FF);
}
.card.clickable {
  cursor: pointer;
}
.card.clickable:active {
  transform: scale(0.99);
}
.card-img-wrapper {
  position: relative;
  background: #f0f0f0;
  width: 100%;
  overflow: hidden;
}
.card-img-wrapper.square {
  padding-bottom: 100%;
}
.card-img-wrapper.square img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.card-img-wrapper:not(.square) img {
  width: 100%;
  height: auto;
  display: block;
  object-fit: contain;
  max-height: 500px;
}
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
.photo-indicators {
  position: absolute;
  bottom: 12px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 6px;
  padding: 6px 12px;
  background: rgba(0, 0, 0, 0.5);
  border-radius: 20px;
  backdrop-filter: blur(4px);
}
.photo-indicators span {
  width: 8px;
  height: 8px;
  background: rgba(255, 255, 255, 0.4);
  border-radius: 50%;
  cursor: pointer;
  transition: all 0.25s ease;
}
.photo-indicators span:hover {
  background: rgba(255, 255, 255, 0.7);
}
.photo-indicators span.active {
  background: var(--accent-color, #8B25FF);
  width: 24px;
  border-radius: 4px;
}
.card-body {
  padding: var(--spacing-lg, 24px);
}
.card-header-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: var(--spacing-sm, 12px);
}
.card-title-wrapper {
  flex: 1;
}
.card h4 {
  margin: 0;
  font-size: 1.125rem;
  font-weight: 700;
  color: var(--text-color, #000);
  letter-spacing: -0.02em;
  line-height: 1.4;
}
.card-date {
  display: inline-block;
  margin-top: 4px;
  font-size: 0.8125rem;
  color: var(--accent-color, #8B25FF);
  font-weight: 600;
  letter-spacing: -0.01em;
}
.card-content p {
  margin: 0 0 var(--spacing-md, 16px) 0;
  color: var(--text-color, #000);
  font-size: 0.9375rem;
  line-height: 1.7;
  font-weight: 500;
  letter-spacing: -0.02em;
}
.card-summary p {
  margin: 0 0 var(--spacing-xs, 8px) 0;
  color: var(--text-color, #000);
  font-size: 0.9375rem;
  line-height: 1.7;
  font-weight: 500;
  letter-spacing: -0.02em;
}
.details-list {
  padding-left: 20px;
  color: var(--text-color, #000);
  font-size: 0.875rem;
  margin: var(--spacing-sm, 12px) 0 0 0;
  line-height: 1.6;
  font-weight: 500;
  letter-spacing: -0.02em;
}
.details-list li {
  margin-bottom: var(--spacing-xs, 8px);
}

/* 비디오 섹션 */
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
.video-wrapper:last-child {
  margin-bottom: 0;
}
.video-wrapper iframe {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
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

/* 모바일 반응형 */
@media (max-width: 640px) {
  .overview-card {
    padding: var(--spacing-md, 16px);
  }
  .overview-item {
    flex-direction: column;
    gap: 4px;
  }
  .overview-label {
    min-width: auto;
  }
  .section-title {
    font-size: 1.125rem;
  }
  .program-card {
    padding: var(--spacing-md, 16px);
  }
  .program-header {
    gap: var(--spacing-sm, 12px);
  }
  .program-number {
    width: 28px;
    height: 28px;
    font-size: 0.8125rem;
  }
  .program-info h4 {
    font-size: 1rem;
  }
  .program-summary,
  .program-card .click-hint {
    padding-left: 40px;
  }
  .program-details {
    padding-left: 56px;
  }
  .clubs-list {
    grid-template-columns: 1fr;
  }
  .activity-item {
    flex-direction: column;
    gap: var(--spacing-xs, 8px);
  }
  .activity-date {
    min-width: auto;
  }
  .open-meeting-header {
    flex-direction: column;
    gap: var(--spacing-xs, 8px);
  }
  .solidarity-card {
    padding: var(--spacing-md, 16px);
  }
  .open-meeting-card {
    padding: var(--spacing-md, 16px);
  }
  .photo-nav {
    width: 40px;
    height: 40px;
    font-size: 18px;
  }
  .photo-nav.prev { left: 8px; }
  .photo-nav.next { right: 8px; }
  .card {
    margin-bottom: var(--spacing-md, 16px);
  }
  .card-body {
    padding: var(--spacing-md, 16px);
  }
  .card h4 {
    font-size: 1rem;
  }
}

@media (max-width: 360px) {
  .program-card {
    padding: var(--spacing-sm, 12px);
  }
  .program-number {
    width: 24px;
    height: 24px;
    font-size: 0.75rem;
  }
  .program-summary,
  .program-card .click-hint {
    padding-left: 36px;
  }
  .program-details {
    padding-left: 52px;
  }
}

/* 터치 디바이스 최적화 */
@media (hover: none) and (pointer: coarse) {
  .card:hover,
  .program-card:hover,
  .open-meeting-card:hover {
    transform: none;
    border-color: rgba(139, 37, 255, 0.15);
  }
  .card.clickable:active,
  .program-card.clickable:active,
  .open-meeting-card.clickable:active {
    transform: scale(0.98);
    border-color: var(--accent-color, #8B25FF);
  }
  .photo-nav:hover {
    transform: translateY(-50%);
    background: rgba(0, 0, 0, 0.6);
  }
  .photo-nav:active {
    transform: translateY(-50%) scale(0.95);
    background: rgba(139, 37, 255, 0.8);
  }
}
</style>
