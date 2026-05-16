/*
 * Author: Mele Nicolo' Emanuele
 * Date: 2026-05-14
 * License: MIT
 * Description: Text editor with optional live markdown preview, code highlighting, and submit action.
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
          <div
            class="text-editor-rendered"
            v-html="renderedContent"
          />
        </div>
      </template>
    </div>
    <div class="text-editor-footer">
      <Button
        @click="onSubmit"
      >
        <span class="material-icons-round">check</span>
        <span>Submit</span>
      </Button>
    </div>
  </div>
</template>

<script>
import Button from 'primevue/button'

const CDN = {
  marked: 'https://cdn.jsdelivr.net/npm/marked@12.0.0/marked.min.js',
  highlight: 'https://cdn.jsdelivr.net/npm/highlight.js@11.9.0/highlight.min.js',
}

const loadedScripts = {}

/**
 * @param src String
 * @return Promise
 * @desc Fetches external script as text and injects it inline.
 */
function loadScript(src) {
  if (loadedScripts[src]) return loadedScripts[src]
  loadedScripts[src] = fetch(src)
    .then((res) => {
      if (!res.ok) throw new Error(`Failed to fetch: ${src}`)
      return res.text()
    })
    .then((code) => {
      const el = document.createElement('script')
      el.textContent = code
      document.head.appendChild(el)
    })
  return loadedScripts[src]
}

export default {
  name: 'TextEditor',

  components: { Button },

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

  data() {
    return {
      markedInstance: null,
      hlInstance: null,
      loadError: null,
    }
  },

  computed: {
    /**
     * @param void
     * @return String
     * @desc Renders modelValue markdown to HTML with syntax highlighting.
     */
    renderedContent() {
      if (!this.modelValue || !this.markedInstance) return this.modelValue || ''

      try {
        const renderer = new this.markedInstance.Renderer()

        renderer.code = function (code, lang) {
          code = String(code)
          const langStr = typeof lang === 'string' ? lang : (lang?.lang || '')
          const validLang = langStr && this.hlInstance?.getLanguage(langStr) ? langStr : null
          let highlighted

          if (this.highlight && this.hlInstance && validLang) {
            highlighted = this.hlInstance.highlight(code, { language: validLang }).value
          } else {
            highlighted = this.escapeHtml(code)
          }

          const langLabel = langStr
            ? `<span class="te-code-lang">${this.escapeHtml(langStr)}</span>`
            : ''

          return `<div class="te-code-block"><div class="te-code-header">${langLabel}</div><pre class="te-pre${validLang ? ` language-${langStr}` : ''}"><code class="te-code${validLang ? ` hljs language-${validLang}` : ''}">${highlighted}</code></pre></div>`
        }.bind(this)

        this.markedInstance.setOptions({ renderer })
        return this.markedInstance.parse(this.modelValue)
      } catch (e) {
        this.loadError = e.message
        return `<p class="te-error">${this.escapeHtml(e.message)}</p>`
      }
    },
  },

  async mounted() {
    try {
      await loadScript(CDN.marked)
      this.markedInstance = window.marked

      if (this.highlight) {
        await loadScript(CDN.highlight)
        this.hlInstance = window.hljs
      }
    } catch (e) {
      this.loadError = e.message
    }
  },

  methods: {
    /**
     * @param str String
     * @return String
     * @desc Escapes HTML special characters.
     */
    escapeHtml(str) {
      return str
        .replace(/&/g, '&amp;')
        .replace(/</g, '&lt;')
        .replace(/>/g, '&gt;')
        .replace(/"/g, '&quot;')
        .replace(/'/g, '&#039;')
    },

    /**
     * @param e Event
     * @return void
     * @desc Emits update:modelValue on textarea input.
     */
    onInput(e) {
      this.$emit('update:modelValue', e.target.value)
    },

    /**
     * @param void
     * @return void
     * @desc Emits onsubmit with the current editor content.
     */
    onSubmit() {
      this.$emit('onsubmit', this.modelValue)
    },
  },
}
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

.text-editor-rendered {
  font-size: 0.9375rem;
  line-height: 1.7;
}

.text-editor-rendered :deep(p) {
  margin: 0 0 1em;
}

.text-editor-rendered :deep(h1),
.text-editor-rendered :deep(h2),
.text-editor-rendered :deep(h3),
.text-editor-rendered :deep(h4),
.text-editor-rendered :deep(h5),
.text-editor-rendered :deep(h6) {
  margin: 1.5em 0 0.5em;
  font-weight: 600;
  line-height: 1.3;
}

.text-editor-rendered :deep(h1) { font-size: 1.5rem; }
.text-editor-rendered :deep(h2) { font-size: 1.25rem; }
.text-editor-rendered :deep(h3) { font-size: 1.125rem; }
.text-editor-rendered :deep(h4) { font-size: 1rem; }
.text-editor-rendered :deep(h5) { font-size: 0.875rem; }
.text-editor-rendered :deep(h6) { font-size: 0.8125rem; }

.text-editor-rendered :deep(ul),
.text-editor-rendered :deep(ol) {
  margin: 0 0 1em;
  padding-left: 1.5em;
}

.text-editor-rendered :deep(li) {
  margin: 0.3em 0;
}

.text-editor-rendered :deep(blockquote) {
  margin: 1em 0;
  padding: 0.6em 1.1rem;
  border-left: 3px solid var(--primary);
  background: color-mix(in srgb, var(--primary) 6%, transparent);
  color: color-mix(in srgb, var(--foreground) 80%, transparent);
  border-radius: 0 6px 6px 0;
}

.text-editor-rendered :deep(blockquote p) {
  margin: 0;
}

.text-editor-rendered :deep(code):not(.te-code) {
  font-family: 'SF Mono', 'Fira Code', 'Cascadia Code', monospace;
  font-size: 0.875em;
  padding: 0.15em 0.4em;
  border-radius: 4px;
  background: color-mix(in srgb, var(--foreground) 8%, transparent);
}

.text-editor-rendered :deep(.te-code-block) {
  margin: 1em 0;
  border-radius: 10px;
  overflow: hidden;
  border: 1px solid color-mix(in srgb, var(--foreground) 10%, transparent);
}

.text-editor-rendered :deep(.te-code-header) {
  display: flex;
  align-items: center;
  padding: 0.4rem 0.75rem;
  background: color-mix(in srgb, var(--foreground) 7%, transparent);
  border-bottom: 1px solid color-mix(in srgb, var(--foreground) 8%, transparent);
  min-height: 2rem;
}

.text-editor-rendered :deep(.te-code-lang) {
  font-family: 'SF Mono', 'Fira Code', monospace;
  font-size: 0.7rem;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: color-mix(in srgb, var(--foreground) 55%, transparent);
  font-weight: 600;
}

.text-editor-rendered :deep(.te-pre) {
  margin: 0;
  padding: 1rem;
  overflow-x: auto;
  background: color-mix(in srgb, var(--foreground) 4%, transparent);
}

.text-editor-rendered :deep(.te-code) {
  font-family: 'SF Mono', 'Fira Code', 'Cascadia Code', monospace;
  font-size: 0.8125rem;
  line-height: 1.65;
  background: transparent !important;
  padding: 0;
}

.text-editor-rendered :deep(.te-error) {
  color: var(--error, #ef4444);
  padding: 0.75rem 1rem;
  background: color-mix(in srgb, var(--error) 10%, transparent);
  border-radius: 8px;
  border: 1px solid color-mix(in srgb, var(--error) 30%, transparent);
  font-size: 0.875em;
}

.text-editor-rendered :deep(table) {
  width: 100%;
  margin: 1em 0;
  border-collapse: collapse;
}

.text-editor-rendered :deep(th),
.text-editor-rendered :deep(td) {
  padding: 0.65em 0.85em;
  border: 1px solid color-mix(in srgb, var(--foreground) 12%, transparent);
  text-align: left;
}

.text-editor-rendered :deep(th) {
  background: color-mix(in srgb, var(--foreground) 6%, transparent);
  font-weight: 600;
}

.text-editor-rendered :deep(tr:nth-child(even) td) {
  background: color-mix(in srgb, var(--foreground) 2.5%, transparent);
}

.text-editor-rendered :deep(hr) {
  margin: 2em 0;
  border: none;
  border-top: 1px solid color-mix(in srgb, var(--foreground) 12%, transparent);
}

.text-editor-footer {
  display: flex;
  justify-content: flex-end;
  padding: 0.6rem 0.75rem;
  background: color-mix(in srgb, var(--foreground) 5%, transparent);
  border-top: 1px solid color-mix(in srgb, var(--foreground) 10%, transparent);
}
</style>
