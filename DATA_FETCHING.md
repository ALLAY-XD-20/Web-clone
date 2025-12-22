# How AniRock Fetches Anime Data

## 🎯 Overview

AniRock fetches all anime data from a **custom backend API** hosted on Vercel. The website uses **Axios** (a popular HTTP client library) to make API requests.

---

## 🔗 API Base URL

```javascript
const API_BASE = "https://apii-orcin-theta.vercel.app/api"
```

This is the main API endpoint that serves all anime data to the website.

---

## 📡 How It Works

### 1. **HTTP Client: Axios**

The application uses **Axios** for making HTTP requests. Axios is configured in the JavaScript bundle and handles:
- GET requests to fetch data
- Error handling
- Response parsing
- Request timeouts

### 2. **API Request Pattern**

```javascript
// Example from the code
async function fetchAnimeInfo(id, random = false) {
  const baseURL = "https://apii-orcin-theta.vercel.app/api";
  
  try {
    if (random) {
      // Get random anime
      const randomResponse = await axios.get(`${baseURL}/random/id`);
      const infoResponse = await axios.get(`${baseURL}/info?id=${randomResponse.data.results}`);
      return infoResponse.data.results;
    } else {
      // Get specific anime by ID
      const response = await axios.get(`${baseURL}/info?id=${id}`);
      return response.data.results;
    }
  } catch (error) {
    console.error("Error fetching anime info:", error);
    return error;
  }
}
```

---

## 🗂️ API Endpoints Used

Based on the code analysis, here are the main API endpoints:

### **Home Page Data**
```
GET /api/home
```
Returns:
- Spotlights (featured anime)
- Latest episodes
- Top airing
- Most favorite
- Latest completed
- Trending
- Top 10

### **Anime Information**
```
GET /api/info?id={anime_id}
```
Returns detailed info about a specific anime:
- Title, Japanese title
- Poster image
- Description/overview
- Genres, studios, producers
- Episodes, duration, status
- Characters & voice actors
- Recommended anime

### **Random Anime**
```
GET /api/random/id
```
Returns a random anime ID

### **Search**
```
GET /api/search/suggest?keyword={query}
```
Returns search suggestions as you type

### **Character List**
```
GET /api/character/list/{anime_id}?page={page_number}
```
Returns paginated list of characters and voice actors

### **Quick Tooltip Info**
```
GET /api/qtip/{anime_id}
```
Returns brief anime info for hover tooltips:
- Title, rating, quality
- Sub/dub count
- Episode count
- Description

---

## 🔄 Data Flow Diagram

```
┌─────────────────────────────────────────────────────┐
│                    User Action                       │
│  (Opens site, searches, clicks anime, etc.)         │
└───────────────────┬─────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────┐
│              React Component                         │
│  (Home, AnimeDetails, Search, etc.)                 │
└───────────────────┬─────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────┐
│           Axios HTTP Request                         │
│  axios.get("https://apii-orcin-theta.vercel.app/api/...")│
└───────────────────┬─────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────┐
│          Vercel API Server                           │
│  (Backend hosted on Vercel)                         │
└───────────────────┬─────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────┐
│      External Anime Data Source                      │
│  (Likely Consumet API or similar)                   │
└───────────────────┬─────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────┐
│         Response (JSON Data)                         │
│  Returns to React component                         │
└───────────────────┬─────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────┐
│          React State Update                          │
│  Component re-renders with new data                 │
└───────────────────┬─────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────┐
│            UI Updates                                │
│  User sees anime cards, details, etc.               │
└─────────────────────────────────────────────────────┘
```

---

## 💡 Example: Loading Home Page

When you visit the home page, here's what happens:

1. **Component Mounts**
   ```javascript
   useEffect(() => {
     fetchHomeData();
   }, []);
   ```

2. **API Request**
   ```javascript
   const response = await axios.get('https://apii-orcin-theta.vercel.app/api/home');
   ```

3. **Response Structure**
   ```json
   {
     "spotlights": [...],
     "latest_episode": [...],
     "top_airing": [...],
     "most_favorite": [...],
     "trending": [...],
     "topten": [...]
   }
   ```

4. **State Update**
   ```javascript
   setHomeInfo(response.data);
   ```

5. **UI Renders**
   - Spotlight carousel shows featured anime
   - Latest episodes section displays recent releases
   - Trending sidebar shows popular anime

---

## 🛡️ Error Handling

The application includes error handling for API failures:

```javascript
try {
  const data = await fetchAnimeData();
  return data;
} catch (error) {
  console.error("Error fetching anime info:", error);
  // Show error page or fallback UI
  return null;
}
```

---

## ⚡ Performance Optimizations

### 1. **Debounced Search**
Search suggestions are debounced (500ms delay) to avoid excessive API calls:
```javascript
useEffect(() => {
  const timeout = setTimeout(() => {
    setDebouncedValue(searchValue);
  }, 500);
  return () => clearTimeout(timeout);
}, [searchValue]);
```

### 2. **Lazy Loading**
Components use React's lazy loading to reduce initial bundle size

### 3. **Caching**
Browser caches API responses automatically via HTTP headers

---

## 🔍 Backend Architecture (Inferred)

The Vercel API (`apii-orcin-theta.vercel.app`) likely:

1. **Acts as a proxy/aggregator** for anime data sources
2. **Fetches from Consumet API** or similar anime databases
3. **Formats and normalizes** data for the frontend
4. **Handles rate limiting** and caching
5. **Provides consistent API** regardless of upstream changes

---

## 📝 Key Takeaways

✅ **All data comes from**: `https://apii-orcin-theta.vercel.app/api`  
✅ **HTTP client used**: Axios  
✅ **Request method**: GET requests with query parameters  
✅ **Data format**: JSON  
✅ **Error handling**: Try-catch blocks with fallback UI  
✅ **Performance**: Debouncing, lazy loading, browser caching  

---

## 🚀 Want to Build Your Own?

To create a similar setup:

1. **Backend**: Deploy a Node.js API on Vercel
2. **Data Source**: Use Consumet API or AniList GraphQL API
3. **Frontend**: React with Axios for HTTP requests
4. **Caching**: Implement Redis or in-memory caching
5. **CDN**: Use Cloudflare for static assets

---

## 📚 Related Documentation

- [Axios Documentation](https://axios-http.com/)
- [Consumet API](https://github.com/consumet/api.consumet.org)
- [Vercel Serverless Functions](https://vercel.com/docs/functions)
- [React Data Fetching](https://react.dev/learn/synchronizing-with-effects)
