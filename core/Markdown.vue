/*
 * Author: Mele Nicolo' Emanuele
 * Date: May 23, 2026
 * License: MIT
 * Description: Lightweight Vue 3 wrapper around MdPreview from md-editor-v3, supporting dark/light theming via Pinia and full LaTeX and Mermaid diagram rendering.
 */

<template>
  <div
    ref="previewContainerRef"
    class="markdown-preview-container"
    @click="handleContentClick"
  >
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
  </div>
</template>

<script>
import { computed, ref } from 'vue';
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

  emits: ['external-link-click'],

  setup(props, { emit }) {
    const genericStore = useGenericStore();
    const { theme: globalTheme } = storeToRefs(genericStore);
    const previewContainerRef = ref(null);

    /**
     * @desc Maps the global Pinia theme string to the MdPreview-compatible theme value.
     * @return {String}
     */
    const resolvedEditorTheme = computed(() =>
      globalTheme.value === 'dark' ? 'dark' : 'light'
    );

    /**
     * @desc Converts a raw string into a lowercase hyphenated slug, matching the same logic md-editor-v3 uses for heading IDs.
     * @param rawText {String}
     * @return {String}
     */
    function slugifyHeadingText(rawText) {
      return rawText
        .trim()
        .toLowerCase()
        .replace(/\s+/g, '-')
        .replace(/[^\w-]/g, '');
    }

    /**
     * @desc Searches all headings inside the preview container and returns the first one whose slugified text matches the given slug.
     * @param slug {String}
     * @return {Element|null}
     */
    function findHeadingBySlug(slug) {
      const container = previewContainerRef.value;
      if (!container) return null;

      const allHeadings = Array.from(container.querySelectorAll('h1, h2, h3, h4, h5, h6'));

      return allHeadings.find((headingElement) => {
        const headingSlug = slugifyHeadingText(headingElement.textContent);
        return headingSlug === slug;
      }) ?? null;
    }

    /**
     * @desc Extracts the raw hash fragment from an href string, stripping the leading '#'.
     * @param anchorHref {String}
     * @return {String|null}
     */
    function extractAnchorSlug(anchorHref) {
      if (!anchorHref) return null;
      const hashIndex = anchorHref.indexOf('#');
      if (hashIndex === -1) return null;
      return decodeURIComponent(anchorHref.slice(hashIndex + 1)) || null;
    }

    /**
     * @desc Intercepts clicks inside the preview, scrolling to the matched heading for hash links and emitting external ones.
     * @param clickEvent {MouseEvent}
     * @return {void}
     */
    function handleContentClick(clickEvent) {
      const clickedAnchor = clickEvent.target.closest('a');
      if (!clickedAnchor) return;

      const href = clickedAnchor.getAttribute('href');
      if (!href) return;

      const slug = extractAnchorSlug(href);

      if (slug) {
        clickEvent.preventDefault();

        const matchedHeading = findHeadingBySlug(slug);
        if (matchedHeading) {
          matchedHeading.scrollIntoView({ behavior: 'smooth', block: 'start' });
        }
        return;
      }

      emit('external-link-click', { href, event: clickEvent });
    }

    return {
      previewContainerRef,
      resolvedEditorTheme,
      handleContentClick,
    };
  },
};
</script>

<style scoped>
.markdown-preview-container {
  width: 100%;
}

.markdown-preview-wrapper {
  width: 100%;
  background: transparent;
}

.markdown-preview-wrapper :deep(.md-editor-preview-wrapper),
.markdown-preview-wrapper :deep(.md-editor-preview) {
  background: transparent;
}

.markdown-preview-wrapper :deep(.md-editor-preview p),
.markdown-preview-wrapper :deep(.md-editor-preview h1),
.markdown-preview-wrapper :deep(.md-editor-preview h2),
.markdown-preview-wrapper :deep(.md-editor-preview h3),
.markdown-preview-wrapper :deep(.md-editor-preview h4),
.markdown-preview-wrapper :deep(.md-editor-preview h5),
.markdown-preview-wrapper :deep(.md-editor-preview h6),
.markdown-preview-wrapper :deep(.md-editor-preview li),
.markdown-preview-wrapper :deep(.md-editor-preview td),
.markdown-preview-wrapper :deep(.md-editor-preview th),
.markdown-preview-wrapper :deep(.md-editor-preview blockquote),
.markdown-preview-wrapper :deep(.md-editor-preview strong),
.markdown-preview-wrapper :deep(.md-editor-preview em),
.markdown-preview-wrapper :deep(.md-editor-preview span) {
  color: var(--foreground);
}
.markdown-preview-wrapper :deep(.md-editor-preview a),
.markdown-preview-wrapper :deep(.md-editor-preview a:hover) {
  color: var(--primary) !important;
  cursor: pointer;
}
.markdown-preview-wrapper :deep(.md-editor-preview blockquote) {
  border-left-color: var(--secondary) !important;
}


.markdown-preview-wrapper :deep(.md-editor-preview a) {
  cursor: pointer;
}
</style>