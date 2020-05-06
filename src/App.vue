<template>
  <div class="container-fluid">
    <div class="row">
      <div class="col-sm-10 col-md-6 col-lg-6 col-xl-6 main-container">
        <div class="form-group search_box">
          <span class="location_span">
            <img src="./assets/seo-and-web.svg" class="location_icon" />
          </span>
          <input
            class="form-control form-control-lg search_input"
            v-model="search"
            type="search"
            placeholder="search"
            @keyup.enter="searchWeather"
          />
          <span class="search_span" onclick="showWeather()">
            <img src="./assets/tools-and-utensils.svg" class="search_icon" @click="searchWeather" />
          </span>
        </div>

        <div id="loader"></div>
        <div id="myDiv">
          <div class="d-flex bd-highlight weather_info">
            <div class="p-2 flex-fill bd-highlight" v-for="date in date_info" :key="date.temp.day">
              <p>{{dateConverter(date.dt)}}</p>
              <p>{{parseFloat((date.temp.day - 273.15).toFixed())}}&#8451;</p>
              <img v-if="date.weather[0].main == 'Clear'" src="./assets/summertime.svg" />
              <img
                v-else-if="date.weather[0].main == 'Rain' || date.weather[0].main == 'Drizzle'|| date.weather[0].main == 'Thunderstorm'"
                src="./assets/meteorology.svg"
              />
              <img v-else-if="date.weather[0].main == 'Clouds'" src="./assets/meteorology (1).svg" />
              <img v-else-if="date.weather[0].main == 'Snow'" src="./assets/nature.svg" />
              <img v-else src="./assets/weather.svg" />
              <p>{{date.weather[0].main}}</p>
            </div>
          </div>
          <div class="temp_container">
            <div class="d-flex bd-highlight">
              <div class="p-2 flex-fill bd-highlight temp_div">
                <h1
                  class="temp_info"
                >{{parseFloat((this.daily_main.temp - 273.15).toFixed())}}&#8451;</h1>
              </div>
            </div>
            <div style="width: 100%;overflow: auto;margin-bottom:15px;">
              <tempgraph :categories="sortedArr" :data="sortedArr1" style="min-width: 1000px;"></tempgraph>
            </div>
            <!-- <div class="d-flex bd-highlight weather_info hour_data">
          <div class="p-2 flex-fill bd-highlight " v-for="hour in hourly_info" :key="hour.temp">
            <p>{{parseFloat((hour.temp - 273.15).toFixed())}}&#8451;</p>
            <p>{{timeConverter(hour.dt)}}</p>
          </div>
            </div>-->
            <div class="d-flex bd-highlight">
              <div class="p-2 flex-fill bd-highlight left_highlight">
                <h6>Pressure</h6>
                <h6>{{this.daily_main.pressure}} hpa</h6>
              </div>
              <div class="p-2 flex-fill bd-highlight right_highlight">
                <h6>Humidity</h6>
                <h6>{{this.daily_main.humidity}} %</h6>
              </div>
            </div>
            <div class="d-flex bd-highlight sun_info">
              <div class="p-2 flex-fill bd-highlight sunrise_div">
                <h6>Sunrise</h6>
                <h6>{{timeConverter(this.daily_sys.sunrise)}}</h6>
              </div>
              <div class="p-2 flex-fill bd-highlight sunset_div">
                <h6>Sunset</h6>
                <h6>{{timeConverter(this.daily_sys.sunset)}}</h6>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import Vue from "vue";
import axios from "axios";
import VueAxios from "vue-axios";
import TempGraph from "./components/graph";

Vue.use(VueAxios, axios);

export default {
  name: "App",
  components: {
    tempgraph: TempGraph
  },
  data() {
    return {
      appId: "9b66a9c4055d9d1f99e3bdcdd1dbfa69",
      daily_main: null,
      daily_sys: null,
      date_info: null,
      hourly_info: null,
      search: null,
      sortedArr: [],
      cityLat:null,
      cityLng:null
    };
  },
  mounted() {
    navigator.geolocation.getCurrentPosition(
      position => {
        console.log(position.coords.latitude);
        console.log(position.coords.longitude);
        this.getAddress(position.coords.latitude, position.coords.longitude);
        this.showWeather(position.coords.latitude, position.coords.longitude);
        this.getCityWeather(position.coords.latitude, position.coords.longitude);
      },
      error => {
        console.log(error.message);
      }
    );
  },

  methods: {
    searchWeather() {
      if (this.search != null) {
        document.getElementById("loader").style.display = "block";
        document.getElementById("myDiv").style.opacity = "0.1";
        axios
          .get(
            "https://api.openweathermap.org/data/2.5/weather?q=" +
              this.search +
              "&APPID=" +
              this.appId
          )
          .then(response => {
            console.log("111", response);
            this.daily_main = response.data.main;
            this.daily_sys = response.data.sys;

            document.getElementById("loader").style.display = "none";
            document.getElementById("myDiv").style.opacity = "1";
            // this.timeConverter(response.data.dt)
          })
          .catch(error => {
            console.log(error, "er");
          });
        axios
          .get(
            "https://maps.googleapis.com/maps/api/geocode/json?address=" +
              this.search +
              "&key=" +
              "AIzaSyCLrHcJWSP_RzIHqG3YuWX8FJ_PzD3R2Fw"
          )
          .then(response => {
            console.log("33", response.data.results[0].geometry.location.lat);
            this.cityLat = response.data.results[0].geometry.location.lat
                 this.cityLng = response.data.results[0].geometry.location.lng
            // this.cityName = JSON.parse(response.data.results[0].geometry.loction);
            
            console.log("city", this.cityLat);
             if (this.cityLat != null && this.cityLng != null) {
          console.log('cityweather')
          this.getCityWeather(this.cityLat ,this.cityLng);
        }
          })
          .catch(error => {
            console.log(error, "er");
          });

       
      }
    },
    dateConverter(UNIX_timestamp) {
      var a = new Date(UNIX_timestamp * 1000);
      var weekDays = ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"];
      var day = weekDays[a.getDay()];
      var time = day;

      return time;
    },
    timeConverter(UNIX_timestamp) {
      const milliseconds = UNIX_timestamp * 1000; // 1575909015000

      const dateObject = new Date(milliseconds);

      const humanDateFormat = dateObject.toLocaleString("en-US", {
        hour: "numeric",
        minute: "numeric"
      });

      return humanDateFormat;
    },
    showWeather(lat, lon) {
      axios
        .get(
          "https://api.openweathermap.org/data/2.5/weather?lat=" +
            lat +
            "&lon=" +
            lon +
            "&APPID=" +
            this.appId
        )
        .then(response => {
          this.daily_main = response.data.main;
          this.daily_sys = response.data.sys;

          // this.timeConverter(response.data.dt)
        })
        .catch(error => {
          console.log(error, "er");
        });
    },
    getAddress(latitude, longitude) {
      console.log("ewfwef");
      axios
        .get(
          "https://maps.googleapis.com/maps/api/geocode/json?latlng=" +
            latitude +
            "," +
            longitude +
            "&key=" +
            "AIzaSyCLrHcJWSP_RzIHqG3YuWX8FJ_PzD3R2Fw"
        )
        .then(response => {
          console.log(response);
        })
        .catch(error => {
          console.log(error, "er");
        });
    },
    getCityWeather(lat,lon) {
      console.log('lat',lat,lon)
      axios
        .get(
          "https://api.openweathermap.org/data/2.5/onecall?lat=" +
            lat +
            "&lon=" +
            lon +
            "&exclude=" +
            "&APPID=" +
            this.appId
        )
        .then(response => {
          this.date_info = response.data.daily;
          this.hourly_info = response.data.hourly;
          console.log("hr", this.hourly_info);
          let temp_arr = [];
          let temp_arr1 = [];
          for (let d of this.hourly_info) {
            temp_arr.push(
              parseFloat((d.temp - 273.15).toFixed()) +
                " " +
                "C" +
                " " +
                this.timeConverter(d.dt)
            );
            temp_arr1.push(parseFloat((d.temp - 273.15).toFixed()));
          }
          this.sortedArr = temp_arr;
          this.sortedArr1 = temp_arr1;
        })
        .catch(error => {
          console.log(error, "er");
        });
    }

    // ipLookUp () {
    //   $.ajax('http://ip-api.com/json')
    //   .then(
    //       function success(response) {
    //           console.log('User\'s Location Data is ', response);
    //           console.log('User\'s Country', response.country);
    //           getAdress(response.lat, response.lon)
    // },

    //       function fail(data, status) {
    //           console.log('Request failed.  Returned status of',
    //                       status);
    //       }
    //   );
    // }
  }
};
</script>

<style>
body {
  background: url("https://image.freepik.com/free-photo/blue-day-clear-sky-with-light-clouds-smooth-blue-white-gradient-sky-wonderful-weather-background-morning-heaven-morning-with-copyspace-slightly-cloudy-backdrop-atmosphere-clear-day_102332-1718.jpg");
  background-size: cover;
  background-repeat: no-repeat;
  overflow-y: auto;
}
.main-container {
  overflow-y: auto;
  background-color: #fff;
  border-radius: 15px;

  /* margin: 0;
  position: absolute;
  top: 50%;
  left: 50%;
  -ms-transform: translate(-50%, -50%);
  transform: translate(-50%, -50%); */
  padding: 15px;
  box-shadow: 0 4px 5px 3px rgba(0, 0, 0, 0.11);
  margin-left: auto;
  margin-right: auto;
  margin-top: 5%;
  margin-bottom: 5%;
}

.location_icon {
  width: 20px;
  height: 20px;
}
.location_span {
  position: absolute;
  left: 10px;
  top: 10px;
}
.search_icon {
  width: 20px;
  height: 20px;
}
.search_span {
  position: absolute;
  right: 10px;
  top: 10px;
}
.search_box {
  position: relative;
}
.search_input,
.search_input:focus {
  padding-left: 35px;
  box-shadow: 0 4px 5px 3px rgba(0, 0, 0, 0.11);
  /* border: .5px solid #c9c9c9; */
  border-radius: 0.5rem;
}
.temp_container {
  box-shadow: 0 4px 5px 3px rgba(0, 0, 0, 0.11);
  /* border: .5px solid #c9c9c9; */
  border-radius: 0.5rem;
  /* margin: 0px; */
  padding-right: 15px;
  padding-left: 15px;
}
.temp_info {
  font-size: 5.5rem;
  font-weight: 600;
}
/* .temp_container_div{
  padding-left: 0px;
} */
.highlight_div {
  background-color: #000;
}
.left_highlight,
.right_highlight {
  background-color: #e8f0f8;
  padding: 10px 15px !important;
}
.left_highlight {
  margin-right: 15px;
}
.right_highlight {
  margin-left: 15px;
}
.temp_div {
  padding: 0px !important;
}
.sunrise_div {
  padding-left: 0px !important;
}
.sunset_div {
  padding-right: 0px !important;
  text-align: right;
}
.sun_info {
  margin-top: 30px;
}
.weather_info {
  overflow-x: auto;
}
.weather_info .p-2 {
  padding: 0.8rem !important;
  text-align: center;
}
.weather_info p {
  margin-bottom: 0.5rem !important;
}
.weather_info img {
  width: 50px;
  height: 50px;
}
#loader {
  display: none;
  position: absolute;
  left: 50%;
  top: 50%;
  z-index: 1;
  width: 100px;
  height: 100px;
  margin: -50px 0 0 -50px;
  border: 8px solid #f3f3f3;
  border-radius: 50%;
  border-top: 8px solid #3498db;
  /* width: 120px;
  height: 120px; */
  -webkit-animation: spin 1s linear infinite;
  animation: spin 1s linear infinite;
}

@-webkit-keyframes spin {
  0% {
    -webkit-transform: rotate(0deg);
  }
  100% {
    -webkit-transform: rotate(360deg);
  }
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
::-webkit-scrollbar {
  width: 1px;
  height: 1px;
}

/* Track */
::-webkit-scrollbar-track {
  box-shadow: inset 0 0 1px #f4f4f4;
  border-radius: 1px;
}
::-webkit-scrollbar-thumb {
  background: #3498db;
  border-radius: 1px;
}
.hour_data {
  margin-bottom: 15px;
}
</style>
