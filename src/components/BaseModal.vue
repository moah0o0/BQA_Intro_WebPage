<template>
  <Teleport to="body">
    <div v-if="isOpen" class="modal-backdrop" @click="$emit('close')">
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <h3>{{ title }}</h3>
          <button class="close-btn" @click="$emit('close')">&times;</button>
        </div>
        <div class="modal-body">
          <slot></slot>
        </div>
      </div>
    </div>
  </Teleport>
</template>

<script setup>
import { onMounted, onUnmounted, watch } from 'vue';

const props = defineProps({
  isOpen: Boolean,
  title: String
});

const emit = defineEmits(['close']);

const handleKeydown = (e) => {
  if (e.key === 'Escape' && props.isOpen) {
    emit('close');
  }
};

// Body 스크롤 제어
watch(() => props.isOpen, (newVal) => {
  if (newVal) {
    // 모달이 열릴 때 body 스크롤 막기
    document.body.style.overflow = 'hidden';
  } else {
    // 모달이 닫힐 때 body 스크롤 복원
    document.body.style.overflow = '';
  }
});

onMounted(() => {
  window.addEventListener('keydown', handleKeydown);
});

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown);
  // 컴포넌트 언마운트 시 body 스크롤 복원
  document.body.style.overflow = '';
});
</script>

<style scoped>
.modal-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(8px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  padding: 20px;
  animation: backdropFadeIn 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  overflow: hidden;
}
.modal-content {
  background: var(--bg-color, #ffffff);
  color: var(--text-color, #000);
  width: 100%;
  max-width: 540px;
  max-height: 90vh;
  overflow-y: auto;
  border-radius: var(--radius-lg, 16px);
  border: 1px solid var(--border-color, #e5e5e5);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.15);
  animation: modalFadeIn 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  margin: auto;
  position: relative;
}

@media (max-width: 640px) {
  .modal-backdrop {
    padding: 0;
    align-items: flex-start;
  }
  .modal-content {
    max-height: 100vh;
    min-height: 100vh;
    border-radius: 0;
    margin: 0;
    border: none;
  }
  .modal-header {
    padding: 16px 20px;
    position: sticky;
    top: 0;
  }
  .modal-header h3 {
    font-size: 1.125rem;
    padding-right: 40px;
  }
  .close-btn {
    width: 40px;
    height: 40px;
    font-size: 2rem;
  }
  .modal-body {
    padding: 20px;
    font-size: 0.9375rem;
    padding-bottom: 40px;
  }
  .modal-body :deep(ul),
  .modal-body :deep(ol) {
    padding-left: 20px;
  }
  .modal-body :deep(blockquote) {
    padding: 14px 16px;
    margin: 14px 0;
  }
}

/* 작은 모바일 화면 (360px 이하) */
@media (max-width: 360px) {
  .modal-header {
    padding: 14px 16px;
  }
  .modal-header h3 {
    font-size: 1rem;
  }
  .modal-body {
    padding: 16px;
    font-size: 0.9375rem;
  }
}

/* 터치 디바이스 최적화 */
@media (hover: none) and (pointer: coarse) {
  .close-btn:hover {
    color: var(--text-color, #000);
  }
  .close-btn:active {
    color: var(--accent-color, #8B25FF);
    transform: scale(0.9);
    background: rgba(139, 37, 255, 0.1);
  }
  .modal-body :deep(a:hover) {
    text-decoration: none;
    opacity: 1;
  }
  .modal-body :deep(a:active) {
    text-decoration: underline;
    opacity: 0.8;
  }
  .modal-body :deep(.action-btn:hover) {
    transform: none;
    box-shadow: none;
  }
  .modal-body :deep(.action-btn:active) {
    transform: scale(0.98);
    background: #7a1fe6;
  }
}
.modal-header {
  padding: 24px;
  border-bottom: 1px solid var(--border-color, #e5e5e5);
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: var(--bg-color, #ffffff);
  position: sticky;
  top: 0;
  z-index: 10;
}
.modal-header h3 {
  margin: 0;
  color: var(--accent-color, #8B25FF);
  font-size: 1.25rem;
  font-weight: 700;
  letter-spacing: -0.02em;
  line-height: 1.3;
}
.close-btn {
  background: none;
  border: none;
  color: var(--text-color, #000);
  font-size: 1.75rem;
  cursor: pointer;
  transition: all 0.25s ease;
  padding: 0;
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  line-height: 1;
  font-weight: 300;
}
.close-btn:hover {
  color: var(--accent-color, #8B25FF);
}
.close-btn:active {
  transform: scale(0.95);
}
.modal-body {
  padding: 24px;
  line-height: 1.7;
  white-space: pre-line;
  color: var(--text-color, #000);
  font-size: 1rem;
  letter-spacing: -0.02em;
  font-weight: 500;
}

/* Modal body HTML elements */
.modal-body :deep(strong) {
  font-weight: 700;
  color: var(--text-color, #000);
}

.modal-body :deep(blockquote) {
  margin: 16px 0;
  padding: 16px 20px;
  background: var(--bg-color, #f8f8f8);
  border-left: 4px solid var(--accent-color, #8B25FF);
  border-radius: 4px;
  font-style: italic;
  color: var(--text-color, #333);
}

.modal-body :deep(ul),
.modal-body :deep(ol) {
  margin: 12px 0;
  padding-left: 24px;
}

.modal-body :deep(li) {
  margin-bottom: 4px;
  line-height: 1.5;
  color: var(--text-color, #000);
  position: relative;
}

.modal-body :deep(ul li)::marker {
  color: var(--accent-color, #8B25FF);
  font-weight: bold;
}

.modal-body :deep(a) {
  color: var(--accent-color, #8B25FF);
  text-decoration: none;
  font-weight: 600;
  transition: all 0.25s ease;
}

.modal-body :deep(a:hover) {
  text-decoration: underline;
  opacity: 0.8;
}

.modal-body :deep(.action-btn) {
  display: inline-block;
  margin-top: 8px;
  padding: 8px 16px;
  background: var(--accent-color, #8B25FF);
  color: white;
  border-radius: 6px;
  text-decoration: none;
  font-weight: 600;
  transition: all 0.25s ease;
}

.modal-body :deep(.action-btn:hover) {
  background: #9d3fff;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(139, 37, 255, 0.3);
  text-decoration: none;
}

@keyframes backdropFadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
@keyframes modalFadeIn {
  from {
    opacity: 0;
    transform: scale(0.95) translateY(-20px);
  }
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

/* Smooth Scrollbar */
.modal-content::-webkit-scrollbar {
  width: 8px;
}
.modal-content::-webkit-scrollbar-track {
  background: transparent;
}
.modal-content::-webkit-scrollbar-thumb {
  background: var(--border-color, #e5e5e5);
  border-radius: 4px;
}
.modal-content::-webkit-scrollbar-thumb:hover {
  background: var(--accent-color, #8B25FF);
}
</style>