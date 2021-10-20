<template>
  <div>
    <!--Auto import components true-->
    <!--Header Component-->
    <HeaderPart/>
    <!-- Search panel -->
    <div class="container search">
      <input type="text" placeholder="Search by Name" v-model.lazy="searchInput" @keyup.enter="$fetch">
      <button v-show="searchInput !== ''" class="button" @click="clearSearch">Clear Search</button>
    </div>
    <!--Loading Spinner   -->
    <Loading v-if="$fetchState.pending" />
    <!--Cinemas List Component-->
    <!--Searched movies-->
    <div v-else class="container cinemas">
      <div v-if="searchInput !== ''" id="cinema-grid id">
        <div class="cinemas-grid">
          <div v-for="(cinema, index) in searchedMovies" :key="index" class="cinema">
            <div class="cinema-img">
              <img width="230" height="340" :src="cinema.image" :alt="`${cinema.name}-poster`">
              <span v-if="delSpecSymbol" class="overview">{{ delSpecSymbol(cinema.description) }}</span>
            </div>
            <div class="info-cinema">
              <p class="title">{{ cinema.name }}</p>
              <NuxtLink class="button button-light"
                        :to="{name: 'cinemas-cinemaid', params: {cinemaid: cinema.id}}">Get More Info
              </NuxtLink>
            </div>
          </div>
        </div>
      </div>
      <div v-else id="cinema-grid">
        <div class="cinemas-grid">
          <div v-for="(cinema, index) in movies" :key="index" class="cinema">
            <div class="cinema-img">
              <img width="230" height="340" :src="cinema.image" :alt="`${cinema.name}-poster`">
              <span v-if="delSpecSymbol" class="overview">{{ delSpecSymbol(cinema.description) }}</span>
            </div>
            <div class="info-cinema">
              <p class="title">{{ delSpecSymbol(cinema.name) }}</p>
              <NuxtLink class="button button-light"
                        :to="{name: 'cinemas-cinemaid', params: {cinemaid: cinema.id}}">Get More Info
              </NuxtLink>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import axios from 'axios'

export default {
  data() {
    return {
      movies: [],
      searchInput: '',
      searchedMovies: [],
    }
  },
  async fetch() {
    try {
      if (this.searchInput === '') {
        await this.getAllMovies();
        return
      }
      if (this.searchInput !== '') {
        await this.searchMovies()
      }
    } catch (err) {
      console.log(err)
    }
  },
  fetchDelay: 1000,
  methods: {
    async getAllMovies() {
      try {
        const res = await axios.get('https://cinema-api-test.y-media.io/v1/movies')
         res.data?.data?.forEach((elem) => {
          this.movies.push(elem)
        })
      } catch(err) {
        console.log(err)
      }
    },
    async searchMovies() {
      try {
        const res = await axios.get(`https://cinema-api-test.y-media.io/v1/movies?name=${this.searchInput}`)
        res.data?.data?.forEach((elem) => {
          this.searchedMovies.push(elem)
        })
      } catch (err) {
        console.log(err)
      }
    },
    delSpecSymbol(str) {
      return str ? `${str.replace(/<\/?[^>]+(>|$)/g, "")
          .replace(/&/g, "&amp;")
          .replace(/>/g, "&gt;")
          .replace(/</g, "&lt;")
          .replace(/"/g, "&quot;")
          .slice(0, 1170)}`
        : false;
    },
    clearSearch() {
      this.searchInput = '';
      this.searchedMovies = [];
    }
  }
}

</script>
<style lang="scss" scoped>
.loading {
  padding-top: 120px;
  align-items: flex-start;
}

.search {
  display: flex;
  padding: 32px 0;

  input {
    max-width: 350px;
    width: 100%;
    padding: 12px 6px;
    font-size: 14px;
    border: none;
    color: black;
    background-color: #fff;

    &:focus {
      outline: none;
    }
  }

  .button {
    margin-left: 10px;
    border-top-left-radius: 0;
    border-bottom-left-radius: 0;
  }
}

.cinemas {
  padding: 32px 0;

  .cinemas-grid {
    display: grid;
    column-gap: 32px;
    row-gap: 64px;
    grid-template-columns: 1fr;
    @media (min-width: 500px) {
      grid-template-columns: repeat(2, 1fr);
    }
    @media (min-width: 750px) {
      grid-template-columns: repeat(2, 1fr);
    }
    @media (min-width: 1100px) {
      grid-template-columns: repeat(4, 1fr);
    }


    .cinema {
      position: relative;
      display: flex;
      flex-direction: column;

      img {
        display: block;
        width: 100%;
        height: 100%;
      }

      .cinema-img {
        position: relative;
        overflow: hidden;

        .overview {
          line-height: 1.5;
          position: absolute;
          bottom: 0;
          background-color: rgba(201, 38, 2, 0.9);
          padding: 12px;
          color: #fff;
          transform: translateY(100%);
          transition: 0.3s ease-in-out all;
        }

        &:hover {
          .overview {
            transform: translateY(0);
            width: 100%;
          }
        }
      }
    }
  }

  .info-cinema {
    margin-top: auto;

    .title {
      margin-top: 8px;
      color: #fff;
      font-size: 20px;
    }

    .release {
      margin-top: 8px;

      & a {
        color: #c9c9c9;
      }
    }

    .button {
      margin-top: 8px;
      color: #fff;
    }
  }
}
</style>
