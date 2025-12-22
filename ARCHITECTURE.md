# AniRock Architecture Documentation

## System Architecture

### High-Level Overview

AniRock is a **Single Page Application (SPA)** built with React (bundled via Vite) that provides anime streaming services. 

> **⚠️ Technical Note:** This codebase is primarily a **production bundle** (`bundle.js`) that has been reverse-engineered and patched. We do not have the original React source files (JSX components). All "modifications" are direct patches to the minified/bundled code.

The architecture follows a client-side rendering pattern with API-driven data fetching.

```
┌─────────────────────────────────────────────────────────────┐
│                         Browser                              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                   React Application                    │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────────────┐    │  │
│  │  │  Router  │  │ Context  │  │    Components    │    │  │
│  │  │ (Routes) │  │  (State) │  │  (UI Elements)   │    │  │
│  │  └──────────┘  └──────────┘  └──────────────────┘    │  │
│  └───────────────────────────────────────────────────────┘  │
│                           │                                  │
│                           ▼                                  │
│                    ┌─────────────┐                          │
│                    │  fetch API  │                          │
│                    └─────────────┘                          │
└─────────────────────────┼───────────────────────────────────┘
                          │
                          ▼
              ┌───────────────────────┐
              │   External Anime API  │
              │  (Consumet/Similar)   │
              └───────────────────────┘
```

---

## Component Structure

### Core Components (Inferred from Bundle)

1. **App Component**
   - Root component
   - Manages global state
   - Renders router

2. **Router Setup**
   - Client-side routing with React Router v6
   - Likely routes:
     - `/` - Home page with featured/trending anime
     - `/anime/:id` - Anime details page
     - `/watch/:id/:episode` - Video player page
     - `/search` - Search results
     - `/genre/:name` - Genre-filtered listings

3. **Layout Components**
   - Navbar (navigation, search, language switcher)
   - Sidebar (mobile menu)
   - Footer

4. **Feature Components**
   - **Spotlight Carousel**: Featured anime slider
   - **Anime Card**: Reusable card component
   - **Video Player**: Streaming player with controls
   - **Search Bar**: Real-time search
   - **Genre Filter**: Category navigation
   - **Episode List**: Episode selection

---

## State Management

### React Context API

```javascript
// Language Context (visible in bundle)
const LanguageContext = createContext();

// Provides:
- language: string (default: "EN")
- toggleLanguage: (lang: string) => void
```

### Component State

- Local state for UI interactions (modals, dropdowns, etc.)
- Async state for API data (loading, error, data)

### Possible State Structure

```javascript
{
  // Global State (Context)
  language: "EN",
  theme: "dark",
  
  // Component State
  animeList: [],
  currentAnime: {},
  episodes: [],
  searchResults: [],
  loading: false,
  error: null
}
```

---

## Data Flow

### 1. Initial Load
```
User visits site
    ↓
index.html loads
    ↓
React bundle (app.bundle.js) loads
    ↓
React app mounts to #root
    ↓
Router determines current route
    ↓
Component fetches data from API
    ↓
UI renders with data
```

### 2. Navigation
```
User clicks anime card
    ↓
React Router updates URL
    ↓
New component mounts
    ↓
Fetch anime details from API
    ↓
Render anime details page
```

### 3. Search Flow
```
User types in search bar
    ↓
Debounced API call
    ↓
Fetch search results
    ↓
Update search results state
    ↓
Render results dropdown/page
```

---

## API Integration

### Endpoints (Inferred)

The application likely uses endpoints similar to:

```
GET /anime/trending
GET /anime/popular
GET /anime/recent
GET /anime/:id
GET /anime/:id/episodes
GET /anime/search?q={query}
GET /anime/genre/:name
GET /episode/:id/sources
```

### API Client Pattern

```javascript
// Likely implementation
const fetchAnime = async (endpoint) => {
  const response = await fetch(`${API_BASE_URL}${endpoint}`);
  const data = await response.json();
  return data;
};
```

### Error Handling

- Try-catch blocks for API calls
- Error state in components
- Fallback UI for errors
- Loading skeletons during fetch

---

## Routing Configuration

### React Router v6 Setup

```javascript
// Inferred structure
<BrowserRouter>
  <Routes>
    <Route path="/" element={<Home />} />
    <Route path="/anime/:id" element={<AnimeDetails />} />
    <Route path="/watch/:id/:episode" element={<WatchPage />} />
    <Route path="/search" element={<SearchResults />} />
    <Route path="/genre/:name" element={<GenrePage />} />
    <Route path="*" element={<NotFound />} />
  </Routes>
</BrowserRouter>
```

### Navigation Features

- Client-side routing (no page reloads)
- Browser history management
- Deep linking support
- URL parameter handling

---

## Performance Optimizations

### Current Optimizations

1. **Code Splitting**
   - Lazy loading with React.lazy()
   - Dynamic imports for routes
   - Reduces initial bundle size

2. **Image Optimization**
   - Lazy loading images
   - Responsive images
   - Modern formats (WebP likely)

3. **Caching**
   - Browser caching for static assets
   - API response caching (possible)

4. **Minification**
   - JavaScript minified (1.27MB)
   - CSS minified (124KB)
   - HTML minified

### Build Optimizations (Vite)

- Tree shaking (removes unused code)
- Module preloading
- CSS code splitting
- Asset optimization

---

## Styling Architecture

### Tailwind CSS Utility Classes

The application uses Tailwind CSS extensively:

```css
/* Utility-first approach */
.flex .items-center .justify-between
.bg-black .text-white
.rounded-lg .shadow-xl
```

### Custom CSS

- Custom animations (fade, slide, glow)
- Component-specific styles
- Theme variables
- Responsive breakpoints

### Design System

```css
/* CSS Variables */
--white-primary: #fff
--black-primary: #000
--radius-sm: 4px to --radius-2xl: 24px
--space-xs: 0.25rem to --space-2xl: 3rem
```

---

## Security Considerations

### Current Implementation

- Client-side only (no sensitive data)
- External API calls (CORS handled)
- No authentication system
- No user data storage

### Recommendations

1. Implement Content Security Policy (CSP)
2. Add rate limiting for API calls
3. Sanitize user inputs (search queries)
4. Use HTTPS for all API calls
5. Implement error boundaries

---

## Deployment Architecture

### Static Hosting

The application can be deployed to:
- Netlify
- Vercel
- GitHub Pages
- AWS S3 + CloudFront
- Cloudflare Pages

### Deployment Process

```bash
# Build (if source available)
npm run build

# Deploy static files
- index.html
- css/
- js/
- fonts/
- images/
```

### CDN Configuration

- Cache static assets (CSS, JS, fonts)
- Set appropriate cache headers
- Enable compression (Gzip/Brotli)

---

## Monitoring & Analytics

### Current Analytics

- Google Analytics (G-QNPNFNWHYT)
- Tracks page views, user interactions

### Recommended Additions

1. Error tracking (Sentry)
2. Performance monitoring (Web Vitals)
3. User behavior analytics
4. API response time tracking

---

## Future Enhancements

### Recommended Improvements

1. **User Authentication**
   - User accounts
   - Watch history
   - Favorites/bookmarks

2. **Progressive Web App (PWA)**
   - Service worker
   - Offline support
   - Install prompt

3. **Advanced Features**
   - Continue watching
   - Recommendations
   - Comments/ratings
   - Download episodes

4. **Performance**
   - Further code splitting
   - Image CDN
   - API response caching
   - Service worker caching

---

## Development Workflow

### With Source Files

```bash
# Install dependencies
npm install

# Development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview
```

### Without Source Files (Current State)

- Direct HTML/CSS/JS editing
- Manual file management
- Browser-based testing
- No build process

---

## Technical Debt

### Current Limitations

1. Minified code (hard to debug)
2. Large bundle size (1.27MB)
3. No source maps
4. No automated testing
5. No CI/CD pipeline

### Recommendations

1. Obtain source files
2. Set up proper build pipeline
3. Implement testing (Jest, React Testing Library)
4. Add TypeScript for type safety
5. Set up ESLint + Prettier
6. Implement CI/CD (GitHub Actions)

---

## Conclusion

AniRock is a well-architected React SPA with modern best practices. The current production build is optimized for performance but lacks source files for deeper development. For continued development, obtaining the original React source code is recommended.
