/*
 * Author: Mele Nicolo' Emanuele
 * Date: May 23, 2026
 * License: MIT
 * Description: Text editor with optional live markdown preview via MdPreview, code highlighting, and submit action.
 */
<template>
  <div class="text-editor">
    <div class="text-editor-body">
      <div class="text-editor-pane text-editor-pane--input">
        <textarea
          class="text-editor-textarea"
          :value="modelValue"
          :placeholder="placeholder"
          @input="onInput"
        />
      </div>
      <template v-if="isMarkdown">
        <div class="text-editor-divider" />
        <div class="text-editor-pane text-editor-pane--preview">
          <Markdown :model-value="modelValue" />
        </div>
      </template>
    </div>
    <div class="text-editor-footer">
      <Button @click="onSubmit">
        <span class="material-icons-round">check</span>
        <span>Submit</span>
      </Button>
    </div>
  </div>
</template>

<script>
import Button from 'primevue/button';
import Markdown from '@/lib/vuelib/core/Markdown.vue';

export default {
  name: 'TextEditor',

  components: {
    Button,
    Markdown,
  },

  props: {
    modelValue: {
      type: String,
      default: '',
    },
    placeholder: {
      type: String,
      default: 'Write your content here...',
    },
    highlight: {
      type: Boolean,
      default: true,
    },
    isMarkdown: {
      type: Boolean,
      default: false,
    },
  },

  emits: ['update:modelValue', 'onsubmit'],

  methods: {
    /**
     * @param inputEvent {Event}
     * @return {void}
     * @desc Emits update:modelValue with the current textarea value on every input event.
     */
    onInput(inputEvent) {
      this.$emit('update:modelValue', inputEvent.target.value);
    },

    /**
     * @return {void}
     * @desc Emits onsubmit with the current modelValue content.
     */
    onSubmit() {
      this.$emit('onsubmit', this.modelValue);
    },
  },
};
</script>

<style scoped>
.text-editor {
  display: flex;
  flex-direction: column;
  border: 1px solid color-mix(in srgb, var(--foreground) 12%, transparent);
  border-radius: 10px;
  overflow: hidden;
  font-family: var(--font-family);
  color: var(--foreground);
  background: var(--background);
}

.text-editor-body {
  display: flex;
  flex-direction: row;
  min-height: 300px;
  max-height: 70vh;
}

.text-editor-pane {
  flex: 1;
  padding: 1rem;
  overflow-y: auto;
}

.text-editor-pane--input {
  display: flex;
  flex-direction: column;
}

.text-editor-textarea {
  width: 100%;
  height: 100%;
  min-height: 260px;
  flex: 1;
  font-family: 'SF Mono', 'Fira Code', 'Cascadia Code', monospace;
  font-size: 0.8125rem;
  line-height: 1.65;
  border: none;
  outline: none;
  resize: none;
  background: transparent;
  color: var(--foreground);
}

.text-editor-textarea::placeholder {
  color: color-mix(in srgb, var(--foreground) 40%, transparent);
}

.text-editor-divider {
  width: 1px;
  background: color-mix(in srgb, var(--foreground) 10%, transparent);
  flex-shrink: 0;
}

.text-editor-footer {
  display: flex;
  justify-content: flex-end;
  padding: 0.6rem 0.75rem;
  background: color-mix(in srgb, var(--foreground) 5%, transparent);
  border-top: 1px solid color-mix(in srgb, var(--foreground) 10%, transparent);
}
</style>