
<template>
  <div>
    <div class="input-container">
      <div class="container">
        <div class="error-message-box">
          <div class="error-message" v-show="error">{{ error }}</div>
        </div>
        <div class="form">
          <div class="input-field">
            <input type="text" placeholder="Enter a location" id="autocomplete" @keyup.enter="onEnterKey" />
            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-geo-alt"
              viewBox="0 0 16 16" @click="getUserLocation">
              <path
                d="M12.166 8.94c-.524 1.062-1.234 2.12-1.96 3.07A31.493 31.493 0 0 1 8 14.58a31.481 31.481 0 0 1-2.206-2.57c-.726-.95-1.436-2.008-1.96-3.07C3.304 7.867 3 6.862 3 6a5 5 0 0 1 10 0c0 .862-.305 1.867-.834 2.94zM8 16s6-5.686 6-10A6 6 0 0 0 2 6c0 4.314 6 10 6 10z" />
              <path d="M8 8a2 2 0 1 1 0-4 2 2 0 0 1 0 4zm0 1a3 3 0 1 0 0-6 3 3 0 0 0 0 6z" />
            </svg>
          </div>
          <button class="button" @click="onClick">Go</button>
        </div>
      </div>
      <div class="pagination-table">
        <div class="time">

          <div v-if="this.results.length > 0">The latest searched location is: {{ this.results[this.results.length -
            1].location }}</div>
          <div v-else>The latest searched location is: </div>
          <div>Timezone: {{ this.timeZone }}</div>
          <div>Time: {{ this.time }}</div>
        </div>
        <table>
          <thead>
            <tr>
              <th><button @click="deleteResult">Delete</button></th>
              <th>Number</th>
              <th>Location</th>
              <th>Latitude</th>
              <th>Longitude</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in displayData" :key="index">
              <td><input type="checkbox" v-model="item.selected" /></td>
              <td>{{ this.currentPage * pageSize + index + 1 }}</td>
              <td> {{ item.location }}</td>
              <td> {{ item.latitude }}</td>
              <td> {{ item.longitude }}</td>
            </tr>
          </tbody>
        </table>
        <div class="arrow">
          <svg :class="{ 'disabled': isFirstPage }" xmlns="http://www.w3.org/2000/svg" width="16" height="16"
            fill="currentColor" class="bi bi-arrow-left" viewBox="0 0 16 16" @click="prevPage">
            <path fill-rule="evenodd"
              d="M15 8a.5.5 0 0 0-.5-.5H2.707l3.147-3.146a.5.5 0 1 0-.708-.708l-4 4a.5.5 0 0 0 0 .708l4 4a.5.5 0 0 0 .708-.708L2.707 8.5H14.5A.5.5 0 0 0 15 8z" />
          </svg>
          <svg :class="{ 'disabled': isLastPage }" xmlns="http://www.w3.org/2000/svg" width="16" height="16"
            fill="currentColor" class="bi bi-arrow-right" viewBox="0 0 16 16" @click="nextPage">
            <path fill-rule="evenodd"
              d="M1 8a.5.5 0 0 1 .5-.5h11.793l-3.147-3.146a.5.5 0 0 1 .708-.708l4 4a.5.5 0 0 1 0 .708l-4 4a.5.5 0 0 1-.708-.708L13.293 8.5H1.5A.5.5 0 0 1 1 8z" />
          </svg>
        </div>
      </div>
    </div>
    <div class="map" id="map"></div>
  </div>
</template>

<script>

import axios from 'axios';

export default {
  data() {
    return {
      location: "",
      locationLat: "",
      locationLng: "",
      error: "",
      map: "",
      results: [],
      marker: "",
      currentPage: 0,
      pageSize: 10,
      timeZone: "",
      time: "",
    }
  },

  computed: {
    // Display the first ten records
    displayData() {
      return this.results.slice(this.currentPage * this.pageSize, (this.currentPage + 1) * this.pageSize)
    },

    // If it's the first page, make the button disabled.
    isFirstPage() {
      return this.currentPage === 0;
    },

    // If it's the last page, make the button disabled.
    isLastPage() {
      const totalPage = Math.ceil(this.results.length / this.pageSize);

      return this.currentPage >= totalPage - 1;
    }
  },

  mounted() {
    // Add autocomplete to the input blank
    const autocomplete = new google.maps.places.Autocomplete(
      document.getElementById("autocomplete")
    );

    autocomplete.addListener("place_changed", () => {
      const place = autocomplete.getPlace()
      this.locationLat = place.geometry.location.lat()
      this.locationLng = place.geometry.location.lng()
      this.location = place.formatted_address;
    });

    // Create a map object
    const map = new google.maps.Map(document.getElementById("map"), {
      zoom: 15,
      center: new google.maps.LatLng(42.9849233, -81.2452768),
      mapTypeControl: false,
    });
    this.map = map;
  },

  methods: {
    onClick() {
      this.showUserLocation(this.locationLat, this.locationLng)
    },

    onEnterKey() {
      this.onClick()
    },

    getUserLocation() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(position => {
          // Change from latitude and longitude to physical address
          // this.getAddressForm(position.coords.latitude, position.coords.longitude);

          // Marker the location on the map
          this.showUserLocation(position.coords.latitude, position.coords.longitude)
        },
          error => {
            this.error = error.message
          })
      } else {
        this.error = "Geolocation is not supported by your browser."
      }
    },

    // getAddressForm(latitude, longitude) {
    //   Determine the address via googleapi
    //   axios.get("https://maps.googleapis.com/maps/api/geocode/json?latlng="
    //     + latitude + "," + longitude
    //     + "&key=AIzaSyCS7yrYaKZF0JsV9XhGeWR69oAelgBv1eM").then(response => {
    //       if (response.data.error_message) {
    //         this.error = response.data.error_message;
    //       } else {
    //         console.log(response.data.results[0].formatted_address)
    //       }
    //     })
    //     .catch(error => {
    //       this.error = error.message
    //     })
    // },

    showUserLocation(latitude, longitude) {
      // Change the center
      this.map.setCenter(new google.maps.LatLng(latitude, longitude));

      // Add a marker
      const marker = new google.maps.Marker({
        position: new google.maps.LatLng(latitude, longitude),
        map: this.map
      });

      // Push a new search result
      const result = {
        location: this.location,
        latitude: latitude,
        longitude: longitude,
        selected: false,
        marker: marker
      }
      this.results.push(result)

      // Get the latest searched location time zone and local time
      this.getTime()

    },

    deleteResult() {
      this.results = this.results.filter((item) => {
        if (item.selected) {
          item.marker.setVisible(false);
        }
        return (!item.selected)
      });
      this.getTime()
    },

    nextPage() {
      this.currentPage += 1
    },

    prevPage() {
      this.currentPage -= 1
    },

    getTime() {
      if (this.results.length > 0) {
        const timestamp = Math.floor(Date.now() / 1000);
        axios.get("https://maps.googleapis.com/maps/api/timezone/json?location="
          + this.results[this.results.length - 1].latitude + "," + this.results[this.results.length - 1].longitude
          + "&timestamp=" + timestamp + "&key=AIzaSyCS7yrYaKZF0JsV9XhGeWR69oAelgBv1eM")
          .then(response => {
            this.timeZone = response.data.timeZoneId;
            const dateObj = new Date((Date.now() + 14400 * 1000 + response.data.dstOffset * 1000 + response.data.rawOffset * 1000));
            const year = dateObj.getFullYear();
            const month = dateObj.getMonth() + 1;
            const day = dateObj.getDate();
            const hours = dateObj.getHours();
            const minutes = dateObj.getMinutes();
            const seconds = dateObj.getSeconds();
            this.time = year + "-" + month + "-" + day + " " + hours + ":" + minutes + ":" + seconds;
          })
          .catch(error => {
            this.error = error.message;
          })
      } else {
        this.time = ""
        this.timeZone = ""
      }

    }
  },



}
</script>

<style>
.input-container {
  width: 600px;
  display: flex;
  height: 100vh;
  flex-direction: column;
  align-items: center;
}

.container {
  border: 1px solid #dad9d8;
  width: 300px;
  height: auto;
  margin-top: 30px;
  display: flex;
  flex-direction: column;
  align-items: center;
  border-radius: 5px;
  z-index: 10;
  position: relative;
  background-color: #fff;
}

.error-message-box {
  height: 30px;
  width: 80%;
  border: 1px solid #a7636a;
  margin-top: 20px;
  background-color: #f4dfda;
  border-radius: 5px;
  overflow: hidden;
}

.error-message {
  text-align: center;
  line-height: 30px;
  color: red;
}

.form {
  border: 1px solid #dad9d8;
  border-radius: 5px;
  width: 80%;
  display: flex;
  flex-direction: column;
  margin: 20px 0;
  padding: 10px;
  align-items: flex-start;
}

.input-field {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
}

.input-field input {
  width: 100%;
  height: 25px;
  padding: 5px;
}

.input-field svg {
  transform: scale(1.5);
  margin-left: 10px;
}

.button {
  border: none;
  border-radius: 5px;
  color: #fff;
  background-color: #df3e42;
  margin-top: 20px;
  width: 60px;
  height: 30px;
}

.button:hover,
svg:hover {
  cursor: pointer;
}

.pac-item {
  cursor: pointer;
}

.pagination-table {
  margin: 50px;
  background-color: #fff;
  z-index: 10;
}

table {
  border: 1px solid black;
  border-collapse: collapse;
  border-radius: 5px;
}

th,
td {
  border-right: 1px solid black;
  padding: 5px;
  text-align: center;
  border-bottom: 1px solid black;
}

.arrow {
  width: 20%;
  margin: auto;
  margin-top: 10px;
}

.arrow svg {
  padding: 5px;
  border: 1px solid black;
  background-color: #f5f5f5;
}

.arrow svg:first-child {
  margin-right: 20px;
}

.disabled {
  opacity: 0.5;
  pointer-events: none;
}

.time {
  padding: 5px;
  line-height: 1.5;
  font-weight: 600;
}


.map {
  position: absolute;
  right: 0;
  top: 0;
  left: 0;
  bottom: 0;
}
</style>

