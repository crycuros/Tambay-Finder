<script setup>
import { ref, computed, onMounted, watch } from 'vue';
import SearchBar from './components/SearchBar.vue';
import PlaceCard from './components/PlaceCard.vue';
import ThreeGlobe from './components/ThreeGlobe.vue';

const allPlaces = ref([]);
const placesLoading = ref(true);

const placesCache = {};

const fetchPlaces = async () => {
  placesLoading.value = true;
  allPlaces.value = [];
  
  const loc = locationQuery.value.trim();
  
  if (!loc) {
    placesLoading.value = false;
    return;
  }
  
  const coords = getCoordinates(loc);
  if (!coords) {
    placesLoading.value = false;
    return;
  }
  
  const cacheKey = loc.toLowerCase();
  if (placesCache[cacheKey]) {
    allPlaces.value = placesCache[cacheKey];
    placesLoading.value = false;
    return;
  }
  
  try {
    const osmPlaces = await Promise.race([
      fetchPlacesData(loc),
      new Promise((_, reject) => setTimeout(() => reject(new Error('timeout')), 8000))
    ]);
    if (osmPlaces?.length) {
      placesCache[cacheKey] = osmPlaces;
      allPlaces.value = osmPlaces;
    } else {
      const areaData = getCoordinates(loc);
      const sample = generateNearbyPlaces(areaData?.lat, areaData?.lon, areaData?.name || loc);
      placesCache[cacheKey] = sample;
      allPlaces.value = sample;
    }
  } catch (e) {
    console.error('Fetch error, using sample data:', e);
    const areaData = getCoordinates(loc);
    const sample = generateNearbyPlaces(areaData?.lat, areaData?.lon, areaData?.name || loc);
    placesCache[cacheKey] = sample;
    allPlaces.value = sample;
  }
  
  placesLoading.value = false;
};

const majorAreas = {
  'manila': { lat: 14.5995, lon: 120.9842, name: 'Manila' },
  'quezon': { lat: 14.6760, lon: 121.0437, name: 'Quezon City' },
  'makati': { lat: 14.5547, lon: 121.0244, name: 'Makati' },
  'bgc': { lat: 14.5538, lon: 121.0479, name: 'Bonifacio Global City' },
  'cebu': { lat: 10.3157, lon: 123.9494, name: 'Cebu City' },
  'davao': { lat: 7.0731, lon: 125.6128, name: 'Davao City' },
  'taguig': { lat: 14.5176, lon: 121.0509, name: 'Taguig' },
  'pasay': { lat: 14.5082, lon: 121.0603, name: 'Pasay' },
  'caloocan': { lat: 14.6612, lon: 120.9832, name: 'Caloocan' },
  'las pinas': { lat: 14.4444, lon: 120.9997, name: 'Las Pinas' },
  'paranaque': { lat: 14.4793, lon: 121.0194, name: 'Paranaque' },
  'alabang': { lat: 14.4177, lon: 121.0725, name: 'Alabang' },
  'ortigas': { lat: 14.5867, lon: 121.0635, name: 'Ortigas' },
  'san juan': { lat: 14.6055, lon: 121.0363, name: 'San Juan' },
  'sta mesa': { lat: 14.6308, lon: 121.0721, name: 'Santa Mesa' },
  'mandaluyong': { lat: 14.5794, lon: 121.0359, name: 'Mandaluyong' },
  'pureza': { lat: 14.5765, lon: 121.0589, name: 'Pureza' },
  'romblooks': { lat: 14.5612, lon: 121.0525, name: 'Romblon' },
  'san antonio': { lat: 14.5689, lon: 121.0342, name: 'San Antonio' },
  'port area': { lat: 14.5833, lon: 120.9500, name: 'Port Area' }
};

const getCoordsFromGps = (loc) => {
  const parts = loc.split(',').map(s => parseFloat(s.trim()));
  if (parts.length === 2 && !isNaN(parts[0]) && !isNaN(parts[1])) {
    return { lat: parts[0], lon: parts[1], name: 'Your Location' };
  }
  return null;
};

const getCoordinates = (loc) => {
  const gps = getCoordsFromGps(loc);
  if (gps) return gps;
  
  const key = loc.toLowerCase().trim();
  if (majorAreas[key]) return majorAreas[key];
  
  const found = Object.keys(majorAreas).find(k => key.includes(k) || k.includes(key));
  if (found) return majorAreas[found];
  
  return null;
};

const generateNearbyPlaces = (lat, lon, areaName) => {
  const sampleNames = [
    'Coffee Hub', 'Study Den', 'Work Lounge', 'Chill Cafe', 'Bean House',
    'The Study Spot', 'Brew & Book', 'Urban Grind', 'Quiet Corner', 'Hub Cafe',
    'Digital Nomad', 'Creative Space', 'The Hangout', 'Roast & Toast', 'Bean Therapy'
  ];
  
  const amenitiesList = [
    ['Wi-Fi', 'Power Outlets'],
    ['Wi-Fi', 'Air Conditioning', 'Power Outlets'],
    ['Wi-Fi', 'Food Available'],
    ['Power Outlets', 'Quiet Zone'],
    ['Wi-Fi', 'Meeting Rooms']
  ];
  
  return sampleNames.slice(0, 12).map((name, i) => {
    const latOffset = (Math.random() - 0.5) * 0.008;
    const lonOffset = (Math.random() - 0.5) * 0.008;
    const amenities = amenitiesList[Math.floor(Math.random() * amenitiesList.length)];
    
    return {
      id: i + 1,
      name: `${name} ${areaName}`,
      category: i % 3 === 0 ? 'cafe' : i % 3 === 1 ? 'study-area' : 'coworking',
      rating: +(3.8 + Math.random() * 1).toFixed(1),
      reviews: Math.floor(Math.random() * 150) + 10,
      priceLevel: 1,
      address: `Street ${Math.floor(Math.random() * 100) + 1}, ${areaName}`,
      description: `A nice spot in ${areaName} with ${amenities.join(', ')}`,
      amenities: amenities,
      hours: '8:00 AM - 10:00 PM',
      image: 'https://images.unsplash.com/photo-1501339847302-ac426a4a7cbb?w=400'
    };
  });
};

const fetchPlacesData = async (loc, areaName) => {
  // Check for demo mode or explicit JSON URL
  if (loc === 'demo' || loc?.startsWith('http')) {
    try {
      const url = loc.startsWith('http') ? loc : '/places.json';
      const resp = await fetch(url);
      const data = await resp.json();
      const places = Array.isArray(data) ? data : data.places || [];
      return places.map((p, i) => ({
        id: p.id || i,
        name: p.name || `Place ${i + 1}`,
        category: p.category || 'cafe',
        rating: p.rating || 4.0,
        reviews: p.reviews || 50,
        priceLevel: p.priceLevel || 1,
        address: p.address || areaName || loc,
        description: p.description || `A place in ${areaName}`,
        amenities: p.amenities || ['Wi-Fi', 'Power Outlets'],
        hours: p.hours || '8:00 AM - 10:00 PM',
        image: p.image || 'https://images.unsplash.com/photo-1501339847302-ac426a4a7cbb?w=400'
      }));
    } catch (e) {
      console.error('JSON fetch error:', e);
      return [];
    }
  }
  
  const areaData = getCoordinates(loc);
  if (!areaData) return [];
  
  const coords = { lat: areaData.lat, lon: areaData.lon };
  const foundArea = areaData.name;
  
  const jsonUrl = import.meta.env.VITE_PLACES_JSON_URL;
  
  if (jsonUrl) {
    try {
      const resp = await fetch(jsonUrl);
      const data = await resp.json();
      const places = Array.isArray(data) ? data : data.places || [];
      return places.map((p, i) => ({
        id: p.id || i,
        name: p.name || `Place ${i + 1}`,
        category: p.category || 'cafe',
        rating: p.rating || 4.0,
        reviews: p.reviews || 50,
        priceLevel: p.priceLevel || 1,
        address: p.address || foundArea,
        description: p.description || `A place in ${foundArea}`,
        amenities: p.amenities || ['Wi-Fi'],
        hours: p.hours || '8:00 AM - 10:00 PM',
        image: p.image || 'https://images.unsplash.com/photo-1501339847302-ac426a4a7cbb?w=400'
      }));
    } catch {}
  }
  
  // Try free Geoapify (no credit card, free tier)
  try {
    const geoapifyKey = import.meta.env.VITE_GEOAPIFY_KEY;
    if (geoapifyKey) {
      const url = `https://api.geoapify.com/v1/geocode/search?text=cafe%20${encodeURIComponent(foundArea)}&apiKey=${geoapifyKey}&limit=15`;
      const resp = await fetch(url);
      const data = await resp.json();
      if (data.results) {
        return data.results.map((place, i) => ({
          id: place.place_id,
          name: place.name || `Cafe ${i + 1}`,
          category: 'cafe',
          rating: 4.0,
          reviews: 50,
          priceLevel: 1,
          address: `${place.city || foundArea}, ${place.country || 'PH'}`,
          description: `A cafe in ${foundArea}`,
          amenities: ['Wi-Fi', 'Power Outlets'],
          hours: '8:00 AM - 10:00 PM',
          image: 'https://images.unsplash.com/photo-1501339847302-ac426a4a7cbb?w=400'
        }));
      }
    }
  } catch {}
  
  // Try Places API (free $200/mo credit - no card on signup sometimes)
  const googleKey = import.meta.env.VITE_GOOGLE_API_KEY;
  if (googleKey) {
    try {
      const url1 = `https://maps.googleapis.com/maps/api/place/nearbysearch/json?location=${coords.lat},${coords.lon}&radius=4000&type=cafe&key=${googleKey}`;
      const resp1 = await fetch(url1);
      const data1 = await resp1.json();
      if (data1.results?.length) {
        return data1.results.slice(0, 15).map((place) => ({
          id: place.place_id,
          name: place.name,
          category: 'cafe',
          rating: place.rating || 4.0,
          reviews: place.user_ratings_total || 50,
          priceLevel: place.price_level || 1,
          address: place.vicinity || foundArea,
          description: `A nice cafe in ${foundArea}`,
          amenities: ['Wi-Fi', 'Power Outlets'],
          hours: place.opening_hours?.open_now !== undefined ? (place.opening_hours.open_now ? 'Open Now' : 'Closed') : '8:00 AM - 10:00 PM',
          image: 'https://images.unsplash.com/photo-1501339847302-ac426a4a7cbb?w=400'
        }));
      }
    } catch {}
  }
  
  // Final fallback - Nominatim search
  try {
    const searchResp = await fetch(
      `https://nominatim.openstreetmap.org/search?format=json&q=cafe+${encodeURIComponent(foundArea)}&limit=15`,
      { headers: { 'User-Agent': 'TambayFinder/1.0' } }
    );
    const searchData = await searchResp.json();
    if (searchData?.length) {
      return searchData.slice(0, 15).map((place, i) => ({
        id: place.place_id || i,
        name: place.display_name?.split(',')[0] || `Cafe ${i + 1}`,
        category: 'cafe',
        rating: +(3.8 + Math.random() * 1).toFixed(1),
        reviews: Math.floor(Math.random() * 100) + 20,
        priceLevel: 1,
        address: place.display_name?.split(',').slice(1, 3).join(',').trim() || foundArea,
        description: `A cafe in ${foundArea}`,
        amenities: ['Wi-Fi', 'Power Outlets'],
        hours: '8:00 AM - 10:00 PM',
        image: 'https://images.unsplash.com/photo-1501339847302-ac426a4a7cbb?w=400'
      }));
    }
  } catch {}
  
  return [];
};

const formatPlace = (place) => ({
  id: place.fsq_id || Math.random(),
  name: place.name,
  category: place.categories?.[0]?.short_name?.toLowerCase() || 'place',
  rating: place.rating || 4.0,
  reviews: place.reviews || Math.floor(Math.random() * 200) + 50,
  priceLevel: place.price_level || 1,
  address: place.location?.address || place.location?.locality || 'Location TBD',
  description: place.categories?.[0]?.name || 'A great place to hang out',
  amenities: ['Wi-Fi', 'Power Outlets'].slice(0, Math.floor(Math.random() * 3) + 1),
  hours: place.hours?.regular?.[0]?.open || '9:00 AM - 10:00 PM',
  image: place.photos?.[0]?.prefix + '300x200' + place.photos?.[0]?.suffix || 'https://images.unsplash.com/photo-1559925393-8be0ec476789?w=400'
});

onMounted(() => {
  locationQuery.value = 'demo';
  fetchPlaces();
});

const aiLoading = ref(false);
const aiRecommendation = ref([]);
const showAi = ref(false);

const searchQuery = ref('');
const locationQuery = ref('');
const selectedCategory = ref('all');

let debounceTimer;
watch(locationQuery, (newVal) => {
  clearTimeout(debounceTimer);
  if (newVal && newVal.trim().length >= 2) {
    const coords = getCoordinates(newVal);
    if (coords) {
      debounceTimer = setTimeout(fetchPlaces, 800);
    }
  }
});

const manualSearch = () => {
  clearTimeout(debounceTimer);
  delete placesCache[locationQuery.value.trim()];
  fetchPlaces();
};

const filteredPlaces = computed(() => {
  let result = allPlaces.value;

  if (selectedCategory.value !== 'all') {
    result = result.filter(place => place.category === selectedCategory.value);
  }

  if (locationQuery.value.trim()) {
    const query = locationQuery.value.toLowerCase().trim();
    result = result.filter(place =>
      place.address.toLowerCase().includes(query)
    );
  } else if (searchQuery.value.trim()) {
    const query = searchQuery.value.toLowerCase().trim();
    result = result.filter(place =>
      place.name.toLowerCase().includes(query) ||
      place.description.toLowerCase().includes(query) ||
      place.address.toLowerCase().includes(query) ||
      place.amenities.some(a => a.toLowerCase().includes(query))
    );
  }

  return result;
});

const resultCount = computed(() => filteredPlaces.value.length);

const getAiRecommendation = async () => {
  aiLoading.value = true;
  showAi.value = true;
  aiRecommendation.value = [];
  
  const loc = locationQuery.value.trim();
  
  try {
    const placesData = await fetchPlacesFromApi(loc);
    aiRecommendation.value = placesData.slice(0, 10);
  } catch (error) {
    console.error('Error fetching places:', error);
    aiRecommendation.value = allPlaces.value.slice(0, 10).map(p => ({
      name: p.name,
      rating: p.rating,
      category: p.category,
      address: p.address
    }));
  }
  
  aiLoading.value = false;
};

const fetchPlacesFromApi = async (location) => {
  return allPlaces.value.slice(0, 10).map(p => ({
    name: p.name,
    rating: p.rating,
    category: p.category,
    address: p.address
  }));
};
</script>

<template>
  <div class="app">
    <ThreeGlobe class="bg-globe" />

    <header class="header">
      <div class="header-content">
        <div class="logo">
          <div class="logo-text">
            <h1>Tambay Finder</h1>
            <p>Find your perfect hangout spot</p>
          </div>
        </div>
      </div>
    </header>

    <main class="main">
      <section class="hero">
        <h2>Discover Great Places</h2>
        <p>Find the best cafés, study areas, and chill spots near you</p>
      </section>

      <SearchBar
        v-model:searchQuery="searchQuery"
        v-model:locationQuery="locationQuery"
        v-model:selectedCategory="selectedCategory"
      />

      <div class="ai-section">
        <button class="ai-btn" @click="getAiRecommendation" :disabled="aiLoading">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M12 2a2 2 0 0 1 2 2c0 .74-.4 1.39-1 1.73V7h1a7 7 0 0 1 7 7h1a1 1 0 0 1 1 1v3a1 1 0 0 1-1 1h-1v1a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-1H2a1 1 0 0 1-1-1v-3a1 1 0 0 1 1-1h1a7 7 0 0 1 7-7h1v-.27A2 2 0 0 1 12 2z"/>
          </svg>
          {{ aiLoading ? 'Thinking...' : 'AI Recommend' }}
        </button>
        
        <div v-if="showAi" class="ai-result" :class="{ loading: aiLoading }">
          <div v-if="aiLoading" class="ai-loading">
            <span class="dot"></span>
            <span class="dot"></span>
            <span class="dot"></span>
          </div>
          <div v-else class="ai-recs">
            <p class="ai-header">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M12 2a2 2 0 0 1 2 2c0 .74-.4 1.39-1 1.73V7h1a7 7 0 0 1 7 7h1a1 1 0 0 1 1 1v3a1 1 0 0 1-1 1h-1v1a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-1H2a1 1 0 0 1-1-1v-3a1 1 0 0 1 1-1h1a7 7 0 0 1 7-7h1v-.27A2 2 0 0 1 12 2z"/>
              </svg>
              Top {{ aiRecommendation.length }} spots for you
            </p>
            <div class="ai-grid">
              <div v-for="(rec, i) in aiRecommendation" :key="i" class="ai-rec-card">
                <span class="rec-num">{{ i + 1 }}</span>
                <div class="rec-info">
                  <p class="rec-name">{{ rec.name }}</p>
                  <p class="rec-meta">{{ rec.rating }} stars - {{ rec.category }}</p>
                  <p class="rec-address">{{ rec.address }}</p>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <section class="results">
        <div class="results-header">
          <span class="results-count">
            <template v-if="placesLoading">Searching in {{ locationQuery }}...</template>
            <template v-else-if="resultCount === 0 && locationQuery">
              No places found in "{{ locationQuery }}"
              <button class="retry-btn" @click="manualSearch">Search</button>
            </template>
            <template v-else>{{ resultCount }} place{{ resultCount !== 1 ? 's' : '' }} found</template>
          </span>
          <button v-if="!placesLoading && resultCount === 0 && locationQuery" class="retry-btn" @click="manualSearch">
            Search
          </button>
          <select class="sort-select">
            <option value="rating">Highest Rated</option>
            <option value="reviews">Most Reviewed</option>
            <option value="name">Name (A-Z)</option>
          </select>
        </div>

        <TransitionGroup name="cards" tag="div" class="places-grid">
          <PlaceCard
            v-for="place in filteredPlaces"
            :key="place.id"
            :place="place"
          />
        </TransitionGroup>

        <div v-if="filteredPlaces.length === 0" class="no-results">
          <h3>No places found</h3>
          <p>Try adjusting your search or filters</p>
        </div>
      </section>
    </main>

    <footer class="footer">
      <p>Made for the community</p>
    </footer>
  </div>
</template>

<style scoped>
.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  position: relative;
}

.bg-globe {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: -1;
}

.bg-globe::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(17, 24, 39, 0.7);
}

.header {
  background: rgba(26, 26, 46, 0.95);
  backdrop-filter: blur(10px);
  padding: 20px 0;
  position: sticky;
  top: 0;
  z-index: 100;
  animation: slideDown 0.5s ease-out;
}

@keyframes slideDown {
  from {
    opacity: 0;
    transform: translateY(-100%);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.header-content {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 24px;
}

.logo {
  display: flex;
  align-items: center;
  gap: 12px;
}

.logo-icon {
  font-size: 36px;
}

.logo-text h1 {
  margin: 0;
  font-size: 24px;
  font-weight: 700;
  color: white;
}

.logo-text p {
  margin: 4px 0 0 0;
  font-size: 14px;
  color: rgba(255, 255, 255, 0.7);
}

.main {
  flex: 1;
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 24px;
  width: 100%;
  box-sizing: border-box;
}

.hero {
  text-align: center;
  padding: 48px 24px 32px;
  animation: fadeInDown 0.6s ease-out;
  position: relative;
  overflow: hidden;
}

@keyframes fadeInDown {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.hero h2 {
  font-size: 32px;
  font-weight: 700;
  color: #f9fafb;
  margin: 0 0 8px 0;
}

.hero p {
  font-size: 18px;
  color: #9ca3af;
  margin: 0;
}

.results {
  padding-bottom: 48px;
}

.results-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
}

.results-count {
  font-size: 14px;
  font-weight: 500;
  color: #9ca3af;
}

.sort-select {
  padding: 8px 16px;
  border: 2px solid rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  font-size: 14px;
  color: #f9fafb;
  background: rgba(31, 41, 55, 0.9);
  backdrop-filter: blur(10px);
  cursor: pointer;
}

.retry-btn {
  padding: 8px 16px;
  background: #6366f1;
  border: none;
  border-radius: 8px;
  font-size: 14px;
  color: white;
  cursor: pointer;
}

.retry-btn:hover {
  background: #818cf8;
}

.places-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 24px;
}

.no-results {
  text-align: center;
  padding: 64px 24px;
}

.no-results h3 {
  font-size: 20px;
  font-weight: 600;
  color: #f9fafb;
  margin: 0 0 8px 0;
}

.no-results p {
  color: #9ca3af;
  margin: 0;
}

.footer {
  background: var(--header-bg, #1a1a2e);
  padding: 24px;
  text-align: center;
}

.footer p {
  margin: 0;
  color: rgba(255, 255, 255, 0.7);
  font-size: 14px;
}

@media (max-width: 768px) {
  .hero h2 {
    font-size: 24px;
  }

  .hero p {
    font-size: 16px;
  }

  .places-grid {
    grid-template-columns: 1fr;
  }
}

.cards-enter-active {
  animation: fadeScale 0.35s ease-out;
}

.cards-leave-active {
  animation: fadeScale 0.25s ease-in reverse;
  position: absolute;
}

.cards-move {
  transition: transform 0.4s ease;
}

@keyframes fadeScale {
  from {
    opacity: 0;
    transform: scale(0.9) translateY(10px);
  }
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: scale(0.95);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

.ai-section {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
  margin-bottom: 32px;
}

.ai-btn {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 12px 24px;
  background: linear-gradient(135deg, #6366f1, #8b5cf6);
  border: none;
  border-radius: 50px;
  color: white;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 20px rgba(99, 102, 241, 0.4);
}

.ai-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 30px rgba(99, 102, 241, 0.5);
}

.ai-btn:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.ai-btn svg {
  width: 20px;
  height: 20px;
}

.ai-result {
  background: rgba(31, 41, 55, 0.9);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(139, 92, 246, 0.3);
  border-radius: 16px;
  padding: 20px 24px;
  max-width: 600px;
  text-align: center;
  animation: fadeIn 0.3s ease-out;
}

.ai-recs {
  text-align: left;
}

.ai-header {
  color: #e5e7eb;
  font-size: 16px;
  font-weight: 600;
  margin: 0 0 20px 0;
  text-align: center;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.ai-header svg {
  width: 20px;
  height: 20px;
  color: #a5b4fc;
}

.ai-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 12px;
}

.ai-rec-card {
  display: flex;
  gap: 12px;
  padding: 14px;
  background: rgba(99, 102, 241, 0.1);
  border: 1px solid rgba(139, 92, 246, 0.2);
  border-radius: 12px;
}

.rec-num {
  width: 32px;
  height: 32px;
  background: linear-gradient(135deg, #6366f1, #8b5cf6);
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-weight: 700;
  font-size: 14px;
  flex-shrink: 0;
}

.rec-info {
  flex: 1;
  min-width: 0;
}

.rec-name {
  color: #f9fafb;
  font-weight: 600;
  font-size: 14px;
  margin: 0 0 4px 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.rec-meta {
  color: #a5b4fc;
  font-size: 12px;
  margin: 0 0 2px 0;
}

.rec-address {
  color: #9ca3af;
  font-size: 12px;
  margin: 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.ai-loading {
  display: flex;
  gap: 8px;
  justify-content: center;
  padding: 8px 0;
}

.ai-loading .dot {
  width: 10px;
  height: 10px;
  background: #8b5cf6;
  border-radius: 50%;
  animation: bounce 1.4s infinite ease-in-out both;
}

.ai-loading .dot:nth-child(1) { animation-delay: -0.32s; }
.ai-loading .dot:nth-child(2) { animation-delay: -0.16s; }

@keyframes bounce {
  0%, 80%, 100% { transform: scale(0); }
  40% { transform: scale(1); }
}
</style>