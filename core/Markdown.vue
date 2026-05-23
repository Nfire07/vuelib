/*
 * Author: Mele Nicolo' Emanuele
 * Date: May 23, 2026
 * License: MIT
 * Description: Lightweight Vue 3 wrapper around MdPreview from md-editor-v3, supporting dark/light theming via Pinia and full LaTeX and Mermaid diagram rendering.
 */

<template>
  <MdPreview
    :model-value="modelValue"
    :theme="resolvedEditorTheme"
    :preview-theme="previewTheme"
    :language="editorLanguage"
    :show-code-row-number="showLineNumbers"
    :no-mermaid="disableMermaid"
    :no-katex="disableMath"
    class="markdown-preview-wrapper"
  />
</template>

<script>
import { computed } from 'vue';
import { storeToRefs } from 'pinia';
import { MdPreview } from 'md-editor-v3';
import 'md-editor-v3/lib/preview.css';
import { useGenericStore } from '@/stores/generic';

export default {
  name: 'Markdown',

  components: {
    MdPreview,
  },

  props: {
    modelValue: {
      type: String,
      default: '',
    },
    previewTheme: {
      type: String,
      default: 'default',
      validator: (value) => ['default', 'github', 'vuepress', 'mk-cute', 'smart-blue', 'cyanosis'].includes(value),
    },
    editorLanguage: {
      type: String,
      default: 'en-US',
    },
    showLineNumbers: {
      type: Boolean,
      default: false,
    },
    disableMermaid: {
      type: Boolean,
      default: false,
    },
    disableMath: {
      type: Boolean,
      default: false,
    },
  },

  setup() {
    const genericStore = useGenericStore();
    const { theme: globalTheme } = storeToRefs(genericStore);

    /**
     * @desc Maps the global Pinia theme string to the MdPreview-compatible theme value.
     * @return {String}
     */
    const resolvedEditorTheme = computed(() =>
      globalTheme.value === 'dark' ? 'dark' : 'light'
    );

    return {
      resolvedEditorTheme,
    };
  },
};
</script>

<style scoped>
.markdown-preview-wrapper {
  width: 100%;
  background: transparent;
}
</style>