/*
 * Author: Mele Nicolo' Emanuele
 * Date: June 6, 2026
 * License: MIT
 * Description: OAuth provider button with branded styling and logos for Google and GitHub
 */
<script>
import { mapState } from 'pinia';
import { useGenericStore } from '@/stores/generic';
import it from '@/locales/it.json';
import en from '@/locales/en.json';

export default {
  name: 'OAuthButton',

  props: {
    provider: {
      type: String,
      required: true,
      validator: (value) => ['google', 'github'].includes(value)
    },
    label: {
      type: String,
      default: ''
    }
  },

  emits: ['click'],

  computed: {
    ...mapState(useGenericStore, ['language']),

    lang() {
      return this.language === 'en' ? en : it;
    },

    label() {
      if (this.$props.label) return this.$props.label;
      const labels = {
        google: this.lang.oauth?.google || 'Sign in with Google',
        github: this.lang.oauth?.github || 'Sign in with GitHub'
      };
      return labels[this.provider] || this.provider;
    }
  },

  methods: {
    handleClick(event) {
      this.$emit('click', event);
    }
  }
}
</script>

<template>
  <button
    type="button"
    class="oauth-button"
    :class="`oauth-button--${provider}`"
    @click="handleClick"
  >
    <span class="oauth-button__logo">
      <svg v-if="provider === 'google'" viewBox="0 0 48 48" width="20" height="20" xmlns="http://www.w3.org/2000/svg">
        <path fill="#EA4335" d="M24 9.5c3.54 0 6.71 1.22 9.21 3.6l6.85-6.85C35.9 2.38 30.47 0 24 0 14.62 0 6.51 5.38 2.56 13.22l7.98 6.19C12.43 13.72 17.74 9.5 24 9.5z"/>
        <path fill="#4285F4" d="M46.98 24.55c0-1.57-.15-3.09-.38-4.55H24v9.02h12.94c-.58 2.96-2.26 5.48-4.78 7.18l7.73 6c4.51-4.18 7.09-10.36 7.09-17.65z"/>
        <path fill="#FBBC05" d="M10.53 28.59A14.5 14.5 0 0 1 9.5 24c0-1.59.28-3.14.76-4.59l-7.98-6.19A23.99 23.99 0 0 0 0 24c0 3.77.87 7.35 2.56 10.56l7.97-5.97z"/>
        <path fill="#34A853" d="M24 48c6.48 0 11.93-2.13 15.89-5.81l-7.73-6c-2.15 1.45-4.92 2.3-8.16 2.3-6.26 0-11.57-4.22-13.47-9.91l-7.98 5.97C6.51 42.62 14.62 48 24 48z"/>
        <path fill="none" d="M0 0h48v48H0z"/>
      </svg>
      <svg v-else viewBox="0 0 16 16" width="20" height="20" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
        <path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/>
      </svg>
    </span>
    <span class="oauth-button__label">{{ label }}</span>
  </button>
</template>

<style scoped>
.oauth-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  width: 100%;
  max-width: 320px;
  padding: 12px 24px;
  border-radius: 8px;
  font-family: var(--font-family, 'DM Sans', sans-serif);
  font-size: 15px;
  font-weight: 500;
  cursor: pointer;
  transition:
    transform 0.15s ease,
    box-shadow 0.15s ease;
  border: none;
  outline: none;
}

.oauth-button:hover {
  transform: translateY(-1px);
}

.oauth-button:active {
  transform: translateY(0);
}

.oauth-button--google {
  background: #ffffff;
  color: #1f1f1f;
  border: 1px solid #dadce0;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.08);
}

.oauth-button--google:hover {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.12);
  background: #f8f9fa;
}

.oauth-button--github {
  background: #24292f;
  color: #ffffff;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
}

.oauth-button--github:hover {
  background: #1b1f23;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
}

.oauth-button__logo {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 20px;
  height: 20px;
  flex-shrink: 0;
}

.oauth-button__logo svg {
  width: 100%;
  height: 100%;
}

.oauth-button__label {
  white-space: nowrap;
}
</style>
