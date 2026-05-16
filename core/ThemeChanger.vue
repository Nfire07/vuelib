/*
 * Author: Mele Nicolo' Emanuele
 * Date: 2026-05-04
 * License: MIT
 * Description: Theme switcher component for toggling between light and dark themes
 */
<script>
import 'material-design-icons-iconfont/dist/material-design-icons.css'
import { mapState, mapActions } from 'pinia';
import { useGenericStore } from '@/stores/generic';
import it from '@/locales/it.json';
import en from '@/locales/en.json';

export default {
  name: 'ThemeSwitcher',
  
  computed: {
    ...mapState(useGenericStore, ['theme', 'language']),
    
    /**
     * @param void
     * @return Object
     * @desc Returns localized text based on current language
     */
    langText() {
      return this.language === 'en' ? en : it;
    }
  },
  
  methods: {
    ...mapActions(useGenericStore, ['setTheme'])
  }
}
</script>

<style scoped>
.theme-switcher {
  padding: 3px;
  width: fit-content;
  display: flex;
  border-radius: 9999px;
  box-shadow: rgba(0, 0, 0, 0.4) 0px 2px 4px, rgba(0, 0, 0, 0.3) 0px 7px 13px -3px, rgba(0, 0, 0, 0.2) 0px -3px 0px inset;
}

.theme-switcher_switch {
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: inherit;
  border: 0;
  background: none;
  cursor: pointer;
  color:var(--foreground);
}

.theme-switcher_switch.active {
  background-color: rgba(128, 128, 128, 0.35);
}

.theme-switcher_switch:hover > .icon {
  color: var(--foreground);
}

.theme-switcher_switch.active > .icon {
  color: var(--foreground);
}

.icon {
  font-size: 18px;
}
</style>

<template>
  <div role="radiogroup" class="theme-switcher">
    <button
      type="button"
      role="radio"
      class="theme-switcher_switch"
      :class="{ active: theme === 'light' }"
      :aria-label="langText.themeSwitcher?.light || 'Light theme'"
      :aria-checked="theme === 'light'"
      @click="setTheme('light')"
    >
      <span class="material-icons icon">light_mode</span>
    </button>

    <button
      type="button"
      role="radio"
      class="theme-switcher_switch"
      :class="{ active: theme === 'dark' }"
      :aria-label="langText.themeSwitcher?.dark || 'Dark theme'"
      :aria-checked="theme === 'dark'"
      @click="setTheme('dark')"
    >
      <span class="material-icons icon">dark_mode</span>
    </button>
  </div>
</template>
