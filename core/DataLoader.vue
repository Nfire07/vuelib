/*
 * Author: Mele Nicolo' Emanuele
 * Date: 2026-05-04
 * License: MIT
 * Description: Data loader with state management for loading, error, empty, and success states
 */
<template>
  <Transition name="dl-fade" mode="out-in">
    <div :key="state" class="data-loader">

      <div v-if="state === 'idle'" class="data-loader__idle">
        <slot name="idle" />
      </div>

      <div v-else-if="state === 'loading'" class="data-loader__loading">
        <template v-if="skeleton">
          <div
            v-for="i in skeletonRows"
            :key="i"
            class="skeleton-row"
            :style="{ animationDelay: (i - 1) * 80 + 'ms' }"
          >
            <div class="skeleton-block skeleton-block--avatar"></div>
            <div class="skeleton-lines">
              <div class="skeleton-block skeleton-block--line" style="width: 65%"></div>
              <div class="skeleton-block skeleton-block--line" style="width: 40%"></div>
            </div>
          </div>
        </template>
        <template v-else>
          <div class="spinner-wrapper">
            <div class="spinner"></div>
            <p class="state-text">{{ loadingText || lang.dataloader.loading }}</p>
          </div>
        </template>
      </div>

      <div v-else-if="state === 'error'" class="data-loader__error">
        <span class="state-icon material-icons-round">cloud_off</span>
        <p class="state-title">{{ errorTitle || lang.dataloader.errorTitle }}</p>
        <p class="state-message">{{ errorMessage || fetchError }}</p>
        <button
          v-if="retryable"
          type="button"
          class="retry-btn"
          @click="load()"
        >
          <span class="material-icons-round">refresh</span>
          {{ retryText || lang.dataloader.retry }}
        </button>
      </div>

      <div v-else-if="state === 'empty'" class="data-loader__empty">
        <span class="state-icon material-icons-round">{{ emptyIcon }}</span>
        <p class="state-title">{{ emptyTitle || lang.dataloader.emptyTitle }}</p>
        <p class="state-message">{{ emptyMessage || lang.dataloader.emptyMessage }}</p>
      </div>

      <div v-else-if="state === 'success'" class="data-loader__content">
        <slot :data="data" />
      </div>

    </div>
  </Transition>
</template>

<script>
import { mapState } from 'pinia';
import { useGenericStore } from '@/stores/generic';
import it from '@/locales/it.json';
import en from '@/locales/en.json';

export default {
  name: 'DataLoader',

  expose: ['load', 'state', 'data'],

  props: {
    url: {
      type: String,
      default: '',
    },
    fetchOptions: {
      type: Object,
      default: () => ({}),
    },
    actionFn: {
      type: Function,
      default: null,
    },
    actionPayload: {
      type: null,
      default: undefined,
    },
    immediate: {
      type: Boolean,
      default: true,
    },
    transformer: {
      type: Function,
      default: null,
    },
    isEmpty: {
      type: Function,
      default: null,
    },
    skeleton: {
      type: Boolean,
      default: false,
    },
    skeletonRows: {
      type: Number,
      default: 4,
    },
    loadingText: {
      type: String,
      default: '',
    },
    retryable: {
      type: Boolean,
      default: true,
    },
    retryText: {
      type: String,
      default: '',
    },
    errorTitle: {
      type: String,
      default: '',
    },
    errorMessage: {
      type: String,
      default: '',
    },
    emptyIcon: {
      type: String,
      default: 'inbox',
    },
    emptyTitle: {
      type: String,
      default: '',
    },
    emptyMessage: {
      type: String,
      default: '',
    },
  },

  emits: ['success', 'error', 'empty'],

  data() {
    return {
      state: this.immediate ? 'loading' : 'idle',
      data: null,
      fetchError: '',
      _abortController: null,
    };
  },

  computed: {
    ...mapState(useGenericStore, ['language']),
    lang() {
      return this.language === 'en' ? en : it;
    },
  },

  watch: {
    actionPayload(newVal, oldVal) {
      if (newVal !== oldVal) {
        this.load(newVal);
      }
    },
  },

  mounted() {
    if (this.immediate) {
      this.load();
    }
  },

  beforeUnmount() {
    if (this._abortController) {
      this._abortController.abort();
    }
  },

  methods: {
    /**
     * @param payload Any
     * @return void
     * @desc Loads data using actionFn or url with optional payload
     */
    async load(payload = undefined) {
      if (this._abortController) {
        this._abortController.abort();
      }
      this._abortController = new AbortController();

      this.state = 'loading';
      this.fetchError = '';

      try {
        let result;
        const payloadToUse = payload !== undefined ? payload : this.actionPayload;

        if (this.actionFn && typeof this.actionFn === 'function') {
          result = payloadToUse !== undefined
            ? await this.actionFn(payloadToUse)
            : await this.actionFn();
        } else if (this.url) {
          const response = await fetch(this.url, {
            ...this.fetchOptions,
            signal: this._abortController.signal,
          });

          if (!response.ok) {
            throw new Error(`${this.lang.dataloader.requestFailed} ${response.status}`);
          }

          result = await response.json();
        } else {
          throw new Error('No url or actionFn provided');
        }

        if (this.transformer) {
          result = this.transformer(result);
        }

        const empty = this.isEmpty
          ? this.isEmpty(result)
          : this.url
            ? Array.isArray(result)
              ? result.length === 0
              : result === null || result === undefined
            : false;

        if (empty) {
          this.data = result;
          this.state = 'empty';
          this.$emit('empty', result);
        } else {
          this.data = result;
          this.state = 'success';
          this.$emit('success', result);
        }
      } catch (err) {
        if (err.name === 'AbortError') return;

        this.fetchError = err.message || this.lang.dataloader.unknownError;
        this.state = 'error';
        this.$emit('error', this.fetchError);
      }
    },
  },
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/icon?family=Material+Icons+Round');
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;600&display=swap');

.dl-fade-enter-active,
.dl-fade-leave-active {
  transition: opacity 0.18s ease, transform 0.18s ease;
}
.dl-fade-enter-from {
  opacity: 0;
  transform: translateY(6px);
}
.dl-fade-leave-to {
  opacity: 0;
  transform: translateY(-4px);
}

.data-loader {
  font-family: var(--font-family);
  width: 100%;
}

.data-loader__loading,
.data-loader__error,
.data-loader__empty {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 3rem 1.5rem;
  gap: 0.6rem;
  text-align: center;
}

.skeleton-row {
  display: flex;
  align-items: center;
  gap: 0.85rem;
  padding: 0.65rem 0;
  width: 100%;
  animation: skeleton-fade-in 0.3s ease both;
}

@keyframes skeleton-fade-in {
  from { opacity: 0; transform: translateY(6px); }
  to   { opacity: 1; transform: translateY(0); }
}

.skeleton-lines {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 0.45rem;
}

.skeleton-block {
  background: linear-gradient(
    90deg,
    rgba(0, 0, 0, 0.08) 25%,
    rgba(0, 0, 0, 0.14) 50%,
    rgba(0, 0, 0, 0.08) 75%
  );
  background-size: 200% 100%;
  animation: skeleton-shimmer 1.4s infinite ease-in-out;
  border-radius: 6px;
}

.skeleton-block--avatar {
  width: 40px;
  height: 40px;
  border-radius: 10px;
  flex-shrink: 0;
}

.skeleton-block--line {
  height: 12px;
  border-radius: 4px;
}

@keyframes skeleton-shimmer {
  0%   { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}

.spinner-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

.spinner {
  width: 36px;
  height: 36px;
  border: 3px solid rgba(0, 0, 0, 0.12);
  border-top-color: var(--primary);
  border-radius: 50%;
  animation: spin 0.75s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.state-icon {
  font-size: 2.5rem;
  user-select: none;
}

.data-loader__error .state-icon {
  color: var(--error, #ef4444);
}

.data-loader__empty .state-icon {
  color: rgba(0, 0, 0, 0.3);
}

.state-title {
  margin: 0;
  font-size: 0.95rem;
  font-weight: 600;
  color: var(--foreground);
}

.state-message {
  margin: 0;
  font-size: 0.825rem;
  color: var(--foreground);
  max-width: 280px;
  line-height: 1.5;
}

.state-text {
  margin: 0;
  font-size: 0.825rem;
  color: var(--foreground);
}

.retry-btn {
  display: inline-flex;
  align-items: center;
  gap: 0.4rem;
  margin-top: 0.4rem;
  padding: 0.55rem 1.1rem;
  background: rgba(0, 0, 0, 0.06);
  border: 1px solid rgba(0, 0, 0, 0.25);
  border-radius: 8px;
  color: var(--primary);
  font-family: 'DM Sans', sans-serif;
  font-size: 0.85rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.18s, border-color 0.18s, transform 0.15s;
}

.retry-btn:hover {
  background: rgba(0, 0, 0, 0.12);
  border-color: var(--primary);
}

.retry-btn:active {
  transform: scale(0.97);
}

.retry-btn .material-icons-round {
  font-size: 1rem;
  user-select: none;
}

.data-loader__content {
  width: 100%;
}
</style>
