## Spotify Album Finder 🦇

Remake of a [codedex tutorial](https://www.codedex.io/projects/build-an-album-finder-with-spotify-api).

I used Vue, Tailwind CSS and Axios alongside the Spotify API.

## Setup

1. Clone the repository:
   ```sh
   git clone https://github.com/Falkern/spotify-album-search.git
   ```
2. Navigate to the project directory:
   ```sh
   cd spotify-album-search
   ```
3. Install dependencies:
   ```sh
   npm install
   ```
4. Create a `.env` file in the root directory and add your Spotify API credentials:
   ```env
   VUE_APP_SPOTIFY_CLIENT_ID=your_client_id
   VUE_APP_SPOTIFY_CLIENT_SECRET=your_client_secret
   ```
5. Run the development server:
   ```sh
   npm run serve
   ```

## Usage

1. Open your browser and go to `http://localhost`.
2. Enter the name of an artist in the search bar.
3. View the list of albums displayed in a grid format.

## Acknowledgements

- [Spotify API](https://developer.spotify.com/documentation/web-api/)
- [Vue.js](https://vuejs.org/)
- [Tailwind CSS](https://tailwindcss.com/)
- [Axios](https://axios-http.com)
