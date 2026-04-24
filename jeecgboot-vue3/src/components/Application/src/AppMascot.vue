<template>
  <div
    class="app-mascot"
    :class="{ collapsed: isCollapsed }"
    :style="{ left: `${position.x}px`, top: `${position.y}px` }"
    @mousedown="startDrag"
    @touchstart.passive="startTouchDrag"
  >
    <div v-if="!isCollapsed" class="app-mascot-bubble" @click="nextMessage">
      {{ currentMessage }}
    </div>
    <button class="app-mascot-toggle" type="button" @click.stop="toggleCollapsed">
      {{ isCollapsed ? '展开' : '收起' }}
    </button>
    <img
      ref="imgRef"
      src="/@/assets/images/sagiri_dark.png"
      alt="mascot"
      class="app-mascot-img"
      :style="{ width: `${displaySize.width}px`, height: `${displaySize.height}px` }"
      @click="nextMessage"
      @load="handleImageLoad"
    />
  </div>
</template>

<script lang="ts" setup>
  import { computed, onBeforeUnmount, onMounted, ref } from 'vue';

  const position = ref({ x: 16, y: window.innerHeight - 148 });
  const isDragging = ref(false);
  const isCollapsed = ref(false);
  const messageIndex = ref(0);
  const imgRef = ref<HTMLImageElement | null>(null);
  const naturalSize = ref({ width: 116, height: 116 });
  const displaySize = ref({ width: 116, height: 116 });

  const messages = ['你好呀，今天也要高效开发哦~', '遇到问题点我一下，也许有灵感！', '记得喝水和保存代码。', '左下角看板娘持续待命中。'];

  const currentMessage = computed(() => messages[messageIndex.value % messages.length]);

  function recalculateSize() {
    const ratio = naturalSize.value.width / naturalSize.value.height || 1;
    const maxWidth = Math.round(Math.min(window.innerWidth * 0.16, 220));
    const maxHeight = Math.round(Math.min(window.innerHeight * 0.24, 260));
    const minWidth = window.innerWidth <= 768 ? 72 : 96;
    const minHeight = window.innerWidth <= 768 ? 72 : 96;

    let width = maxWidth;
    let height = Math.round(width / ratio);
    if (height > maxHeight) {
      height = maxHeight;
      width = Math.round(height * ratio);
    }

    displaySize.value = {
      width: Math.max(minWidth, width),
      height: Math.max(minHeight, height),
    };
    position.value = clampPosition(position.value.x, position.value.y);
  }

  function handleImageLoad() {
    if (!imgRef.value) return;
    naturalSize.value = {
      width: imgRef.value.naturalWidth || 116,
      height: imgRef.value.naturalHeight || 116,
    };
    recalculateSize();
  }

  function clampPosition(x: number, y: number) {
    const maxX = Math.max(12, window.innerWidth - displaySize.value.width - 12);
    const maxY = Math.max(12, window.innerHeight - displaySize.value.height - 12);
    return {
      x: Math.min(Math.max(12, x), maxX),
      y: Math.min(Math.max(12, y), maxY),
    };
  }

  function nextMessage() {
    messageIndex.value += 1;
  }

  function toggleCollapsed() {
    isCollapsed.value = !isCollapsed.value;
  }

  function onMouseMove(e: MouseEvent) {
    if (!isDragging.value) return;
    position.value = clampPosition(e.clientX - displaySize.value.width / 2, e.clientY - displaySize.value.height / 2);
  }

  function onMouseUp() {
    isDragging.value = false;
    window.removeEventListener('mousemove', onMouseMove);
    window.removeEventListener('mouseup', onMouseUp);
  }

  function startDrag() {
    isDragging.value = true;
    window.addEventListener('mousemove', onMouseMove);
    window.addEventListener('mouseup', onMouseUp);
  }

  function onTouchMove(e: TouchEvent) {
    if (!isDragging.value || !e.touches?.[0]) return;
    const touch = e.touches[0];
    position.value = clampPosition(touch.clientX - displaySize.value.width / 2, touch.clientY - displaySize.value.height / 2);
  }

  function onTouchEnd() {
    isDragging.value = false;
    window.removeEventListener('touchmove', onTouchMove);
    window.removeEventListener('touchend', onTouchEnd);
  }

  function startTouchDrag() {
    isDragging.value = true;
    window.addEventListener('touchmove', onTouchMove, { passive: true });
    window.addEventListener('touchend', onTouchEnd);
  }

  function handleResize() {
    recalculateSize();
  }

  onMounted(() => {
    recalculateSize();
    window.addEventListener('resize', handleResize);
  });

  onBeforeUnmount(() => {
    window.removeEventListener('resize', handleResize);
    window.removeEventListener('mousemove', onMouseMove);
    window.removeEventListener('mouseup', onMouseUp);
    window.removeEventListener('touchmove', onTouchMove);
    window.removeEventListener('touchend', onTouchEnd);
  });
</script>

<style lang="less" scoped>
  .app-mascot {
    position: fixed;
    z-index: 300;
    pointer-events: auto;
    user-select: none;
    animation: mascot-float 3.2s ease-in-out infinite;
    cursor: grab;
  }

  .app-mascot:active {
    cursor: grabbing;
  }

  .app-mascot-bubble {
    max-width: 220px;
    margin-bottom: 8px;
    padding: 8px 12px;
    color: #e2e8f0;
    font-size: 12px;
    line-height: 1.5;
    border: 1px solid rgb(148 163 184 / 30%);
    border-radius: 12px;
    backdrop-filter: blur(10px);
    background: rgb(15 23 42 / 75%);
    box-shadow: 0 6px 18px rgb(15 23 42 / 32%);
  }

  .app-mascot-toggle {
    position: absolute;
    top: -6px;
    right: -8px;
    padding: 2px 8px;
    color: #dbeafe;
    font-size: 11px;
    border: 1px solid rgb(148 163 184 / 35%);
    border-radius: 999px;
    background: rgb(30 41 59 / 75%);
    cursor: pointer;
  }

  .collapsed .app-mascot-bubble {
    display: none;
  }

  .app-mascot-img {
    object-fit: contain;
    border-radius: 14px;
    border: 2px solid rgb(255 255 255 / 42%);
    box-shadow: 0 10px 28px rgb(15 23 42 / 35%);
    opacity: 0.92;

    max-width: 220px;
    max-height: 260px;
  }

  @media (max-width: 768px) {
    .app-mascot-bubble {
      max-width: 170px;
    }
  }

  @keyframes mascot-float {
    0%,
    100% {
      transform: translateY(0);
    }
    50% {
      transform: translateY(-6px);
    }
  }
</style>
