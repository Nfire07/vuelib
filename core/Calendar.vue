/*
 * Author: Mele Nicolo' Emanuele
 * Date: September 17, 2026
 * License: MIT
 * Description: Calendar component with month navigation, event display, and task management
 */
<template>
  <div class="calendar">
    <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet" />

    <div class="header">
      <button class="nav-btn" @click="prevMonth" :aria-label="lang.calendar.prevMonth">
        <span class="material-icons">chevron_left</span>
      </button>
      <h2>{{ monthYear }}</h2>
      <button class="nav-btn" @click="nextMonth" :aria-label="lang.calendar.nextMonth">
        <span class="material-icons">chevron_right</span>
      </button>
    </div>

    <div class="weekdays">
      <div v-for="day in weekdays" :key="day" class="weekday">{{ day }}</div>
    </div>

    <div class="days">
      <div
        v-for="(day, index) in days"
        :key="day.date ? day.date.toISOString() : 'empty-' + index"
        class="day"
        :class="{ 'day--active': day.date && getEventsForDay(day.date).length > 0 }"
        @click="day.date && openModal(day.date)"
      >
        <div v-if="day.date" :class="{ 'today-wrapper': isToday(day.date) }">
          {{ day.day }}
        </div>
        <div v-if="day.date && getEventsForDay(day.date).length > 0" class="event-dots">
          <span
            v-for="(dayEvent, eventIndex) in getEventsForDay(day.date).slice(0, 3)"
            :key="eventIndex"
            class="event-dot"
            :class="{ 'event-dot--google': isGoogleEvent(dayEvent) }"
          ></span>
        </div>
      </div>
    </div>

    <Transition name="modal-fade">
      <div v-if="modalOpen" class="modal-overlay" @click.self="closeModal">
        <div class="modal">
          <div class="modal-header">
            <div class="modal-title-group">
              <span class="modal-date-label">{{ formatModalDate(selectedDate) }}</span>
              <h3 class="modal-title">{{ lang.calendar.tasksTitle }}</h3>
            </div>
            <button class="modal-close" @click="closeModal">
              <span class="material-icons">close</span>
            </button>
          </div>

          <div class="modal-body">
             <div class="events-list" v-if="selectedDayEvents.length > 0">
               <div v-for="(event, i) in selectedDayEvents" :key="i" class="event-item">
                  <div class="event-info">
                    <span class="event-title">{{ event.name || event.title }}</span>
                    <span v-if="event.time" class="event-time">{{ formatTime(event.time) }}</span>
                    <span v-if="event.location" class="event-location">
                      <span class="material-icons event-location-icon">place</span>
                      {{ event.location }}
                    </span>
                    <span v-if="event.description" class="event-description">{{ event.description }}</span>
                  </div>
                  <span v-if="isGoogleEvent(event)" class="event-badge">
                    {{ lang.calendar.googleBadge }}
                  </span>
                  <button class="event-delete" @click="deleteEvent(event)" :aria-label="lang.calendar.deleteTask">
                    <span class="material-icons">delete</span>
                  </button>
                </div>
             </div>
            <div v-else class="no-events">
              <span class="material-icons no-events-icon">event_available</span>
              <p>{{ lang.calendar.noTasks }}</p>
            </div>

            <div class="form-divider-section">
              <span class="divider-text">{{ lang.calendar.addTask }}</span>
            </div>

    <DynamicForm
               :fields="resolvedFormFields"
               :submitText="lang.calendar.addTask"
               :loading="formLoading"
               @submit-form="handleFormSubmit"
             />
          </div>
        </div>
      </div>
    </Transition>
  </div>
</template>

<script>
import DynamicForm from './DynamicForm.vue';
import { mapState } from 'pinia';
import { useGenericStore } from '@/stores/generic';

import it from '@/locales/it.json';
import en from '@/locales/en.json';

export default {
  name: "Calendar",

  components: {
    DynamicForm,
  },
  emits: ["submit-form", "on-delete", "month-change"],

  data() {
    return {
      currentDate: new Date(),
      modalOpen: false,
      selectedDate: null,
    };
  },

  computed: {
    ...mapState(useGenericStore, ['language']),

    /**
     * @param void
     * @return Object
     * @desc Returns localized text based on current language
     */
    lang() {
      return this.language === 'en' ? en : it;
    },

    /**
     * @param void
     * @return String
     * @desc Returns locale string based on current language
     */
    currentLocale() {
      return this.language === 'en' ? 'en-US' : 'it-IT';
    },

    /**
     * @param void
     * @return Array
     * @desc Returns array of weekday names from localized strings
     */
    weekdays() {
      return this.lang.calendar.weekdays;
    },

    /**
     * @param void
     * @return String
     * @desc Returns formatted month and year string
     */
    monthYear() {
      return this.currentDate.toLocaleDateString(this.currentLocale, {
        month: "long",
        year: "numeric",
      });
    },

    /**
     * @param void
     * @return Array
     * @desc Returns array of day objects for current month view
     */
    days() {
      const year = this.currentDate.getFullYear();
      const month = this.currentDate.getMonth();
      const firstDay = new Date(year, month, 1);
      const lastDay = new Date(year, month + 1, 0);
      const result = [];
      let startDay = firstDay.getDay();
      startDay = startDay === 0 ? 6 : startDay - 1;
      for (let i = 0; i < startDay; i++) result.push({ day: "", date: null });
      for (let i = 1; i <= lastDay.getDate(); i++) {
        result.push({ day: i, date: new Date(year, month, i) });
      }
      return result;
    },

    /**
     * @param void
     * @return Array
     * @desc Returns events for the currently selected day
     */
    selectedDayEvents() {
      if (!this.selectedDate) return [];
      return this.getEventsForDay(this.selectedDate);
    },

    /**
     * @param void
     * @return Array
     * @desc Returns localized default form fields or prop-provided fields
     */
    resolvedFormFields() {
      if (this.formFields !== null) {
        return this.formFields;
      }
      return [
        { type: 'divider', name: 'divider', label: this.lang.calendar.addTask },
        { name: "name", label: this.lang.calendar.name, type: "text", required: true, placeholder: this.lang.calendar.placeholderName, fullWidth: true },
        { name: "time", label: this.lang.calendar.time, type: "time", required: true, fullWidth: true },
        { name: "description", label: this.lang.calendar.description, type: "textarea", required: true, placeholder: this.lang.calendar.placeholderDescription, fullWidth: true },
      ];
    },
  },

  props: {
    events: {
      type: Array,
      default: () => [],
    },
    formFields: {
      type: Array,
      default: null,
    },
    submitText: {
      type: String,
      default: "Add Task",
    },
    onAddEvent: {
      type: Function,
      default: null,
    },
    formLoading: {
      type: Boolean,
      default: false,
    },
  },


  methods: {
    /**
     * @param void
     * @return void
     * @desc Navigates to previous month and notifies the parent view
     */
    prevMonth() {
      this.currentDate = new Date(this.currentDate.getFullYear(), this.currentDate.getMonth() - 1, 1);
      this.$emit("month-change", this.currentDate);
    },

    /**
     * @param void
     * @return void
     * @desc Navigates to next month and notifies the parent view
     */
    nextMonth() {
      this.currentDate = new Date(this.currentDate.getFullYear(), this.currentDate.getMonth() + 1, 1);
      this.$emit("month-change", this.currentDate);
    },

    /**
     * @param date Date
     * @return Boolean
     * @desc Checks if given date is today
     */
    isToday(date) {
      if (!date) return false;
      return date.toDateString() === new Date().toDateString();
    },

    /**
     * @param date Date
     * @return Array
     * @desc Returns events matching the given date
     */
    getEventsForDay(date) {
      if (!date) return [];
      const dateStr = this.formatDate(date);
      return this.events.filter((e) => e.date === dateStr);
    },

    /**
     * @param event Object Event object with source information
     * @return Boolean
     * @desc Flags events that originate from the Google Calendar API
     */
    isGoogleEvent(event) {
      return Boolean(event) && (event._source === 'google' || event.isGoogle === true);
    },

    /**
     * @param date Date
     * @return void
     * @desc Opens modal for selected date
     */
    openModal(date) {
      this.selectedDate = date;
      this.modalOpen = true;
    },

    /**
     * @param void
     * @return void
     * @desc Closes the event modal
     */
    closeModal() {
      this.modalOpen = false;
      this.selectedDate = null;
    },

    /**
     * @param date Date
     * @return String
     * @desc Formats date for modal display with full weekday and date
     */
    formatModalDate(date) {
      if (!date) return "";
      return date.toLocaleDateString(this.currentLocale, { 
        weekday: "long", 
        day: "numeric", 
        month: "long", 
        year: "numeric" 
      });
    },

    /**
     * @param date Date
     * @return String
     * @desc Returns date as "YYYY-MM-DD" string in local time
     */
    formatDate(date) {
      if (!date) return "";
      const year = date.getFullYear();
      const month = String(date.getMonth() + 1).padStart(2, '0');
      const day = String(date.getDate()).padStart(2, '0');
      return `${year}-${month}-${day}`;
    },

    /**
     * @param time String
     * @return String
     * @desc Formats time from "HH:MM:SS" to "HH:MM"
     */
    formatTime(time) {
      if (!time) return '';
      return time.length > 5 ? time.substring(0, 5) : time;
    },

    /**
     * @param formData Object
     * @return void
     * @desc Handles form submission for adding new event
     */
    handleFormSubmit(formData) {
      const payload = { ...formData, date: this.formatDate(this.selectedDate) };
      if (this.onAddEvent) {
        this.onAddEvent(payload);
      }
      this.$emit("submit-form", payload);
      this.closeModal();
    },

    deleteEvent(event) {
      this.$emit("on-delete", event);
    },
  },
};
</script>

<style scoped>
.calendar {
  width: 90vw;
  font-family: var(--font-family);
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}

.header h2 {
  font-size: 1.1rem;
  font-weight: 600;
  color: var(--foreground);
  text-transform: capitalize;
}

.weekdays,
.days {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
}

.weekday {
  text-align: center;
  padding: 10px 20px;
  font-size: 0.75rem;
  font-weight: 600;
  color: color-mix(in srgb, var(--foreground) 50%, transparent);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.day {
  text-align: center;
  padding: 12px 20px;
  min-height: 80px;
  cursor: pointer;
  border-radius: 8px;
  transition: background 0.15s ease;
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
}

.day:hover {
  background: color-mix(in srgb, var(--foreground) 8%, transparent);
}

.day--active {
  background: color-mix(in srgb, var(--secondary) 8%, transparent);
}

.day--active:hover {
  background: color-mix(in srgb, var(--secondary) 15%, transparent);
}

.today-wrapper {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--secondary);
  color: var(--background);
  font-weight: 700;
  font-size: 0.9rem;
}

.event-dots {
  display: flex;
  gap: 3px;
  justify-content: center;
}

.event-dot {
  width: 5px;
  height: 5px;
  border-radius: 50%;
  background: var(--secondary);
}

.event-dot--google {
  background: #4285F4;
}

.nav-btn {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  border: 1px solid color-mix(in srgb, var(--foreground) 60%, transparent);
  background: color-mix(in srgb, var(--foreground) 30%, transparent);
  color: var(--foreground);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s ease;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.08);
}

.nav-btn .material-icons {
  font-size: 22px;
}

.nav-btn:hover {
  background: color-mix(in srgb, var(--foreground) 50%, transparent);
  border-color: color-mix(in srgb, var(--foreground) 70%, transparent);
}

.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.45);
  backdrop-filter: blur(4px);
  z-index: 1000;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 16px;
}

.modal {
  background: var(--background);
  border-radius: 16px;
  width: 100%;
  max-width: 480px;
  max-height: 90vh;
  overflow-y: auto;
  box-shadow: 0 24px 60px rgba(0, 0, 0, 0.25);
}

.modal-header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  padding: 24px 24px 16px;
  border-bottom: 1px solid color-mix(in srgb, var(--foreground) 12%, transparent);
  position: sticky;
  top: 0;
  background: var(--background);
  border-radius: 16px 16px 0 0;
  z-index: 1;
}

.modal-title-group {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.modal-date-label {
  font-size: 0.75rem;
  color: color-mix(in srgb, var(--foreground) 55%, transparent);
  text-transform: capitalize;
  font-weight: 500;
}

.modal-title {
  font-size: 1.25rem;
  font-weight: 700;
  color: var(--foreground);
  margin: 0;
}

.modal-close {
  background: color-mix(in srgb, var(--foreground) 10%, transparent);
  border: none;
  width: 36px;
  height: 36px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  color: var(--foreground);
  transition: background 0.2s;
  flex-shrink: 0;
}

.modal-close:hover {
  background: color-mix(in srgb, var(--foreground) 20%, transparent);
}

.modal-close .material-icons {
  font-size: 20px;
}

.modal-body {
  padding: 20px 24px 24px;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.events-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.event-item {
  display: flex;
  align-items: flex-start;
  padding: 12px 14px;
  background: color-mix(in srgb, var(--foreground) 6%, transparent);
  border-radius: 10px;
  border-left: 3px solid var(--secondary);
}

.event-info {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.event-title {
  font-weight: 600;
  font-size: 0.9rem;
  color: var(--foreground);
}

.event-time {
  font-size: 0.78rem;
  color: var(--secondary);
  font-weight: 500;
}

.event-description {
  font-size: 0.8rem;
  color: color-mix(in srgb, var(--foreground) 65%, transparent);
  margin-top: 2px;
}

.event-location {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 0.78rem;
  color: color-mix(in srgb, var(--foreground) 55%, transparent);
}

.event-location-icon {
  font-size: 14px;
}

.event-badge {
  align-self: center;
  flex-shrink: 0;
  margin-left: 10px;
  font-size: 0.65rem;
  font-weight: 600;
  letter-spacing: 0.03em;
  text-transform: uppercase;
  color: #4285F4;
  background: color-mix(in srgb, #4285F4 12%, transparent);
  border: 1px solid color-mix(in srgb, #4285F4 35%, transparent);
  border-radius: 999px;
  padding: 3px 8px;
  white-space: nowrap;
}

.event-delete {
  background: transparent;
  border: none;
  color: color-mix(in srgb, var(--foreground) 40%, transparent);
  cursor: pointer;
  padding: 4px;
  border-radius: 4px;
  transition: all 0.2s;
  margin-left: auto;
  flex-shrink: 0;
}

.event-delete:hover {
  background: color-mix(in srgb, red 15%, transparent);
  color: red;
}

.event-delete .material-icons {
  font-size: 18px;
}

.no-events {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  padding: 20px;
  color: color-mix(in srgb, var(--foreground) 40%, transparent);
}

.no-events-icon {
  font-size: 32px;
}

.no-events p {
  font-size: 0.875rem;
  margin: 0;
}

.form-divider-section {
  display: flex;
  align-items: center;
  gap: 12px;
  margin: 4px 0;
}

.form-divider-section::before,
.form-divider-section::after {
  content: "";
  flex: 1;
  height: 1px;
  background: color-mix(in srgb, var(--foreground) 15%, transparent);
}

.divider-text {
  font-size: 0.75rem;
  font-weight: 600;
  color: color-mix(in srgb, var(--foreground) 50%, transparent);
  text-transform: uppercase;
  letter-spacing: 0.06em;
  white-space: nowrap;
}

.modal-fade-enter-active,
.modal-fade-leave-active {
  transition: opacity 0.25s ease;
}

.modal-fade-enter-from,
.modal-fade-leave-to {
  opacity: 0;
}

.modal-fade-enter-active .modal,
.modal-fade-leave-active .modal {
  transition: transform 0.25s ease;
}

.modal-fade-enter-from .modal {
  transform: scale(0.95) translateY(12px);
}

.modal-fade-leave-to .modal {
  transform: scale(0.95) translateY(12px);
}
</style>
