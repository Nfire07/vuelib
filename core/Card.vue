/*
 * Author: Mele Nicolo' Emanuele
 * Date: 2026-05-09
 * License: MIT
 * Description: A flexible card component with icon, title, description, border customization, scrollable items and modal
 */
<template>
  <component
    :is="tag"
    class="card"
    :class="cardClasses"
    :style="cardStyle"
    :tabindex="clickable ? 0 : undefined"
    @click="handleClick"
    @keydown.enter="handleClick"
  >
    <div v-if="image" class="card__image-wrapper">
      <img
        :src="image"
        :alt="imageAlt"
        class="card__image"
        :style="imageStyle"
        loading="lazy"
      />
    </div>
    
    <div class="card__body">
      <div v-if="icon" class="card__icon-wrapper">
        <span class="card__icon material-icons-round">{{ icon }}</span>
      </div>
      <h3 v-if="title" class="card__title">{{ title }}</h3>
      <p v-if="description" class="card__description">{{ description }}</p>
      
      <div v-if="items && items.length" class="card__items">
        <div
          v-for="item in items"
          :key="getItemKey(item)"
          class="card__item"
          @click.stop="handleItemClick(item)"
        >
          {{ getItemLabel(item) }}
        </div>
      </div>
      
      <slot />
      
      <div v-if="resolvedCta || $slots.footer" class="card__footer">
        <slot name="footer">
          <span class="card__cta">{{ resolvedCta }}</span>
        </slot>
      </div>
    </div>
  </component>
</template>

<script>
import { mapState } from 'pinia'
import { useGenericStore } from '@/stores/generic'
import en from '@/locales/en.json'
import it from '@/locales/it.json'

export default {
  name: 'Card',

  props: {
    tag: {
      type: String,
      default: 'div',
    },
    title: {
      type: String,
      default: null,
    },
    description: {
      type: String,
      default: null,
    },
    image: {
      type: String,
      default: null,
    },
    imageAlt: {
      type: String,
      default: '',
    },
    imageHeight: {
      type: [String, Number],
      default: null,
    },
    cta: {
      type: String,
      default: null,
    },
    ctaI18n: {
      type: String,
      default: 'readMore',
    },
    width: {
      type: [String, Number],
      default: null,
    },
    minWidth: {
      type: [String, Number],
      default: null,
    },
    maxWidth: {
      type: [String, Number],
      default: null,
    },
    height: {
      type: [String, Number],
      default: null,
    },
    minHeight: {
      type: [String, Number],
      default: null,
    },
    maxHeight: {
      type: [String, Number],
      default: null,
    },
    padding: {
      type: [String, Number],
      default: null,
    },
    clickable: {
      type: Boolean,
      default: false,
    },
    outlined: {
      type: Boolean,
      default: true,
    },
    elevated: {
      type: Boolean,
      default: false,
    },
    bordered: {
      type: Boolean,
      default: true,
    },
    icon: {
      type: String,
      default: null,
    },
    borderColor: {
      type: String,
      default: null,
    },
    borderWidth: {
      type: [String, Number],
      default: null,
    },
    borderStyle: {
      type: String,
      default: 'solid',
    },
    items: {
      type: Array,
      default: () => [],
    },
    itemKey: {
      type: String,
      default: 'id',
    },
    itemLabel: {
      type: String,
      default: 'label',
    },
  },

  emits: ['click', 'item-click'],

  computed: {
    ...mapState(useGenericStore, ['language']),

    lang() {
      return this.language === 'en' ? en : it
    },

    resolvedCta() {
      if (this.cta) return this.cta
      if (this.ctaI18n) {
        const keys = this.ctaI18n.split('.')
        let value = this.lang
        for (const key of keys) {
          value = value?.[key]
        }
        return value || null
      }
      return null
    },

    cardClasses() {
      return {
        'card--clickable': this.clickable,
        'card--outlined': this.outlined,
        'card--elevated': this.elevated,
        'card--bordered': this.bordered,
        'card--with-image': !!this.image,
      }
    },

    cardStyle() {
      const style = {}
      
      if (this.width) style.width = this.unit(this.width)
      if (this.minWidth) style.minWidth = this.unit(this.minWidth)
      if (this.maxWidth) style.maxWidth = this.unit(this.maxWidth)
      if (this.height) style.height = this.unit(this.height)
      if (this.minHeight) style.minHeight = this.unit(this.minHeight)
      if (this.maxHeight) style.maxHeight = this.unit(this.maxHeight)
      if (this.padding) style.padding = this.unit(this.padding)
      if (this.borderColor) style.borderColor = this.borderColor
      if (this.borderWidth) style.borderWidth = this.unit(this.borderWidth)
      if (this.borderStyle) style.borderStyle = this.borderStyle
      
      return style
    },

    imageStyle() {
      if (!this.imageHeight) return {}
      return { height: this.unit(this.imageHeight) }
    },
  },

  methods: {
    /**
     * @param val The value to convert to a CSS unit string
     * @return The value with 'px' appended if it's a number, otherwise the value itself
     * @desc Converts a numeric value to a pixel string, passes through string values
     */
    unit(val) {
      if (val === null || val === undefined) return undefined
      if (typeof val === 'number') return `${val}px`
      return val
    },

    /**
     * @param e The click event
     * @return void
     * @desc Handles card click events and emits click event if card is clickable
     */
    handleClick(e) {
      if (this.clickable) {
        this.$emit('click', e)
      }
    },

    /**
     * @param item The item that was clicked
     * @return void
     * @desc Handles item click by emitting item-click event
     */
    handleItemClick(item) {
      this.$emit('item-click', item)
    },

    /**
     * @param item The item to get the key for
     * @return The unique key for the item
     * @desc Extracts the unique key from an item using the itemKey prop
     */
    getItemKey(item) {
      return item[this.itemKey] || item.id || JSON.stringify(item)
    },

    /**
     * @param item The item to get the label for
     * @return The display label for the item
     * @desc Extracts the display label from an item using the itemLabel prop
     */
    getItemLabel(item) {
      return item[this.itemLabel] || item.label || item.name || 'Item'
    },
  },
}
</script>

<style scoped>
.card {
  display: flex;
  flex-direction: column;
  background: var(--background);
  border-radius: 12px;
  overflow: hidden;
  box-sizing: border-box;
  position: relative;
}

.card--outlined {
  border: 1.5px solid color-mix(in srgb, var(--foreground) 10%, transparent);
}

.card--bordered {
  border: 1.5px solid color-mix(in srgb, var(--foreground) 10%, transparent);
}

.card--clickable {
  cursor: pointer;
  transition: all 0.2s ease;
}

.card--clickable:hover {
  border-color: var(--primary);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px color-mix(in srgb, var(--foreground) 8%, transparent);
}

.card--clickable:focus {
  outline: 2px solid var(--primary);
  outline-offset: 2px;
}

.card--elevated {
  box-shadow: 
    0 2px 4px color-mix(in srgb, var(--foreground) 6%, transparent),
    0 8px 24px color-mix(in srgb, var(--foreground) 12%, transparent);
}

.card__image-wrapper {
  overflow: hidden;
}

.card__image {
  width: 100%;
  height: 180px;
  object-fit: cover;
  display: block;
}

.card__icon-wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 48px;
  height: 48px;
  border-radius: 12px;
  background: color-mix(in srgb, var(--primary) 10%, transparent);
  margin-bottom: 0.75rem;
}

.card__icon {
  font-size: 1.5rem;
  color: var(--primary);
}

.card__body {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  padding: 1.5rem;
}

.card__title {
  font-size: 1.1rem;
  font-weight: 600;
  color: var(--foreground);
  margin: 0;
}

.card__description {
  font-size: 0.875rem;
  color: color-mix(in srgb, var(--foreground) 60%, transparent);
  margin: 0;
  line-height: 1.5;
}

.card__items {
  max-height: 300px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  margin: 0.5rem 0;
}

.card__item {
  padding: 0.75rem;
  background: color-mix(in srgb, var(--foreground) 5%, transparent);
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.2s ease;
}

.card__item:hover {
  background: color-mix(in srgb, var(--primary) 10%, transparent);
}

.card__footer {
  margin-top: 0.5rem;
  padding-top: 0.75rem;
  border-top: 1px solid color-mix(in srgb, var(--foreground) 8%, transparent);
}

.card__cta {
  font-size: 0.8125rem;
  color: var(--primary);
  font-weight: 500;
}

</style>
