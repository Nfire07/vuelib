/*
 * Author: Mele Nicolo' Emanuele
 * Date: 2026-05-06
 * License: MIT
 * Description: Loader overlay component with spinner and optional text
 */
<template>
  <Transition name="loader-fade">
    <div v-if="loading" class="loader-overlay">
      <div class="loader-spinner"></div>
      <p v-if="text" class="loader-text">{{ text }}</p>
    </div>
  </Transition>
</template>

<script>
export default {
  name: 'Loader',

  props: {
    /**
     * @param loading Boolean
     * @return void
     * @desc Controls loader visibility
     */
    loading: {
      type: Boolean,
      default: false,
    },
    /**
     * @param text String
     * @return void
     * @desc Optional text displayed under spinner
     */
    text: {
      type: String,
      default: '',
    },
  },
}
</script>

<style scoped>
.loader-fade-enter-active,
.loader-fade-leave-active {
  transition: opacity 0.2s ease;
}
.loader-fade-enter-from,
.loader-fade-leave-to {
  opacity: 0;
}

.loader-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.45);
  backdrop-filter: blur(4px);
  z-index: 9999;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
}

.loader-spinner {
  width: 40px;
  height: 40px;
  border: 3px solid rgba(255, 255, 255, 0.3);
  border-top-color: var(--primary);
  border-radius: 50%;
  animation: loader-spin 0.8s linear infinite;
}

@keyframes loader-spin {
  to { transform: rotate(360deg); }
}

.loader-text {
  margin: 0;
  font-size: 0.825rem;
  color: var(--foreground);
  font-family: var(--font-family);
}
</style>
