/*
 * Author: Mele Nicolo' Emanuele
 * Date: 2026-05-13
 * License: MIT
 * Description: Markdown component with syntax highlighting, frontmatter parsing, __#slug__ anchor navigation support, and KaTeX math rendering.
 */
<template>
  <component
    :is="tag"
    class="markdown-content"
    :class="[
      `markdown--${variant}`,
      {
        'markdown--prose': prose,
        'markdown--no-margin': noMargin,
        'markdown--compact': compact,
      },
    ]"
    :style="contentStyle"
    v-html="renderedContent"
  />
</template>

<script>
import { mapState } from 'pinia';
import { useGenericStore } from '@/stores/generic';

const CDN = {
  marked:    'https://cdn.jsdelivr.net/npm/marked@12.0.0/marked.min.js',
  highlight: 'https://cdn.jsdelivr.net/npm/highlight.js@11.9.0/highlight.min.js',
  katex:     'https://cdn.jsdelivr.net/npm/katex@0.16.10/katex.min.js',
}

const CSS = {
  github:       'https://cdn.jsdelivr.net/npm/highlight.js@11.9.0/styles/github.min.css',
  githubDark:   'https://cdn.jsdelivr.net/npm/highlight.js@11.9.0/styles/github-dark.min.css',
  atomOneDark:  'https://cdn.jsdelivr.net/npm/highlight.js@11.9.0/styles/atom-one-dark.min.css',
  atomOneLight: 'https://cdn.jsdelivr.net/npm/highlight.js@11.9.0/styles/atom-one-light.min.css',
  nord:         'https://cdn.jsdelivr.net/npm/highlight.js@11.9.0/styles/nord.min.css',
  monokai:      'https://cdn.jsdelivr.net/npm/highlight.js@11.9.0/styles/monokai.min.css',
  dracula:      'https://cdn.jsdelivr.net/npm/highlight.js@11.9.0/styles/dracula.min.css',
  vitesse:      'https://cdn.jsdelivr.net/npm/highlight.js@11.9.0/styles/base16/material.min.css',
  katex:        'https://cdn.jsdelivr.net/npm/katex@0.16.10/katex.min.css',
}

const loadedScripts = {}
const loadedStyles  = {}

const FRONTMATTER_PATTERN  = /^---\s*\n([\s\S]*?)\n---\s*\n?/
const ANCHOR_LINK_PATTERN  = /\[([^\]]+)\]\(__#([^)]+)__\)/g

/**
 * @param src String
 * @return Promise
 * @desc Fetches external script as text and injects it inline to avoid MIME type sniff blocks.
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

/**
 * @param href String
 * @return void
 * @desc Loads external stylesheet and caches it to avoid duplicates.
 */
function loadStyle(href) {
  if (loadedStyles[href]) return
  loadedStyles[href] = true
  if (document.querySelector(`link[href="${href}"]`)) return
  const el = document.createElement('link')
  el.rel = 'stylesheet'
  el.href = href
  document.head.appendChild(el)
}

/**
 * @param str String
 * @return String
 * @desc Escapes HTML special characters to prevent XSS injection.
 */
function escapeHtml(str) {
  return str
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#039;')
}

/**
 * @param raw String
 * @return String
 * @desc Strips YAML frontmatter block from the top of a markdown string.
 */
function stripFrontmatter(raw) {
  return raw.replace(FRONTMATTER_PATTERN, '')
}

/**
 * @param text String
 * @return String
 * @desc Converts heading text to a URL slug matching the __#slug__ TOC format, stripping dots.
 */
function slugify(text) {
  return text
    .toLowerCase()
    .replace(/\./g, '')
    .replace(/[^\w]+/g, '-')
    .replace(/^-+|-+$/g, '')
}

/**
 * @param source String
 * @return String
 * @desc Replaces [text](__#slug__) occurrences with button HTML before markdown parsing.
 */
function preprocessAnchorLinks(source) {
  return source.replace(ANCHOR_LINK_PATTERN, (match, text, slug) => {
    return `<button type="button" data-anchor="${slug}" class="md-anchor-btn">${text}</button>`
  })
}

/**
 * @param source String
 * @return String
 * @desc Extracts and escapes math expressions before markdown parsing to prevent
 *       marked from mangling LaTeX syntax (e.g. underscores, backslashes).
 *       Display math  $$…$$  and inline math  $…$  are replaced with
 *       placeholder tokens that are restored after rendering.
 */
function extractMath(source) {
  const blocks = []

  /**
   * @param expr String
   * @param display Boolean
   * @return String
   * @desc Stores a math expression and returns a unique placeholder token.
   */
  function store(expr, display) {
    const idx = blocks.push({ expr, display }) - 1
    return `@@MATH${idx}@@`
  }

  // Display math: $$...$$
  source = source.replace(/\$\$([\s\S]+?)\$\$/g, (_, expr) => store(expr, true))

  // Inline math: $...$ — skip $$ by requiring no adjacent $
  source = source.replace(/(?<!\$)\$(?!\$)((?:[^$\n]|\\.)+?)\$(?!\$)/g, (_, expr) => store(expr, false))

  return { source, blocks }
}

/**
 * @param html String
 * @param blocks Array<{ expr: String, display: Boolean }>
 * @param katex Object
 * @return String
 * @desc Replaces @@MATHn@@ placeholders in the rendered HTML with KaTeX output.
 */
function restoreMath(html, blocks, katex) {
  return html.replace(/@@MATH(\d+)@@/g, (_, idx) => {
    const { expr, display } = blocks[Number(idx)]
    try {
      return katex.renderToString(expr, {
        displayMode: display,
        throwOnError: false,
        output: 'html',
      })
    } catch (e) {
      return `<span class="md-math-error" title="${escapeHtml(e.message)}">${escapeHtml(expr)}</span>`
    }
  })
}

export default {
  name: 'Markdown',

  props: {
    modelValue: {
      type: String,
      default: '',
    },

    tag: {
      type: String,
      default: 'div',
    },

    variant: {
      type: String,
      default: 'default',
      validator: (v) => ['default', 'minimal', 'docs', 'blog', 'chat'].includes(v),
    },

    prose: {
      type: Boolean,
      default: true,
    },

    breaks: {
      type: Boolean,
      default: true,
    },

    gfm: {
      type: Boolean,
      default: true,
    },

    highlight: {
      type: Boolean,
      default: true,
    },

    highlightTheme: {
      type: String,
      default: 'auto',
      validator: (v) => ['auto', 'github', 'githubDark', 'atomOneDark', 'atomOneLight', 'nord', 'monokai', 'dracula', 'vitesse'].includes(v),
    },

    showCopyButton: {
      type: Boolean,
      default: true,
    },

    showLanguageLabel: {
      type: Boolean,
      default: true,
    },

    lineNumbers: {
      type: Boolean,
      default: false,
    },

    linkTarget: {
      type: String,
      default: '_blank',
      validator: (v) => ['_blank', '_self', '_parent', '_top', ''].includes(v),
    },

    linkRel: {
      type: String,
      default: 'noopener noreferrer',
    },

    headingIds: {
      type: Boolean,
      default: true,
    },

    headingPrefix: {
      type: String,
      default: '',
    },

    math: {
      type: Boolean,
      default: true,
    },

    compact: {
      type: Boolean,
      default: false,
    },

    noMargin: {
      type: Boolean,
      default: false,
    },

    fontSize: {
      type: String,
      default: null,
    },

    maxWidth: {
      type: String,
      default: null,
    },

    customStyles: {
      type: Object,
      default: null,
    },

    copyLabel: {
      type: String,
      default: 'Copy',
    },

    copiedLabel: {
      type: String,
      default: 'Copied!',
    },
  },

  emits: ['loaded', 'error', 'link-click'],

  data() {
    return {
      markedInstance: null,
      hlInstance:     null,
      katexInstance:  null,
      loadError:      null,
      isLoaded:       false,
    };
  },

  computed: {
    ...mapState(useGenericStore, ['language', 'theme']),

    /**
     * @param void
     * @return String
     * @desc Resolves highlight theme based on prop or current app theme.
     */
    resolvedTheme() {
      if (this.highlightTheme !== 'auto') return this.highlightTheme
      return this.theme === 'dark' ? 'atomOneDark' : 'github'
    },

    /**
     * @param void
     * @return String
     * @desc Returns the preprocessed source: frontmatter stripped, math extracted,
     *       and anchor links converted to button elements.
     */
    parsedSource() {
      return preprocessAnchorLinks(stripFrontmatter(this.modelValue || ''))
    },

    /**
     * @param void
     * @return String
     * @desc Renders markdown to HTML using marked and highlight.js.
     *       When math is enabled, LaTeX expressions are extracted before parsing
     *       and restored as KaTeX HTML afterwards.
     */
    renderedContent() {
      if (!this.parsedSource || !this.markedInstance) return ''

      try {
        // Extract math before marked touches the source to protect LaTeX syntax
        const { source, blocks } = (this.math && this.katexInstance)
          ? extractMath(this.parsedSource)
          : { source: this.parsedSource, blocks: [] }

        const renderer = new this.markedInstance.Renderer()

        renderer.code = function(code, lang) {
          code = String(code)
          const langStr = typeof lang === 'string' ? lang : (lang?.lang || '')
          const validLang = langStr && this.hlInstance?.getLanguage(langStr) ? langStr : null
          let highlighted

          if (this.highlight && this.hlInstance && validLang) {
            highlighted = this.hlInstance.highlight(code, { language: validLang }).value
          } else {
            highlighted = escapeHtml(code)
          }

          if (this.lineNumbers) {
            const lines = highlighted.split('\n')
            const numbered = lines
              .map((line, i) => `<span class="md-line"><span class="md-line-num">${i + 1}</span>${line}</span>`)
              .join('\n')
            highlighted = `<span class="md-lines">${numbered}</span>`
          }

          const langLabel = this.showLanguageLabel && langStr
            ? `<span class="md-code-lang">${escapeHtml(langStr)}</span>`
            : ''

          const copyBtn = this.showCopyButton
            ? `<button class="md-copy-btn" data-code="${escapeHtml(code)}" type="button">${this.copyLabel}</button>`
            : ''

          const header = langLabel || copyBtn
            ? `<div class="md-code-header">${langLabel}${copyBtn}</div>`
            : ''

          return `<div class="md-code-block">${header}<pre class="md-pre${validLang ? ` language-${langStr}` : ''}"><code class="md-code${validLang ? ` hljs language-${validLang}` : ''}">${highlighted}</code></pre></div>`
        }.bind(this)

        renderer.link = (href, title, text) => {
          const titleAttr = title ? ` title="${escapeHtml(title)}"` : ''
          if (href && href.startsWith('#')) {
            return `<a href="${href}"${titleAttr} class="md-link md-link--anchor">${text}</a>`
          }
          const target = this.linkTarget ? ` target="${this.linkTarget}"` : ''
          const rel    = this.linkRel    ? ` rel="${this.linkRel}"`       : ''
          return `<a href="${href}"${titleAttr}${target}${rel} class="md-link">${text}</a>`
        }

        renderer.heading = (text, level) => {
          const slug   = `${this.headingPrefix}${slugify(text)}`
          const anchor = this.headingIds
            ? `<a class="md-anchor" href="#${slug}" aria-hidden="true">#</a>`
            : ''
          return `<h${level} id="${slug}" class="md-heading md-h${level}">${anchor}${text}</h${level}>`
        }

        renderer.image = (href, title, text) => {
          const t = title ? ` title="${escapeHtml(title)}"` : ''
          const a = text  ? ` alt="${escapeHtml(text)}"`    : ''
          return `<figure class="md-figure"><img src="${href}"${a}${t} class="md-img" loading="lazy" />${text ? `<figcaption class="md-caption">${escapeHtml(text)}</figcaption>` : ''}</figure>`
        }

        this.markedInstance.setOptions({ breaks: this.breaks, gfm: this.gfm, renderer })
        let html = this.markedInstance.parse(source)

        // Restore math placeholders with KaTeX-rendered HTML
        if (blocks.length) {
          html = restoreMath(html, blocks, this.katexInstance)
        }

        return html
      } catch (e) {
        this.loadError = e.message
        this.$emit('error', e)
        return `<p class="md-error">Error parsing markdown: ${escapeHtml(e.message)}</p>`
      }
    },

    /**
     * @param void
     * @return Object
     * @desc Computes inline style object for the content wrapper element.
     */
    contentStyle() {
      const style = {}
      if (this.fontSize) style.fontSize = this.fontSize
      if (this.maxWidth) style.maxWidth = this.maxWidth
      return { ...style, ...(this.customStyles || {}) }
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

      if (this.math) {
        await loadScript(CDN.katex)
        this.katexInstance = window.katex
        loadStyle(CSS.katex)
      }

      this.applyHighlightTheme()
      this.isLoaded = true
      this.$emit('loaded')
    } catch (e) {
      this.loadError = e.message
      this.$emit('error', e)
    }

    this.$el.addEventListener('click', this.handleClick)
  },

  beforeUnmount() {
    this.$el?.removeEventListener('click', this.handleClick)
  },

  watch: {
    resolvedTheme() {
      this.applyHighlightTheme()
    },

    modelValue() {
      if (this.markedInstance && !this.isLoaded) {
        this.isLoaded = true
      }
    },
  },

  methods: {
    /**
     * @param void
     * @return void
     * @desc Applies the resolved highlight.js theme stylesheet to the document.
     */
    applyHighlightTheme() {
      if (!this.highlight) return
      const href = CSS[this.resolvedTheme]
      if (href) loadStyle(href)
    },

    /**
     * @param e Event
     * @return void
     * @desc Handles clicks for copy buttons, anchor links, and external markdown links.
     */
    handleClick(e) {
      const copyButton = e.target.closest('.md-copy-btn')
      if (copyButton) {
        const code = copyButton.dataset.code || ''
        navigator.clipboard?.writeText(code).then(() => {
          const previousLabel = copyButton.textContent
          copyButton.textContent = this.copiedLabel
          copyButton.classList.add('md-copy-btn--copied')
          setTimeout(() => {
            copyButton.textContent = previousLabel
            copyButton.classList.remove('md-copy-btn--copied')
          }, 2000)
        })
        return
      }

      const anchorButton = e.target.closest('.md-anchor-btn')
      if (anchorButton) {
        const targetId      = anchorButton.dataset.anchor
        const targetElement = document.getElementById(targetId)
        if (targetElement) {
          targetElement.scrollIntoView({ behavior: 'smooth', block: 'start' })
        }
        return
      }

      const anchorLink = e.target.closest('.md-link--anchor')
      if (anchorLink) {
        e.preventDefault()
        const targetId      = anchorLink.getAttribute('href').slice(1)
        const targetElement = document.getElementById(targetId)
        if (targetElement) {
          targetElement.scrollIntoView({ behavior: 'smooth', block: 'start' })
        }
        return
      }

      const externalLink = e.target.closest('.md-link:not(.md-link--anchor)')
      if (externalLink) {
        this.$emit('link-click', { href: externalLink.href, event: e })
      }
    },
  },
};
</script>

<style scoped>
.markdown-content {
  font-family: var(--font-family);
  color: var(--foreground);
  line-height: 1.7;
  box-sizing: border-box;
}

.markdown--default { font-size: 1rem; }
.markdown--minimal { font-size: 0.875rem; }
.markdown--docs    { font-size: 0.9375rem; }
.markdown--blog    { font-size: 1.0625rem; }
.markdown--chat    { font-size: 0.9375rem; line-height: 1.6; }

.markdown--compact :deep(p),
.markdown--compact :deep(ul),
.markdown--compact :deep(ol),
.markdown--compact :deep(blockquote) {
  margin-bottom: 0.5em;
}

.markdown--no-margin :deep(> *:first-child) { margin-top: 0; }
.markdown--no-margin :deep(> *:last-child)  { margin-bottom: 0; }

.markdown--prose :deep(.md-heading) {
  margin: 1.5em 0 0.5em;
  font-weight: 600;
  line-height: 1.3;
  color: var(--foreground);
  position: relative;
  scroll-margin-top: 80px;
}

.markdown--prose :deep(.md-h1) {
  font-size: 2rem;
  padding-bottom: 0.5em;
  border-bottom: 2px solid color-mix(in srgb, var(--foreground) 12%, transparent);
}

.markdown--prose :deep(.md-h2) {
  font-size: 1.5rem;
  padding-bottom: 0.25em;
  border-bottom: 1px solid color-mix(in srgb, var(--foreground) 8%, transparent);
}

.markdown--prose :deep(.md-h3) { font-size: 1.25rem; }
.markdown--prose :deep(.md-h4) { font-size: 1.125rem; }
.markdown--prose :deep(.md-h5) { font-size: 1rem; }

.markdown--prose :deep(.md-h6) {
  font-size: 0.875rem;
  color: color-mix(in srgb, var(--foreground) 65%, transparent);
}

.markdown--prose :deep(.md-anchor) {
  position: absolute;
  left: -1.2em;
  opacity: 0;
  text-decoration: none;
  color: var(--primary, currentColor);
  font-weight: 400;
  transition: opacity 0.15s;
}

.markdown--prose :deep(.md-heading:hover .md-anchor) {
  opacity: 0.5;
}

.markdown--prose :deep(p) {
  margin: 0 0 1em;
}

.markdown--prose :deep(.md-link) {
  color: var(--primary);
  text-decoration: none;
  border-bottom: 1px solid color-mix(in srgb, var(--primary) 40%, transparent);
  transition: border-color 0.15s;
  cursor: pointer;
}

.markdown--prose :deep(.md-link:hover) {
  border-color: var(--primary);
}

.markdown--prose :deep(.md-anchor-btn) {
  display: inline-flex;
  align-items: center;
  background: color-mix(in srgb, var(--primary) 10%, transparent);
  border: 1px solid color-mix(in srgb, var(--primary) 30%, transparent);
  border-radius: 6px;
  padding: 0.2em 0.75em;
  margin: 0.15em 0;
  font: inherit;
  font-size: 0.875em;
  color: var(--primary);
  cursor: pointer;
  transition: background 0.15s, border-color 0.15s;
}

.markdown--prose :deep(.md-anchor-btn:hover) {
  background: color-mix(in srgb, var(--primary) 20%, transparent);
  border-color: var(--primary);
}

.markdown--prose :deep(strong) { font-weight: 600; }
.markdown--prose :deep(em)     { font-style: italic; }

.markdown--prose :deep(code):not(.md-code) {
  font-family: 'SF Mono', 'Fira Code', 'Cascadia Code', monospace;
  font-size: 0.875em;
  padding: 0.15em 0.4em;
  border-radius: 4px;
  background: color-mix(in srgb, var(--foreground) 8%, transparent);
}

.markdown--prose :deep(.md-code-block) {
  margin: 1em 0;
  border-radius: 10px;
  overflow: hidden;
  border: 1px solid color-mix(in srgb, var(--foreground) 10%, transparent);
}

.markdown--prose :deep(.md-code-header) {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0.4rem 0.75rem;
  background: color-mix(in srgb, var(--foreground) 7%, transparent);
  border-bottom: 1px solid color-mix(in srgb, var(--foreground) 8%, transparent);
  min-height: 2rem;
}

.markdown--prose :deep(.md-code-lang) {
  font-family: 'SF Mono', 'Fira Code', monospace;
  font-size: 0.7rem;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: color-mix(in srgb, var(--foreground) 55%, transparent);
  font-weight: 600;
}

.markdown--prose :deep(.md-copy-btn) {
  font-family: var(--font-family);
  font-size: 0.7rem;
  padding: 0.2em 0.6em;
  border-radius: 4px;
  border: 1px solid color-mix(in srgb, var(--foreground) 18%, transparent);
  background: transparent;
  color: color-mix(in srgb, var(--foreground) 60%, transparent);
  cursor: pointer;
  transition: all 0.15s;
}

.markdown--prose :deep(.md-copy-btn:hover) {
  background: color-mix(in srgb, var(--foreground) 8%, transparent);
  border-color: color-mix(in srgb, var(--foreground) 30%, transparent);
}

.markdown--prose :deep(.md-copy-btn--copied) {
  color: var(--success, #22c55e);
  border-color: var(--success, #22c55e);
}

.markdown--prose :deep(.md-pre) {
  margin: 0;
  padding: 1rem;
  overflow-x: auto;
  background: color-mix(in srgb, var(--foreground) 4%, transparent);
}

.markdown--prose :deep(.md-code) {
  font-family: 'SF Mono', 'Fira Code', 'Cascadia Code', monospace;
  font-size: 0.8125rem;
  line-height: 1.65;
  background: transparent !important;
  padding: 0;
}

.markdown--prose :deep(.md-lines) {
  display: table;
  width: 100%;
}

.markdown--prose :deep(.md-line) {
  display: table-row;
}

.markdown--prose :deep(.md-line-num) {
  display: table-cell;
  padding-right: 1.25em;
  text-align: right;
  user-select: none;
  color: color-mix(in srgb, var(--foreground) 30%, transparent);
  min-width: 2em;
  font-size: 0.8em;
}

.markdown--prose :deep(blockquote) {
  margin: 1em 0;
  padding: 0.6em 1.1rem;
  border-left: 3px solid var(--primary);
  background: color-mix(in srgb, var(--primary) 6%, transparent);
  color: color-mix(in srgb, var(--foreground) 80%, transparent);
  border-radius: 0 6px 6px 0;
}

.markdown--prose :deep(blockquote p) {
  margin: 0;
}

.markdown--prose :deep(ul),
.markdown--prose :deep(ol) {
  margin: 0 0 1em;
  padding-left: 1.5em;
}

.markdown--prose :deep(li) {
  margin: 0.3em 0;
}

.markdown--prose :deep(ul li::marker) {
  color: var(--primary);
}

.markdown--prose :deep(ol li::marker) {
  color: var(--primary);
  font-weight: 600;
}

.markdown--prose :deep(hr) {
  margin: 2em 0;
  border: none;
  border-top: 1px solid color-mix(in srgb, var(--foreground) 12%, transparent);
}

.markdown--prose :deep(.md-figure) {
  margin: 1.25em 0;
  text-align: center;
}

.markdown--prose :deep(.md-img) {
  max-width: 100%;
  height: auto;
  border-radius: 8px;
  display: block;
  margin: 0 auto;
}

.markdown--prose :deep(.md-caption) {
  margin-top: 0.4em;
  font-size: 0.8125rem;
  color: color-mix(in srgb, var(--foreground) 55%, transparent);
  font-style: italic;
}

.markdown--prose :deep(table) {
  width: 100%;
  margin: 1em 0;
  border-collapse: collapse;
}

.markdown--prose :deep(th),
.markdown--prose :deep(td) {
  padding: 0.65em 0.85em;
  border: 1px solid color-mix(in srgb, var(--foreground) 12%, transparent);
  text-align: left;
}

.markdown--prose :deep(th) {
  background: color-mix(in srgb, var(--foreground) 6%, transparent);
  font-weight: 600;
}

.markdown--prose :deep(tr:nth-child(even) td) {
  background: color-mix(in srgb, var(--foreground) 2.5%, transparent);
}

/* ── Math ────────────────────────────────────────────────────────────────── */

/*
 * Display math blocks: KaTeX wraps displayMode output in .katex-display.
 * Add vertical rhythm and horizontal scroll for wide equations.
 */
.markdown--prose :deep(.katex-display) {
  margin: 1.25em 0;
  overflow-x: auto;
  overflow-y: hidden;
  padding: 0.25em 0;
}

/*
 * Inherit the document font size so math scales correctly with variant classes
 * instead of rendering at KaTeX's default 1.21em standalone size.
 */
.markdown--prose :deep(.katex) {
  font-size: 1.05em;
}

/*
 * Fallback pill shown when KaTeX fails to parse an expression.
 */
.markdown--prose :deep(.md-math-error) {
  display: inline-block;
  font-family: 'SF Mono', 'Fira Code', 'Cascadia Code', monospace;
  font-size: 0.875em;
  padding: 0.15em 0.5em;
  border-radius: 4px;
  background: color-mix(in srgb, var(--error, #ef4444) 10%, transparent);
  border: 1px solid color-mix(in srgb, var(--error, #ef4444) 30%, transparent);
  color: var(--error, #ef4444);
  cursor: help;
}

/* ── Errors ──────────────────────────────────────────────────────────────── */

.markdown--prose :deep(.md-error) {
  color: var(--error, #ef4444);
  padding: 0.75rem 1rem;
  background: color-mix(in srgb, var(--error) 10%, transparent);
  border-radius: 8px;
  border: 1px solid color-mix(in srgb, var(--error) 30%, transparent);
  font-size: 0.875em;
}
</style>
