# Tambay Finder

A Vue 3 web application that helps users discover the best cafés, study areas, and chill spots across the Philippines. Built with Vite, Three.js for 3D visualization, and integrates multiple location APIs.

## Features

- **3D Globe Background** - Interactive Three.js globe visualization
- **Location Search** - Search places in major Philippine cities (Manila, Quezon City, Makati, Cebu, Davao, etc.)
- **Category Filtering** - Filter by cafe, study area, or coworking spaces
- **AI Recommendations** - Get intelligent place suggestions
- **Place Cards** - Detailed view with ratings, amenities, and hours

## Tech Stack

- Vue 3 (Composition API)
- Vite
- Three.js
- CSS Animations

## Getting Started

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build
```

## Project Structure

```
src/
├── App.vue              # Main application component
├── main.js              # App entry point
├── style.css            # Global styles
├── components/
│   ├── SearchBar.vue    # Search and filter component
│   ├── PlaceCard.vue    # Place display card
│   └── ThreeGlobe.vue   # 3D globe component
└── assets/
    ├── hero.png
    ├── vue.svg
    └── vite.svg
```

## Environment Variables (Optional)

Create `.env` file for API integrations:

```
VITE_GOOGLE_API_KEY=your_google_places_api_key
VITE_GEOAPIFY_KEY=your_geoapify_api_key
VITE_PLACES_JSON_URL=your_custom_places_json_url
```

## License

MIT