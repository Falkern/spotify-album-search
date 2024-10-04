<template>
  <div class="max-w-3xl mx-auto p-6 bg-black text-white rounded-lg shadow-lg">
    <div class="flex mb-6">
      <select
        v-model="searchType"
        class="border-2 border-gray-600 p-2 rounded-l-md w-1/3 bg-black text-white hover:border-white focus:outline-none transition duration-300"
      >
        <option value="artists">Artists</option>
        <option value="albums">Albums</option>
        <option value="songs">Songs</option>
      </select>
      <input
        v-model="searchInput"
        :placeholder="`Search For ${capitalizeFirstLetter(searchType)}`"
        class="border-2 border-gray-600 p-2 rounded-none w-2/3 bg-black text-white hover:border-white focus:outline-none transition duration-300"
        @input="debouncedSearch()"
      />
      <button
        class="bg-white text-black py-2 px-4 rounded-r-md hover:bg-gray-200 transition duration-300"
        @click="search"
      >
        Search
      </button>
      <button
        @click="clearSearch"
        class="ml-2 bg-gray-700 text-white py-2 px-4 rounded hover:bg-gray-600 transition duration-300"
      >
        Clear
      </button>
    </div>

    <div v-if="loading" class="text-white text-center" aria-live="polite">
      <span class="loader"></span> Loading...
    </div>
    <div v-if="errorMessage" class="text-red-600 text-center" aria-live="assertive">
      {{ errorMessage }}
    </div>

    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6 mt-4">
      <template v-if="searchType === 'albums'">
        <div
          v-for="album in albums"
          :key="album.id"
          class="album-card bg-gray-800 p-4 rounded-lg shadow-md hover:shadow-xl transition-shadow duration-300 transform hover:scale-105"
        >
          <img
            :src="album.images?.[0]?.url || 'path/to/default/image.png'"
            alt="Album Cover"
            class="w-full h-40 object-cover rounded mb-2 transition-transform duration-300 transform hover:scale-110"
          />
          <h3 class="font-bold text-lg transition duration-300 hover:text-gray-300">{{ album.name }}</h3>
          <p class="text-gray-400">Release Date: {{ album.release_date }}</p>
          <a
            :href="album.external_urls.spotify"
            class="text-gray-200 hover:text-white mt-2 inline-block transition duration-300"
            target="_blank"
            rel="noopener noreferrer"
          >Album Link</a>
        </div>
      </template>

      <template v-if="searchType === 'artists'">
        <div
          v-for="artist in artists"
          :key="artist.id"
          class="artist-card bg-gray-800 p-4 rounded-lg shadow-md hover:shadow-xl transition-shadow duration-300 transform hover:scale-105"
        >
          <img
            :src="artist.images?.[0]?.url || 'path/to/default/image.png'"
            alt="Artist"
            class="w-full h-40 object-cover rounded mb-2 transition-transform duration-300 transform hover:scale-110"
          />
          <h3 class="font-bold text-lg transition duration-300 hover:text-gray-300">{{ artist.name }}</h3>
          <a
            :href="artist.external_urls.spotify"
            class="text-gray-200 hover:text-white mt-2 inline-block transition duration-300"
            target="_blank"
            rel="noopener noreferrer"
          >Artist Link</a>
        </div>
      </template>

      <template v-if="searchType === 'songs'">
        <div
          v-for="song in songs"
          :key="song.id"
          class="song-card bg-gray-800 p-4 rounded-lg shadow-md hover:shadow-xl transition-shadow duration-300 transform hover:scale-105"
        >
          <img
            :src="song.album.images?.[0]?.url || 'path/to/default/image.png'"
            alt="Album Cover"
            class="w-full h-40 object-cover rounded mb-2 transition-transform duration-300 transform hover:scale-110"
          />
          <h3 class="font-bold text-lg transition duration-300 hover:text-gray-300">{{ song.name }}</h3>
          <p class="text-gray-400">Artist: {{ song.artists[0].name }}</p>
          <a
            :href="song.external_urls.spotify"
            class="text-gray-200 hover:text-white mt-2 inline-block transition duration-300"
            target="_blank"
            rel="noopener noreferrer"
          >Song Link</a>
        </div>
      </template>
    </div>
  </div>

  <footer class="mt-8 text-center text-gray-400">
      <p>
        &copy; {{ new Date().getFullYear() }} FALKERN. All rights reserved.
      </p>
      <a
        href="https://github.com/Falkern"
        target="_blank"
        rel="noopener noreferrer"
        class="hover:text-white transition duration-300"
      >
        Visit my GitHub
      </a>
    </footer>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      searchInput: '',
      accessToken: '',
      albums: [],
      artists: [],
      songs: [],
      searchType: 'artists', // Default search type
      loading: false,
      errorMessage: '',
    };
  },
  mounted() {
    this.getSpotifyToken();
    this.debouncedSearch = this.debounce(this.search, 300);
  },
  methods: {
    debounce(func, delay) {
      let timeout;
      return (...args) => {
        const context = this;
        clearTimeout(timeout);
        timeout = setTimeout(() => func.apply(context, args), delay);
      };
    },
    async getSpotifyToken() {
      const client_id = import.meta.env.VITE_CLIENT_ID;
      const client_secret = import.meta.env.VITE_CLIENT_SECRET;
      try {
        const response = await axios.post(
          'https://accounts.spotify.com/api/token',
          new URLSearchParams({
            grant_type: 'client_credentials',
            client_id,
            client_secret,
          }),
          {
            headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
          }
        );
        this.accessToken = response.data.access_token;
      } catch (error) {
        console.error('Error fetching access token:', error);
        this.errorMessage = 'Failed to get access token.';
      }
    },
    async search() {
      this.loading = true; // Start loading
      this.errorMessage = ''; // Reset error message

      try {
        const params = {
          headers: { Authorization: `Bearer ${this.accessToken}` },
        };

        // Determine the search type
        if (this.searchType === 'artists') {
          const artistResponse = await axios.get(
            `https://api.spotify.com/v1/search?q=${this.searchInput}&type=artist`,
            params
          );
          this.artists = artistResponse.data.artists.items;
          this.albums = [];
          this.songs = [];
        } else if (this.searchType === 'albums') {
          const albumResponse = await axios.get(
            `https://api.spotify.com/v1/search?q=${this.searchInput}&type=album`,
            params
          );
          this.albums = albumResponse.data.albums.items;
          this.artists = [];
          this.songs = [];
        } else if (this.searchType === 'songs') {
          const songResponse = await axios.get(
            `https://api.spotify.com/v1/search?q=${this.searchInput}&type=track`,
            params
          );
          this.songs = songResponse.data.tracks.items;
          this.albums = [];
          this.artists = [];
        }
      } catch (error) {
        console.error('Error fetching data:', error);
        this.errorMessage = 'An error occurred while fetching data. Please try again.';
      } finally {
        this.loading = false; // Stop loading
      }
    },
    clearSearch() {
      this.searchInput = '';
      this.albums = [];
      this.artists = [];
      this.songs = [];
      this.errorMessage = ''; // Reset error message
    },
    capitalizeFirstLetter(string) {
      return string.charAt(0).toUpperCase() + string.slice(1);
    },
  },
};
</script>

<style scoped>
.loader {
  border: 4px solid rgba(255, 255, 255, 0.2);
  border-radius: 50%;
  border-top: 4px solid white;
  width: 30px;
  height: 30px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

/* General body styles for B/W theme */
body {
  background-color: #000; /* Black background for the body */
  color: #fff; /* White text for the body */
  font-family: 'Poppins', sans-serif; /* Use the Poppins font for a modern look */
}

h1 {
  font-family: 'Poppins', sans-serif; /* Use the Poppins font for the title */
  color: white; /* Title color */
}

footer {
  margin-top: 2rem; /* Add some space at the top */
  padding: 1rem; /* Add padding */
  font-size: 0.875rem; /* Smaller font size */
}
</style>
