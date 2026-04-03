<template>
  <div class="place-card">
    <div class="card-image">
      <img :src="place.image" :alt="place.name" />
      <span class="category-badge">
          <svg class="badge-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path :d="categoryIcon"/>
          </svg>
          {{ categoryName }}
        </span>
    </div>
    <div class="card-content">
      <div class="card-header">
        <h3 class="place-name">{{ place.name }}</h3>
        <div class="rating">
          <svg class="star-icon" viewBox="0 0 24 24" fill="currentColor">
            <path d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"/>
          </svg>
          <span class="rating-value">{{ place.rating.toFixed(1) }}</span>
          <span class="reviews">({{ place.reviews }} reviews)</span>
        </div>
      </div>
      <p class="address">{{ place.address }}</p>
      <p class="description">{{ place.description }}</p>
      <div class="amenities">
        <span v-for="amenity in displayedAmenities" :key="amenity" class="amenity-tag">
          {{ amenity }}
        </span>
        <span v-if="place.amenities.length > 3" class="amenity-more">
          +{{ place.amenities.length - 3 }}
        </span>
      </div>
      <div class="card-footer">
        <span class="hours">{{ place.hours }}</span>
        <span class="price">{{ '$'.repeat(place.priceLevel) }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue';

const props = defineProps({
  place: {
    type: Object,
    required: true
  }
});

const categoryIcons = {
  'cafe': 'M2 21h14a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2H4a2 2 0 0 0-2 2v10a4 4 0 0 0 2 2zm4-9h4v4h-4v-4zm-2 6h2v2H4v-2zm3-4h2v2H7V8zm3 0h2v2h-2V8zm3 0h2v2h-2V8zM7 14h2v2H7v-2zm3 0h2v2h-2v-2zm3 0h2v2h-2v-2z',
  'study-area': 'M4 19.5A2.5 2.5 0 0 1 6.5 17H20M4 4.5v15A2.5 2.5 0 0 1 2.5 17H2v-3.5a2.5 2.5 0 0 1 2.5-2.5h1M2 7h4M2 10h4M2 13h4M14 7h4M14 10h4M14 13h4',
  'coworking': 'M20 6h-4V4c0-1.11-.89-2-2-2h-4c-1.11 0-2 .89-2 2v2H4c-1.11 0-1.99.89-1.99 2L2 19c0 1.11.89 2 2 2h16c1.11 0 2-.89 2-2V8c0-1.11-.89-2-2-2zm-6 0h-4V4h4v2z',
  'chill': 'M10 20v-6h4v6h5v-8h3L12 3 2 12h3v8z'
};

const categoryNames = {
  'cafe': 'Cafe',
  'study-area': 'Study Area',
  'coworking': 'Coworking',
  'chill': 'Chill Spot'
};

const categoryName = computed(() => {
  return categoryNames[props.place.category] || props.place.category;
});

const categoryIcon = computed(() => {
  return categoryIcons[props.place.category] || categoryIcons['cafe'];
});

const displayedAmenities = computed(() => {
  return props.place.amenities.slice(0, 3);
});
</script>

<style scoped>
.place-card {
  background: rgba(31, 41, 55, 0.9);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 16px;
  overflow: hidden;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  cursor: pointer;
}

@keyframes cardFadeIn {
  from {
    opacity: 0;
    transform: translateY(20px) scale(0.98);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.place-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 40px rgba(99, 102, 241, 0.2);
}

.card-image {
  position: relative;
  height: 180px;
  overflow: hidden;
}

.card-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.place-card:hover .card-image img {
  transform: scale(1.05);
}

.category-badge {
  position: absolute;
  top: 12px;
  left: 12px;
  background: #ffffff;
  padding: 6px 12px;
  border-radius: 20px;
  font-size: 14px;
  font-weight: 600;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
  display: flex;
  align-items: center;
  gap: 6px;
  color: #1a1a2e;
}

.badge-icon {
  width: 16px;
  height: 16px;
  color: #6b7280;
}

.card-content {
  padding: 16px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 8px;
}

.place-name {
  font-size: 18px;
  font-weight: 700;
  margin: 0;
  color: #f9fafb;
  flex: 1;
  margin-right: 8px;
}

.rating {
  display: flex;
  align-items: center;
  gap: 4px;
  flex-shrink: 0;
}

.star-icon {
  width: 16px;
  height: 16px;
  color: #f59e0b;
}

.rating-value {
  font-weight: 700;
  color: #f9fafb;
}

.reviews {
  color: #9ca3af;
  font-size: 12px;
}

.address {
  color: #9ca3af;
  font-size: 13px;
  margin: 0 0 8px 0;
}

.description {
  color: #d1d5db;
  font-size: 14px;
  margin: 0 0 12px 0;
  line-height: 1.5;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.amenities {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-bottom: 12px;
}

.amenity-tag {
  background: rgba(99, 102, 241, 0.2);
  color: #a5b4fc;
  padding: 4px 10px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: 500;
}

.amenity-more {
  background: var(--accent-color, #6366f1);
  color: white;
  padding: 4px 10px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: 500;
}

.card-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 12px;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
}

.hours {
  font-size: 12px;
  color: #9ca3af;
}

.price {
  color: var(--accent-color, #6366f1);
  font-weight: 700;
  letter-spacing: 2px;
}
</style>