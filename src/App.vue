<template>
  <div class="max-w-xl mx-auto p-4 bg-black text-white">
    <div class="flex mb-4">
      <input
        v-model="searchInput"
        placeholder="Search For Artist"
        class="border-2 border-gray-300 p-2 rounded-l w-full focus:outline-none focus:border-white transition duration-300 bg-black text-white"
        @keyup.enter="search"
      />
      <button class="bg-white text-black py-2 px-4 rounded-r" @click="search">
        Search
      </button>
    </div>

    <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
      <div v-for="album in albums" :key="album.id" class="bg-gray-800 p-4 rounded shadow">
        <img :src="album.images[0].url" alt="Album Cover" class="w-full rounded" />
        <h3 class="font-bold mt-2">{{ album.name }}</h3>
        <p class="text-gray-400">Release Date: {{ album.release_date }}</p>
        <a :href="album.external_urls.spotify" class="text-white mt-2 inline-block">Album Link</a>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      searchInput: '',
      accessToken: '',
      albums: [],
    };
  },
  mounted() {
    this.getSpotifyToken();
  },
  methods: {
    async getSpotifyToken() {
      const client_id = import.meta.env.VITE_CLIENT_ID;
      const client_secret = import.meta.env.VITE_CLIENT_SECRET;
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
    },
    async search() {
      const artistParams = {
        headers: { Authorization: `Bearer ${this.accessToken}` },
      };

      const artistID = await axios.get(
        `https://api.spotify.com/v1/search?q=${this.searchInput}&type=artist`,
        artistParams
      ).then(result => result.data.artists.items[0]?.id);

      if (artistID) {
        const albumResponse = await axios.get(
          `https://api.spotify.com/v1/artists/${artistID}/albums?include_groups=album&market=US&limit=50`,
          artistParams
        );
        this.albums = albumResponse.data.items;
      } else {
        this.albums = []; 
      }
    },
  },
};
</script>

<style scoped>
.bg-black {
  background-color: #000;
}
.text-white {
  color: #fff;
}
.bg-gray-800 {
  background-color: #2d2d2d;
}
.text-gray-400 {
  color: #b3b3b3;
}
</style>

<style>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap');

body {
  background-color: #000;
  font-family: 'Poppins', sans-serif;
  color: #fff;
}
</style>
