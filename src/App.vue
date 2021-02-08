<template>
  <div class="App">
    <header class="App-header">
      <img src="{logo}" class="App-logo" alt="logo" />
      <!-- {!this.state.token && ( -->
      <a v-if="!token" class="btn btn--loginApp-link" :href="returnAuthSpotify">
        Login to Spotify
      </a>
      <!-- )} -->
      <!-- {this.state.token && !this.state.no_data && ( -->
      <Player v-if="!token"
        item="{this.state.item}"
        is_playing="{this.state.is_playing}"
        progress_ms="{this.state.progress_ms}"
      />
      <!-- )} -->
      <!-- {this.state.no_data && ( -->
      <p v-if="!no_data">
        You need to be playing a song on Spotify, for something to appear here.
      </p>
      <!-- )} -->
    </header>
  </div>
</template>

<script>
import Player from './Player';
import hash from './hash';
import { authEndpoint, clientId, redirectUri, scopes } from './config';
import * as $ from 'jquery';
import logo from './logo.svg';
import './App.css';
import axios from 'axios';

export default {
  data() {
    return {
      token: null,
      item: {
        album: {
          images: [{ url: '' }],
        },
        name: '',
        artists: [{ name: '' }],
        duration_ms: 0,
      },
      is_playing: 'Paused',
      progress_ms: 0,
      no_data: false,
    };
  },
  methods: {
    async getCurrentlyPlaying() {
      const { data } = await axios.get('https://api.spotify.com/v1/me/player', {
        headers: {
          Authorization: `Bearer ${token}`,
        },
      });
      // Checks if the data is not empty
      if (!data) {
        this.no_data = true;
        return;
      }
      this.item = data.item;
      this.is_playing = data.is_playing;
      this.progress_ms = data.progress_ms;
      this.no_data = false; /* We need to "reset" the boolean, in case the
                            user does not give F5 and has opened his Spotify. */
    },
    tick() {
      if (this.token) {
        this.getCurrentlyPlaying(this.token);
      }
    },
  },
  computed: {
    returnAuthSpotify: () =>
      `${authEndpoint}?client_id=${clientId}&redirect_uri=${redirectUri}&scope=${scopes.join(
        '%20'
      )}&response_type=token&show_dialog=true`,
  },
  mounted() {
    // Set token
    let _token = hash.access_token;

    if (_token) {
      // Set token
      this.setState({
        token: _token,
      });
      this.getCurrentlyPlaying(_token);
    }

    // set interval for polling every 5 seconds
    this.interval = setInterval(() => this.tick(), 5000);
  },
  beforeDestroy() {
    // clear the interval to save resources
    clearInterval(this.interval);
  },
};
</script>

`${authEndpoint}?client_id=${clientId}&redirect_uri=${redirectUri}&scope=${scopes.join(
"%20" )}&response_type=token&show_dialog=true`
