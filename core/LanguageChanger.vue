/*
 * Author: Mele Nicolo' Emanuele
 * Date: 2026-05-04
 * License: MIT
 * Description: Language switcher with accessbility support for EN/IT toggle
 */
<template>
  <div class="lang-switcher" role="radiogroup">
    <button
      class="lang-btn"
      :class="{ active: language === 'en' }"
      role="radio"
      :aria-checked="language === 'en'"
      :aria-label="langText.languageChanger.english"
      @click="handleLangChange('en')"
    >
      EN
    </button>

    <button
      class="lang-btn"
      :class="{ active: language === 'it' }"
      role="radio"
      :aria-checked="language === 'it'"
      :aria-label="langText.languageChanger.italian"
      @click="handleLangChange('it')"
    >
      IT
    </button>
  </div>
</template>

<script>
import { mapState, mapActions } from 'pinia';
import { useGenericStore } from '@/stores/generic';
import it from '@/locales/it.json';
import en from '@/locales/en.json';

export default {
  name: 'LanguageChanger',

  computed: {
    ...mapState(useGenericStore, ['language']),
    
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
    ...mapActions(useGenericStore, ['changeLanguage']),

    /**
     * @param lang String
     * @return void
     * @desc Changes application language and emits change event
     */
    handleLangChange(l) {
      this.changeLanguage(l);
      this.$emit('change', l);
    }
  }
}
</script>

<style scoped>
.lang-switcher {
  display: flex;
  padding: 4px;
  border-radius: 9999px;
  box-shadow: rgba(0, 0, 0, 0.4) 0px 2px 4px, rgba(0, 0, 0, 0.3) 0px 7px 13px -3px, rgba(0, 0, 0, 0.2) 0px -3px 0px inset;
  gap: 4px;
}

.lang-btn {
  border: none;
  background: transparent;
  color: var(--foreground);
  font-weight: 600;
  font-size: 12px;
  letter-spacing: 0.1em;
  padding: 6px 10px;
  border-radius: 9999px;
  cursor: pointer;
  color:var(--foreground);
}

.lang-btn:hover {
  color: #323232;
}

.lang-btn.active {
  background-color: rgba(172, 172, 172, 0.35);
  color: var(--foreground);
  box-shadow: 0 2px 6px rgba(0,0,0,0.2);
}

</style>
