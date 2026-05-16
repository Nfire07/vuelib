/*
 * Author: Mele Nicolo' Emanuele
 * Date: 2026-05-04
 * License: MIT
 * Description: Flexible layout component supporting flex and grid with responsive props
 */
<template>
  <component
    :is="tag"
    :style="computedStyle"
    :class="['layout', { 'layout--container': container }]"
  >
    <div v-if="container" class="layout__container">
      <slot />
    </div>
    <slot v-else />
  </component>
</template>

<script>
const UNITLESS = new Set(['flex', 'order', 'flexGrow', 'flexShrink', 'gridColumn', 'gridRow', 'zIndex', 'opacity'])

/**
 * @param val Number or String
 * @param prop String
 * @return String or undefined
 * @desc Converts numeric values to px for CSS, except unitless properties
 */
function unit(val, prop) {
  if (val === null || val === undefined || val === '') return undefined
  if (typeof val === 'number') return UNITLESS.has(prop) ? val : `${val}px`
  return val
}

/**
 * @param val Number or String
 * @return String or undefined
 * @desc Resolves grid template columns value
 */
function resolveColumns(val) {
  if (!val && val !== 0) return undefined
  if (typeof val === 'number') return `repeat(${val}, 1fr)`
  return val
}

/**
 * @param val Number or String
 * @return String or undefined
 * @desc Resolves grid template rows value
 */
function resolveRows(val) {
  if (!val && val !== 0) return undefined
  if (typeof val === 'number') return `repeat(${val}, 1fr)`
  return val
}

/**
 * @param val String or Array
 * @return String or undefined
 * @desc Resolves grid template areas value
 */
function resolveAreas(val) {
  if (!val) return undefined
  if (Array.isArray(val)) return val.map(r => `"${r}"`).join(' ')
  return val
}

/**
 * @param type String
 * @param inline Boolean
 * @return String
 * @desc Determines display property based on type and inline settings
 */
function resolveDisplay(type, inline) {
  const t = (type || '').toLowerCase().trim()
  const isGrid = t === 'grid' || t === 'inline-grid' || /^\d+$/.test(t) || t.includes('fr') || t.includes('repeat(') || t.includes('minmax(')
  if (inline) return isGrid ? 'inline-grid' : 'inline-flex'
  return isGrid ? 'grid' : 'flex'
}

/**
 * @param display String
 * @return Boolean
 * @desc Checks if display value is a grid layout
 */
function isGridDisplay(display) {
  return display === 'grid' || display === 'inline-grid'
}

export default {
  name: 'Layout',

  props: {
    tag: {
      type: String,
      default: 'div',
    },

    type: {
      type: String,
      default: 'flex',
    },

    inline: {
      type: Boolean,
      default: false,
    },

    container: {
      type: Boolean,
      default: false,
    },

    containerMaxWidth: {
      type: [String, Number],
      default: '1400px',
    },

    containerPadding: {
      type: String,
      default: '0 1.5rem',
    },

    columns: {
      type: [String, Number],
      default: null,
    },

    rows: {
      type: [String, Number],
      default: null,
    },

    areas: {
      type: [String, Array],
      default: null,
    },

    autoFill: {
      type: Boolean,
      default: false,
    },

    autoFit: {
      type: Boolean,
      default: false,
    },

    minWidth: {
      type: String,
      default: null,
    },

    direction: {
      type: String,
      default: 'row',
    },

    wrap: {
      type: [Boolean, String],
      default: true,
    },

    gap: {
      type: [String, Number],
      default: null,
    },

    rowGap: {
      type: [String, Number],
      default: null,
    },

    columnGap: {
      type: [String, Number],
      default: null,
    },

    align: {
      type: String,
      default: null,
    },

    alignContent: {
      type: String,
      default: null,
    },

    justify: {
      type: String,
      default: null,
    },

    justifyItems: {
      type: String,
      default: null,
    },

    place: {
      type: String,
      default: null,
    },

    placeItems: {
      type: String,
      default: null,
    },

    placeContent: {
      type: String,
      default: null,
    },

    center: {
      type: Boolean,
      default: false,
    },

    width: {
      type: String,
      default: null,
    },

    height: {
      type: String,
      default: null,
    },

    minHeight: {
      type: String,
      default: null,
    },

    maxWidth: {
      type: [String, Number],
      default: null,
    },

    maxHeight: {
      type: [String, Number],
      default: null,
    },

    padding: {
      type: String,
      default: null,
    },

    margin: {
      type: String,
      default: null,
    },

    grow: {
      type: [Boolean, Number],
      default: null,
    },

    shrink: {
      type: [Boolean, Number],
      default: null,
    },

    basis: {
      type: String,
      default: null,
    },

    flex: {
      type: String,
      default: null,
    },

    position: {
      type: String,
      default: null,
    },

    overflow: {
      type: String,
      default: null,
    },

    overflowX: {
      type: String,
      default: null,
    },

    overflowY: {
      type: String,
      default: null,
    },

    zIndex: {
      type: Number,
      default: null,
    },

    background: {
      type: String,
      default: null,
    },

    borderRadius: {
      type: [String, Number],
      default: null,
    },

    dense: {
      type: Boolean,
      default: false,
    },

    fullHeight: {
      type: Boolean,
      default: false,
    },

    fullWidth: {
      type: Boolean,
      default: false,
    },

    stack: {
      type: Boolean,
      default: false,
    },

    reverse: {
      type: Boolean,
      default: false,
    },
  },

  computed: {
    computedStyle() {
      const style = {}

      const display = resolveDisplay(this.type, this.inline)
      style.display = display
      const isGrid = isGridDisplay(display)

      if (isGrid) {
        if (this.autoFill && this.minWidth) {
          style.gridTemplateColumns = `repeat(auto-fill, minmax(${this.minWidth}, 1fr))`
        } else if (this.autoFit && this.minWidth) {
          style.gridTemplateColumns = `repeat(auto-fit, minmax(${this.minWidth}, 1fr))`
        } else if (this.columns) {
          style.gridTemplateColumns = resolveColumns(this.columns)
        }

        if (this.rows) {
          style.gridTemplateRows = resolveRows(this.rows)
        }

        if (this.areas) {
          style.gridTemplateAreas = resolveAreas(this.areas)
        }

        if (this.dense) {
          style.gridAutoFlow = 'dense'
        }

        if (this.justifyItems) style.justifyItems = this.justifyItems
        if (this.placeItems) style.placeItems = this.placeItems
        if (this.placeContent) style.placeContent = this.placeContent
        if (this.place) {
          style.placeItems = this.place
          style.placeContent = this.place
        }
      } else {
        const stackDir = this.stack ? 'column' : this.direction
        const map = { col: 'column', column: 'column', stack: 'column', row: 'row' }
        const base = map[stackDir] || stackDir || 'row'
        style.flexDirection = this.reverse ? `${base}-reverse` : base

        if (this.flex) {
          style.flex = this.flex
        } else {
          if (this.grow !== null) style.flexGrow = this.grow === true ? 1 : this.grow === false ? 0 : this.grow
          if (this.shrink !== null) style.flexShrink = this.shrink === true ? 1 : this.shrink === false ? 0 : this.shrink
          if (this.basis) style.flexBasis = this.basis
        }

        const wrapVal = typeof this.wrap === 'string' ? this.wrap : this.wrap ? 'wrap' : 'nowrap'
        style.flexWrap = this.reverse && wrapVal === 'wrap' ? 'wrap-reverse' : wrapVal
      }

      if (this.gap !== null) style.gap = unit(this.gap)
      if (this.rowGap !== null) style.rowGap = unit(this.rowGap)
      if (this.columnGap !== null) style.columnGap = unit(this.columnGap)

      if (this.align) style.alignItems = this.align
      if (this.alignContent) style.alignContent = this.alignContent
      if (this.justify) style.justifyContent = this.justify

      if (this.center) {
        style.marginInline = 'auto'
        if (!this.maxWidth) style.maxWidth = '100%'
      }

      if (this.width) style.width = this.width
      else if (this.fullWidth) style.width = '100%'

      if (this.height) style.height = this.height
      else if (this.fullHeight) style.height = '100%'

      if (this.minHeight) style.minHeight = this.minHeight
      if (this.maxWidth !== null) style.maxWidth = unit(this.maxWidth)
      if (this.maxHeight !== null) style.maxHeight = unit(this.maxHeight)

      if (this.padding) style.padding = this.padding
      if (this.margin) style.margin = this.margin
      if (this.position) style.position = this.position
      if (this.overflow) style.overflow = this.overflow
      if (this.overflowX) style.overflowX = this.overflowX
      if (this.overflowY) style.overflowY = this.overflowY
      if (this.zIndex !== null) style.zIndex = this.zIndex
      if (this.background) style.background = this.background
      if (this.borderRadius !== null) style.borderRadius = unit(this.borderRadius)

      return style
    },
  },
}
</script>

<style scoped>
.layout {
  box-sizing: border-box;
}

.layout--container {
  width: 100%;
}

.layout__container {
  width: 100%;
  max-width: v-bind('typeof containerMaxWidth === "number" ? containerMaxWidth + "px" : containerMaxWidth');
  padding: v-bind('containerPadding');
  margin-inline: auto;
  box-sizing: border-box;
}
</style>
