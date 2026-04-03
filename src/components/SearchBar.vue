<template>
  <div class="search-container">
    <div class="search-row">
      <div class="search-input-wrapper location-input">
        <svg class="search-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <circle cx="12" cy="12" r="10"/>
          <circle cx="12" cy="12" r="3"/>
          <line x1="12" y1="2" x2="12" y2="4"/>
          <line x1="12" y1="20" x2="12" y2="22"/>
          <line x1="2" y1="12" x2="4" y2="12"/>
          <line x1="20" y1="12" x2="22" y2="12"/>
        </svg>
        <input
          type="text"
          class="search-input"
          placeholder="Enter location or use Near Me..."
          :value="locationQuery"
          @input="$emit('update:locationQuery', $event.target.value)"
        />
        <button
          v-if="!locationQuery"
          class="near-me-btn"
          @click="nearMe"
          :disabled="detecting"
          title="Use my current location"
        >
          <template v-if="detecting">...</template>
          <template v-else>Near Me</template>
        </button>
        <button
          v-else
          class="clear-btn"
          @click="$emit('update:locationQuery', '')"
        >
          x
        </button>
      </div>
      <div class="search-input-wrapper">
        <svg class="search-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <circle cx="11" cy="11" r="8"/>
          <path d="m21 21-4.35-4.35"/>
        </svg>
        <input
          type="text"
          class="search-input"
          :placeholder="placeholder"
          :value="searchQuery"
          @input="$emit('update:searchQuery', $event.target.value)"
        />
        <button
          v-if="searchQuery"
          class="clear-btn"
          @click="$emit('update:searchQuery', '')"
        >
          x
        </button>
      </div>
    </div>
    <div class="category-filters">
      <button
        v-for="category in categories"
        :key="category.id"
        class="category-btn"
        :class="{ active: selectedCategory === category.id }"
        @click="$emit('update:selectedCategory', category.id)"
      >
        <span class="category-name">{{ category.name }}</span>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const categories = [
  { id: 'all', name: 'All Places' },
  { id: 'cafe', name: 'Cafes' },
  { id: 'study-area', name: 'Study Areas' },
  { id: 'coworking', name: 'Coworking' },
  { id: 'chill', name: 'Chill Spots' }
];

defineProps({
  searchQuery: {
    type: String,
    default: ''
  },
  locationQuery: {
    type: String,
    default: ''
  },
  selectedCategory: {
    type: String,
    default: 'all'
  },
  placeholder: {
    type: String,
    default: 'Search for cafes, study areas, chill spots...'
  }
});

const emit = defineEmits(['update:searchQuery', 'update:locationQuery', 'update:selectedCategory']);

const detecting = ref(false);

const nearMe = () => {
  if (!navigator.geolocation) {
    alert('Geolocation not supported');
    return;
  }
  
  detecting.value = true;
  
  navigator.geolocation.getCurrentPosition(
    (position) => {
      const { latitude, longitude } = position.coords;
      emit('update:locationQuery', `${latitude.toFixed(4)}, ${longitude.toFixed(4)}`);
      detecting.value = false;
    },
    (error) => {
      alert('Could not get location. Please allow location permission.');
      detecting.value = false;
    },
    { enableHighAccuracy: true, timeout: 10000 }
  );
};
</script>

<style scoped>
.search-container {
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
}

.search-row {
  display: flex;
  gap: 12px;
  margin-bottom: 20px;
}

.search-input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
  background: rgba(31, 41, 55, 0.9);
  border: 1px solid rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  border-radius: 50px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
  padding: 8px 24px;
  flex: 1;
}

.search-icon {
  width: 20px;
  height: 20px;
  margin-right: 12px;
  color: var(--text-secondary, #9ca3af);
}

.search-input {
  flex: 1;
  border: none;
  outline: none;
  font-size: 16px;
  color: #f9fafb;
  background: transparent;
  padding: 8px 0;
}

.search-input::placeholder {
  color: var(--text-secondary, #9ca3af);
}

.clear-btn {
  background: var(--tag-bg, #f3f4f6);
  border: none;
  width: 28px;
  height: 28px;
  border-radius: 50%;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  color: var(--text-secondary, #6b7280);
  transition: all 0.2s ease;
}

.clear-btn:hover {
  background: var(--border-color, #e5e7eb);
  color: var(--text-primary, #1a1a2e);
}

.near-me-btn {
  background: #6366f1;
  border: none;
  padding: 6px 12px;
  border-radius: 20px;
  cursor: pointer;
  color: white;
  font-size: 13px;
  font-weight: 600;
  transition: all 0.2s ease;
  white-space: nowrap;
}

.near-me-btn:hover:not(:disabled) {
  background: #818cf8;
}

.near-me-btn:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.category-filters {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  justify-content: center;
}

.category-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 10px 18px;
  border: 2px solid rgba(255, 255, 255, 0.1);
  border-radius: 25px;
  background: rgba(31, 41, 55, 0.8);
  backdrop-filter: blur(10px);
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  color: #9ca3af;
  transition: all 0.2s ease;
}

.category-btn:hover {
  border-color: var(--accent-color, #6366f1);
  color: #a5b4fc;
}

.category-btn.active {
  background: var(--accent-color, #6366f1);
  border-color: var(--accent-color, #6366f1);
  color: white;
}

.category-name {
  white-space: nowrap;
}

@media (max-width: 640px) {
  .search-input-wrapper {
    padding: 8px 16px;
    border-radius: 16px;
  }

  .category-btn {
    padding: 8px 14px;
    font-size: 13px;
  }

  .category-name {
    display: none;
  }
}
</style>