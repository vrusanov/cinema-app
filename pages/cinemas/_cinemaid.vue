<template>
  <Loading v-if="$fetchState.pending || isLoading"/>
  <div v-else class="container single-cinema">
    <NuxtLink class="button" :to="{ name: 'index' }">Back</NuxtLink>
    <div class="cinema-info-box">
      <span>
          <h1>{{ cinema.data[0].name }}</h1>
          <img width="330" height="440" :src="cinema.data[0].image" :alt="`${cinema.data[0].name}-poster`">
      </span>
      <div class="cinema-page-info">
        <div class="cinema-item-list" v-html="cinema.data[0].additional"></div>
      </div>
    </div>

    <div class="cinema-booking">
      <div class="date-time-select-bar">
        <div class="date-select-list">
          <p>{{ selectedDate }}</p>
          <v-radio-group
            v-model="selectedDate"
            mandatory
            :row="true"
            @change="getTimeForSelect(selectedDate)"
          >
            <v-radio
              v-for="(date, index) in dates" :key="index"
              :label="date"
              :value="date"
            ></v-radio>
          </v-radio-group>
        </div>
        <div class="time-select-list">
          <p>{{ selectedTime }}</p>
          <v-radio-group
            v-model="selectedTime"
            mandatory
            :row="true"
            @change="checkPlaces"
          >
            <v-radio
              v-for="(time) in arrTimeForSelectedDate" :key="time"
              :label="time"
              :value="time"
            ></v-radio>
          </v-radio-group>
        </div>
      </div>
      <div class="cinema-book-place">
        <p>Row - {{ bookPlaceData.row }}</p>
        <p>Seat - {{ bookPlaceData.seat }}</p>
        <span v-for="(elem, index) in placesForBooking" :key=index class="row-cinema-place" :data-row="index">
        {{ index }}
        <span v-for="(seat, index) in elem" :key=index class="seat-cinema-place">
            <v-btn class="seat" :disabled="!seat.is_free" @click="addSeatPlace(seat.seat)"> {{
                seat.seat
              }}</v-btn>
        </span>
      </span>
      </div>
      <v-btn
        class="ma-2"
        :loading="isLoading"
        :disabled="isLoading"
        color="success"
        @click="bookSelectedPlace"
      >
        Book place
      </v-btn>
    </div>
  </div>
</template>
<script>
import axios from "axios";

export default {
  name: 'single-cinema',
  data() {
    return {
      cinema: null,
      isLoading: false,
      selectedTime: '',
      selectedDate: '',
      isLoadingPlaces: false,
      placesForBooking: [],
      cinemaBookingInfo: [],
      arrTimeForSelectedDate: [],
      modifyCinemaBookingInfo: {},
      cinemaId: this.$route.params.cinemaid,
      bookPlaceData: {
        "movie_id": this.$route.params.cinemaid,
        "row": null,
        "seat": null,
        "showdate": '',
        "daytime": ''
      },
    }
  },
  async fetch() {
    await this.getSingleMovieInfo();
    await this.getBookingInfo();
  },
  computed: {
    dates() {
      return Object.keys(this.modifyCinemaBookingInfo)
    },
  },
  methods: {
    async getSingleMovieInfo() {
      try {
        const {data} = await axios.get(`https://cinema-api-test.y-media.io/v1/movies?movie_id=${this.cinemaId}`)
        this.cinema = data
      } catch (err) {
        console.log(err)
      }
    },
    async getBookingInfo() {
      this.isLoading = true;
      try {
        const {data} = await axios.get(`https://cinema-api-test.y-media.io/v1/movieShows?movie_id=${this.cinemaId}`)
        data.data[this.$route.params.cinemaid].forEach((elem) => {
          this.cinemaBookingInfo.push(elem);
        })
        await this.modifyArr(this.cinemaBookingInfo);
      } catch (err) {
        console.log(err)
      } finally {
        this.isLoading = false;
      }
    },
    modifyArr(arr) {
      this.modifyCinemaBookingInfo = arr.reduce((acc, current) => {
        acc[current.showdate] = current.daytime.split(';')
        return acc;
      }, {});
    },
    async checkPlaces() {
      this.isLoadingPlaces = true;
      this.placesForBooking = {};
      try {
        const {data} = await axios.get(`https://cinema-api-test.y-media.io/v1/showPlaces?movie_id=${this.cinemaId}&daytime=${this.selectedTime}&showdate=${this.selectedDate}`);
        this.selectedTimeAndDate();
        this.placesForBooking = data?.data?.reduce((acc, [row, ...arr]) => {
          acc[row.row] = arr[0]
          return acc
        }, {})
      } catch (err) {
        console.log(err)
      } finally {
        this.isLoadingPlaces = false;
      }
    },
    getTimeForSelect(selectedDate) {
      this.arrTimeForSelectedDate = [];
      const tempTimeArr = this.modifyCinemaBookingInfo[selectedDate]
      tempTimeArr.forEach((elem) => {
        this.arrTimeForSelectedDate.push(elem);
      })
    },
    addSeatPlace(seat) {
      this.bookPlaceData.seat = seat;
      this.bookPlaceData.row = Number(event.target.closest('.row-cinema-place')
        .getAttribute('data-row'));
    },
    selectedTimeAndDate() {
      this.bookPlaceData.daytime = this.selectedTime;
      this.bookPlaceData.showdate = this.selectedDate;
    },
    async bookSelectedPlace() {
      try {
        if (this.bookPlaceData) {
          const {data} = await axios.post('https://cinema-api-test.y-media.io/v1/bookPlace', this.bookPlaceData)
          console.log(data)
        }
      } catch (err) {
        console.log(err)
      }
    }
  },
  fetchDelay: 1000,
}
</script>
<style lang="scss">
.single-cinema {
  display: flex;
  flex-flow: column;

  .button {
    color: #fff;
    width: fit-content;
    padding-bottom: 14px;
  }

  .cinema-info-box {
    display: flex;
    flex-flow: row;
    border-bottom: 1px solid #fff;
    padding-bottom: 10px;

    span {
      display: flex;
      flex-flow: column;
      align-items: center;
    }

    .cinema-page-info {
      display: inline-block;

      .cinema-item-list {
        margin: 5px 15px;
        width: 100%;

        ul {
          list-style: none;
          padding: 0;

          .rating {
            color: yellow;
          }

          p {
            margin: 0;

            &.key {
              font-weight: bold;
            }

            &.val {
              font-style: italic;
            }
          }
        }
      }
    }
  }

  .date-select-bar {
    display: flex;

    .date-select-list {
      display: flex;
    }
  }

  .row-cinema-place {
    display: flex;
    justify-content: center;
    flex-flow: row wrap;
    align-items: center;

    .seat-cinema-place {
      padding: 2px;

      .seat {
        height: 25px;
        min-width: 35px;
        padding: 0 12px;
      }
    }
  }
}
</style>
