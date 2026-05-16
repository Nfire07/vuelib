/*
 * Author: Mele Nicolo' Emanuele
 * Date: 2026-05-04
 * License: MIT
 * Description: Animation wrapper with preset transitions for fade, slide, scale, and flip effects
 */
<template>
  <Transition
    :name="transitionName"
    :mode="mode"
    :appear="appear"
    :duration="duration"
    v-on="jsHooks"
  >
    <slot />
  </Transition>
</template>

<script>
const PRESETS = new Set(['fade', 'slide-up', 'slide-down', 'slide-left', 'slide-right', 'scale', 'scale-fade', 'flip-x', 'flip-y'])

export default {
  name: 'Animate',

  props: {
    name: {
      type: String,
      default: 'fade',
      validator: (v) => PRESETS.has(v) || v.startsWith('custom-'),
    },

    mode: {
      type: String,
      default: 'out-in',
      validator: (v) => ['out-in', 'in-out', 'default'].includes(v),
    },

    appear: {
      type: Boolean,
      default: false,
    },

    duration: {
      type: [Number, Object],
      default: null,
    },

    delay: {
      type: Number,
      default: 0,
    },

    easing: {
      type: String,
      default: 'ease',
    },

    distance: {
      type: [String, Number],
      default: 16,
    },

    onBeforeEnter: { type: Function, default: null },
    onEnter:       { type: Function, default: null },
    onAfterEnter:  { type: Function, default: null },
    onBeforeLeave: { type: Function, default: null },
    onLeave:       { type: Function, default: null },
    onAfterLeave:  { type: Function, default: null },
  },

  computed: {
    transitionName() {
      return `an-${this.name}`;
    },

    distancePx() {
      const d = this.distance;
      return typeof d === 'number' ? `${d}px` : d;
    },

    cssVars() {
      return {
        '--an-delay':    `${this.delay}ms`,
        '--an-easing':   this.easing,
        '--an-distance': this.distancePx,
      };
    },

    jsHooks() {
      const hooks = {};
      if (this.onBeforeEnter) hooks['before-enter'] = this.onBeforeEnter;
      if (this.onEnter)       hooks['enter']        = this.onEnter;
      if (this.onAfterEnter)  hooks['after-enter']  = this.onAfterEnter;
      if (this.onBeforeLeave) hooks['before-leave'] = this.onBeforeLeave;
      if (this.onLeave)       hooks['leave']        = this.onLeave;
      if (this.onAfterLeave)  hooks['after-leave']  = this.onAfterLeave;
      return hooks;
    },
  },
}
</script>

<style>
:root {
  --an-duration-enter: 220ms;
  --an-duration-leave: 180ms;
  --an-delay:    0ms;
  --an-easing:   ease;
  --an-distance: 16px;
}

[class*="an-"][class*="-enter-active"],
[class*="an-"][class*="-leave-active"] {
  transition-duration: var(--an-duration-enter);
  transition-timing-function: var(--an-easing);
  transition-delay: var(--an-delay);
}

[class*="an-"][class*="-leave-active"] {
  transition-duration: var(--an-duration-leave);
}

.an-fade-enter-active,
.an-fade-leave-active {
  transition-property: opacity;
}

.an-fade-enter-from,
.an-fade-leave-to {
  opacity: 0;
}

.an-slide-up-enter-active,
.an-slide-up-leave-active {
  transition-property: opacity, transform;
}

.an-slide-up-enter-from {
  opacity: 0;
  transform: translateY(var(--an-distance));
}

.an-slide-up-leave-to {
  opacity: 0;
  transform: translateY(calc(var(--an-distance) * -1));
}

.an-slide-down-enter-active,
.an-slide-down-leave-active {
  transition-property: opacity, transform;
}

.an-slide-down-enter-from {
  opacity: 0;
  transform: translateY(calc(var(--an-distance) * -1));
}

.an-slide-down-leave-to {
  opacity: 0;
  transform: translateY(var(--an-distance));
}

.an-slide-left-enter-active,
.an-slide-left-leave-active {
  transition-property: opacity, transform;
}

.an-slide-left-enter-from {
  opacity: 0;
  transform: translateX(var(--an-distance));
}

.an-slide-left-leave-to {
  opacity: 0;
  transform: translateX(calc(var(--an-distance) * -1));
}

.an-slide-right-enter-active,
.an-slide-right-leave-active {
  transition-property: opacity, transform;
}

.an-slide-right-enter-from {
  opacity: 0;
  transform: translateX(calc(var(--an-distance) * -1));
}

.an-slide-right-leave-to {
  opacity: 0;
  transform: translateX(var(--an-distance));
}

.an-scale-enter-active,
.an-scale-leave-active {
  transition-property: opacity, transform;
}

.an-scale-enter-from {
  opacity: 0;
  transform: scale(0.92);
}

.an-scale-leave-to {
  opacity: 0;
  transform: scale(1.04);
}

.an-scale-fade-enter-active,
.an-scale-fade-leave-active {
  transition-property: opacity, transform;
}

.an-scale-fade-enter-from,
.an-scale-fade-leave-to {
  opacity: 0;
  transform: scale(0.96);
}

.an-flip-x-enter-active,
.an-flip-x-leave-active {
  transition-property: opacity, transform;
  backface-visibility: hidden;
  perspective: 600px;
}

.an-flip-x-enter-from {
  opacity: 0;
  transform: rotateX(-90deg);
}

.an-flip-x-leave-to {
  opacity: 0;
  transform: rotateX(90deg);
}

.an-flip-y-enter-active,
.an-flip-y-leave-active {
  transition-property: opacity, transform;
  backface-visibility: hidden;
  perspective: 600px;
}

.an-flip-y-enter-from {
  opacity: 0;
  transform: rotateY(-90deg);
}

.an-flip-y-leave-to {
  opacity: 0;
  transform: rotateY(90deg);
}
</style>
