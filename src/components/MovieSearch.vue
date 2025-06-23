<script setup>
import { ref } from 'vue'
import { OMDB_API_KEY } from '../../api.config.js'

const apiKey = OMDB_API_KEY
const query = ref('')
const movies = ref([])
const loading = ref(false)
const error = ref('')

const selectedMovie = ref(null)
const showModal = ref(false)

const searchMovies = async () => {
  if (!query.value) return
  loading.value = true
  error.value = ''
  movies.value = []
  try {
    const res = await fetch(`https://www.omdbapi.com/?apikey=${apiKey}&s=${encodeURIComponent(query.value)}`)
    const data = await res.json()
    if (data.Response === 'True') {
      movies.value = data.Search
    } else {
      error.value = data.Error || 'No results found.'
    }
  } catch (e) {
    error.value = 'Network error.'
  } finally {
    loading.value = false
  }
}

const fetchMovieDetails = async (imdbID) => {
  try {
    const res = await fetch(`https://www.omdbapi.com/?apikey=${apiKey}&i=${imdbID}`)
    const data = await res.json()
    selectedMovie.value = data
    showModal.value = true
  } catch (e) {
    selectedMovie.value = null
    showModal.value = false
  }
}

const closeModal = () => {
  showModal.value = false
  selectedMovie.value = null
}
</script>

<template>
  <div class="movie-search">
    <div class="search-bar">
      <h2>Movie Search</h2>
      <form @submit.prevent="searchMovies">
        <input v-model="query" type="text" placeholder="Search for a movie..." />
        <button type="submit" :disabled="loading">Search</button>
      </form>
      <div v-if="loading" class="loading">Loading...</div>
      <div v-if="error" class="error">{{ error }}</div>
    </div>
    <div class="results-container no-scroll">
      <div v-if="movies.length" class="movies-grid">
        <div v-for="movie in movies" :key="movie.imdbID" class="movie-card" @click="fetchMovieDetails(movie.imdbID)">
          <template v-if="movie.Poster !== 'N/A'">
            <img :src="movie.Poster" :alt="movie.Title" width="100" height="140" />
          </template>
          <template v-else>
            <div class="no-poster">
              <svg width="60" height="60" viewBox="0 0 48 48" fill="none" xmlns="http://www.w3.org/2000/svg">
                <rect width="48" height="48" fill="white" fill-opacity="0.01"/>
                <path d="M8 40V16H14L18 8H30L34 16H40V40H8Z" stroke="#8f5cff" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"/>
                <circle cx="24" cy="28" r="8" stroke="#8f5cff" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"/>
                <path d="M10 10L38 38" stroke="#8f5cff" stroke-width="3" stroke-linecap="round"/>
              </svg>
            </div>
          </template>
          <div class="movie-title">{{ movie.Title }}</div>
          <div class="movie-year">{{ movie.Year }}</div>
        </div>
      </div>
    </div>
    <div v-if="showModal && selectedMovie" class="modal-overlay" @click.self="closeModal">
      <div class="modal">
        <button class="close-btn" @click="closeModal" aria-label="Close">&times;</button>
        <template v-if="selectedMovie.Poster !== 'N/A'">
          <img :src="selectedMovie.Poster" :alt="selectedMovie.Title" class="modal-poster" />
        </template>
        <template v-else>
          <div class="modal-no-poster">
            <svg width="90" height="90" viewBox="0 0 48 48" fill="none" xmlns="http://www.w3.org/2000/svg">
              <rect width="48" height="48" fill="white" fill-opacity="0.01"/>
              <path d="M8 40V16H14L18 8H30L34 16H40V40H8Z" stroke="#8f5cff" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"/>
              <circle cx="24" cy="28" r="8" stroke="#8f5cff" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"/>
              <path d="M10 10L38 38" stroke="#8f5cff" stroke-width="3" stroke-linecap="round"/>
            </svg>
          </div>
        </template>
        <h3>{{ selectedMovie.Title }} ({{ selectedMovie.Year }})</h3>
        <p><strong>Genre:</strong> {{ selectedMovie.Genre }}</p>
        <p><strong>Director:</strong> {{ selectedMovie.Director }}</p>
        <p><strong>Actors:</strong> {{ selectedMovie.Actors }}</p>
        <p><strong>Plot:</strong> {{ selectedMovie.Plot }}</p>
        <p><strong>IMDB Rating:</strong> {{ selectedMovie.imdbRating }}</p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.movie-search {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  background: #f5f5f5;
  box-sizing: border-box;
  width: 100vw;
  height: 100vh;
  padding: 0;
  font-family: 'Inter', 'Segoe UI', Arial, sans-serif;
}
.search-bar {
  width: 100%;
  max-width: 600px;
  margin: 0 auto;
  background: #f5f5f5;
  z-index: 2;
  padding: 2.5rem 2rem 1.2rem 2rem;
  box-sizing: border-box;
  border-radius: 0 0 18px 18px;
  box-shadow: 0 2px 24px rgba(0,0,0,0.06);
  position: sticky;
  top: 0;
  transition: box-shadow 0.2s;
}
h2 {
  margin-bottom: 2rem;
  font-size: 2.3rem;
  color: #1a1a1a;
  letter-spacing: 1px;
  font-weight: 700;
  text-align: center;
}
form {
  display: flex;
  gap: 0.7rem;
  margin-bottom: 1.2rem;
  width: 100%;
}
input[type="text"] {
  flex: 1;
  padding: 0.85rem 1.1rem;
  border-radius: 7px;
  border: 1.5px solid #d1d5db;
  font-size: 1.13rem;
  background: #fff;
  transition: border 0.2s, box-shadow 0.2s;
  box-shadow: 0 1px 2px rgba(0,0,0,0.03);
}
input[type="text"]:focus {
  border: 1.5px solid #8f5cff;
  outline: none;
  box-shadow: 0 2px 8px rgba(143,92,255,0.13);
}
button {
  padding: 0.85rem 1.7rem;
  border-radius: 7px;
  border: none;
  background: linear-gradient(90deg, #8f5cff 60%, #a084e8 100%);
  color: white;
  font-size: 1.13rem;
  font-weight: 700;
  cursor: pointer;
  transition: background 0.2s, box-shadow 0.2s, transform 0.1s;
  box-shadow: 0 1px 4px rgba(143,92,255,0.08);
  letter-spacing: 0.5px;
}
button:disabled {
  background: #d1c4f6;
  cursor: not-allowed;
  color: #fff;
}
button:not(:disabled):hover, button:not(:disabled):focus {
  background: linear-gradient(90deg, #a084e8 60%, #8f5cff 100%);
  box-shadow: 0 2px 12px rgba(143,92,255,0.13);
  transform: translateY(-2px) scale(1.03);
}
.loading {
  color: #8f5cff;
  font-weight: 500;
  margin-bottom: 1rem;
  text-align: center;
}
.error {
  color: #d33;
  margin-bottom: 1rem;
  font-weight: 500;
  text-align: center;
}
.results-container {
  flex: 1 1 auto;
  width: 100vw;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 0;
  margin-top: 2rem;
}
.results-container.no-scroll {
  overflow-y: visible;
}
.movies-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 2.2rem;
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
}
.movie-card {
  background: #fff;
  border-radius: 14px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.07);
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 1.2rem 0.7rem 1.5rem 0.7rem;
  transition: box-shadow 0.2s, transform 0.18s;
  min-height: 260px;
  cursor: pointer;
  border: 1.5px solid #f0f0f0;
  position: relative;
}
.movie-card:hover {
  border: 1.5px solid #8f5cff;
  box-shadow: 0 6px 24px rgba(143,92,255,0.16);
}
.movie-card img {
  border-radius: 10px;
  box-shadow: 0 1px 6px rgba(0,0,0,0.10);
  background: #f3f3f3;
  margin-bottom: 1.1rem;
  width: 110px;
  height: 155px;
  object-fit: cover;
  transition: box-shadow 0.2s;
}
.movie-title {
  font-size: 1.18rem;
  font-weight: 700;
  color: #1a1a1a;
  text-align: center;
  margin-bottom: 0.35rem;
  letter-spacing: 0.2px;
}
.movie-year {
  color: #888;
  font-size: 1.04rem;
  font-weight: 500;
  text-align: center;
}
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0,0,0,0.38);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  animation: fadeIn 0.2s;
}
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}
.modal {
  background: #fff;
  border-radius: 16px;
  padding: 2.2rem 2.7rem 1.7rem 2.7rem;
  max-width: 420px;
  width: 92vw;
  box-shadow: 0 10px 40px rgba(0,0,0,0.20);
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  animation: popIn 0.22s;
}
@keyframes popIn {
  from { transform: scale(0.95); opacity: 0.7; }
  to { transform: scale(1); opacity: 1; }
}
.close-btn {
  position: absolute;
  top: 0.7rem;
  right: 1.2rem;
  width: 2.3rem;
  height: 2.3rem;
  background: transparent !important;
  border: none;
  border-radius: 50%;
  font-size: 1.6rem;
  color: #222;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: none !important;
  transition: color 0.2s, transform 0.1s, background 0.2s;
  z-index: 10;
}
.close-btn:hover, .close-btn:focus {
  background: transparent !important;
  color: #ff4d4f;
  transform: scale(1.12);
  box-shadow: none !important;
  outline: none;
}
.modal-poster {
  width: 130px;
  height: 185px;
  object-fit: cover;
  border-radius: 10px;
  margin-bottom: 1.3rem;
  box-shadow: 0 2px 12px rgba(0,0,0,0.13);
}
.modal h3 {
  color: #222;
  font-size: 1.35rem;
  font-weight: 700;
  margin-bottom: 0.7rem;
  text-align: center;
}
.modal p {
  color: #222;
  font-size: 1.07rem;
  margin-bottom: 0.5rem;
  text-align: left;
  width: 100%;
}
.modal p strong {
  color: #8f5cff;
  font-weight: 600;
}
.no-poster {
  width: 100px;
  height: 140px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f3f3f3;
  border-radius: 10px;
  margin-bottom: 1.1rem;
}
.modal-no-poster {
  width: 130px;
  height: 185px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f3f3f3;
  border-radius: 10px;
  margin-bottom: 1.3rem;
}

@media (prefers-color-scheme: dark) {
  .movie-search {
    background: #181a1b;
  }
  .search-bar {
    background: #232526;
    box-shadow: 0 2px 24px rgba(0,0,0,0.32);
    color: #f3f3f3;
  }
  h2 {
    color: #f3f3f3;
  }
  input[type="text"] {
    background: #232526;
    color: #f3f3f3;
    border: 1.5px solid #33393d;
    box-shadow: 0 1px 2px rgba(0,0,0,0.18);
  }
  input[type="text"]:focus {
    border: 1.5px solid #8f5cff;
    background: #232526;
    color: #fff;
    box-shadow: 0 2px 8px rgba(143,92,255,0.13);
  }
  button {
    background: linear-gradient(90deg, #8f5cff 60%, #a084e8 100%);
    color: #fff;
    box-shadow: 0 1px 4px rgba(143,92,255,0.18);
  }
  button:disabled {
    background: #2d223a;
    color: #aaa;
  }
  .loading {
    color: #a084e8;
  }
  .error {
    color: #ff6b6b;
  }
  .results-container {
    background: transparent;
  }
  .movies-grid {
    background: transparent;
  }
  .movie-card {
    background: #232526;
    color: #f3f3f3;
    border: 1.5px solid #232526;
    box-shadow: 0 2px 12px rgba(0,0,0,0.22);
  }
  .movie-card:hover {
    border: 1.5px solid #8f5cff;
    box-shadow: 0 6px 24px rgba(143,92,255,0.22);
  }
  .movie-card img {
    background: #181a1b;
    box-shadow: 0 1px 6px rgba(0,0,0,0.22);
  }
  .movie-title {
    color: #f3f3f3;
  }
  .movie-year {
    color: #b5b5b5;
  }
  .modal-overlay {
    background: rgba(0,0,0,0.72);
  }
  .modal {
    background: #232526;
    color: #f3f3f3;
    box-shadow: 0 10px 40px rgba(0,0,0,0.44);
  }
  .modal h3 {
    color: #fff;
  }
  .modal p {
    color: #e0e0e0;
  }
  .modal p strong {
    color: #a084e8;
  }
  .close-btn {
    background: transparent !important;
    color: #e0e0e0;
    box-shadow: none !important;
  }
  .close-btn:hover, .close-btn:focus {
    background: transparent !important;
    color: #ff4d4f;
    box-shadow: none !important;
    outline: none;
  }
  .no-poster {
    background: #181a1b;
  }
  .modal-no-poster {
    background: #181a1b;
  }
}
</style> 