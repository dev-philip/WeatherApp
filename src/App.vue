<template>
  <div id="app" :class="typeof weather.main != 'undefined' && weather.main.temp > 16 ? 'warm' : ''">
    <main>
      <div class="search-box">
        <input 
        type="text" 
        class="search-bar" 
        placeholder="Search By Location or City ......"
        v-model="query"
        @keypress="getWeather"/>
      </div>
      
      <!-- {{query}} -->

      <div class="weather-wrap animate__animated animate__fadeIn" v-if="typeof weather.main != 'undefined'">
        <div class="location-box">
          <div class="location"> {{ weather.name }}, {{ weather.sys.country}} 
            <img :src="'https://openweathermap.org/img/w/'+ weather.weather[0].icon +'.png'" alt="Weather icon">
          </div>
          <div class="date">{{ dateBuilder() }}</div>
        </div>

        <div class="weather-box animate__animated animate__bounceIn">
          <div class="temp"> 
              {{ Math.round(weather.main.temp)}}°c
  
            <div class="subtemp">
              <span>H:{{ Math.round(weather.main.temp_max) }}°</span>
              <span style="margin-left: 10px;">L:{{ Math.round(weather.main.temp_min)  }}°</span>
            </div>
          </div>
          <div class="weather">{{ weather.weather[0].description }}</div>
        </div>

        <table class="table tabledesign">
            <tbody>
              <tr>
                <th scope="row">Description</th>
                <td>
                  {{weather.weather[0].main}} 
                </td>
              </tr>
              <tr>
                <th scope="row">Feels Like</th>
                <td>{{Math.round(weather.main.feels_like)}}°</td>
              </tr>
              <tr>
                <th scope="row">Pressure</th>
                <td>{{Math.round(weather.main.pressure)}}</td>
              </tr>
              <tr>
                <th scope="row">Humidity</th>
                <td>{{ Math.round(weather.main.humidity) }}</td>
              </tr>
            </tbody>
        </table>
        
      </div>

      <div v-if="error === 'city not found'">
          <!-- Jumbotron -->
            <div class="p-5 text-center text-white">
              <h1 class="mb-3">Hey!!  Location or city was not found</h1>
              <h4 class="mb-3">Try something else.</h4>
            </div>
          <!-- Jumbotron -->
      </div>
    </main>
  </div>
</template>

<script>

export default {
  name: 'app',
    data(){
      return {
        api_key: process.env.VUE_APP_WEATHER_API_PUBLIC,
        url_base: 'https://api.openweathermap.org/data/2.5/',
        query: '',
        error: '',
        weather: {},
      }
    },
    created() {

      const success = (position) => {
          const latitude  = position.coords.latitude;
          const longitude = position.coords.longitude;

          this.getLocationFromGoogleApi(latitude, longitude);
      };

      const error = (err) => {
         return err;
      };

      // This will open permission popup
      navigator.geolocation.getCurrentPosition(success, error);
  },
  methods: {
    getLocationFromGoogleApi(lat,long){
      
      fetch("https://maps.googleapis.com/maps/api/geocode/json?latlng=" + lat + "," + long +  "&key=" + process.env.VUE_APP_WEATHER_API_SECRET , {
        method: "GET",
      })
      .then((response) => {
        return response.json();
      })
      .catch((err) =>{
        console.log(err);
      })
      .then((results) => {
        
        this.query = results.results[6].address_components[0].long_name;
        this.fetchWeather();
      });

    },
    fetchWeather(){
      fetch(`${this.url_base}weather?q=${this.query}&units=metric&APPID=${this.api_key}`)
        .then((res)=> {
          return res.json();
        }).then(this.setResults)
        .catch(()=> {
          console.log('City not found')
        });
    },
    getWeather(e){
      if(e.key == "Enter"){
        this.fetchWeather();
      }
    },
    setResults (results){
      //console.log(results);
      this.error = results.message;
      this.weather = results;
    },
    dateBuilder (){
      let d = new Date();
      let months = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];
      let days = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
      let day = days[d.getDay()];
      let date = d.getDate();
      let month = months[d.getMonth()];
      let year = d.getFullYear();
      return `${day} ${date}, ${month} ${year}`;
    }
  }
}

</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
body {
  font-family: 'montserrat', sans-serif;
}
#app {
  background: #87CEEB;
  background-image: url('./assets/bg.jpg');
  transition: 0.4s;
}
#app.warm {
  background-size: cover;
  background-position: bottom;
}
main {
  min-height: 100vh;
  padding: 25px;
  background-image: linear-gradient(to bottom, rgba(0, 0, 0, 0.25), rgba(0, 0, 0, 0.75));
}
.search-box {
  width: 100%;
  margin-bottom: 30px;
}
.search-box .search-bar {
  display: block;
  width: 100%;
  padding: 15px;
  
  color: #313131;
  font-size: 20px;
  appearance: none;
  border:none;
  outline: none;
  background: none;
  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.5);
  border-radius: 0px 16px 0px 16px;
  transition: 0.4s;
}
.search-box .search-bar:focus {
  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.75);
  border-radius: 16px 0px 16px 0px;
}
.location-box .location {
  color: #FFF;
  font-size: 32px;
  font-weight: 500;
  text-align: center;
  text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
}
.location-box .date {
  color: #FFF;
  font-family: 'Roboto', sans-serif;
  font-size: 20px;
  font-weight: 300;
  text-align: center;
}

.weather-box {
  text-align: center;
}
.weather-box .temp {
  display: inline-block;
  padding: 10px 25px;
  color: #FFF;
  font-size: 102px;
  font-weight: 900;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
  background-color:rgba(255, 255, 255, 0.25);
  border-radius: 16px;
  margin: 30px 0px;
  box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.subtemp{
 font-size: 16px;
 font-family: 'Roboto', sans-serif;
 font-weight: 400;
 color: #FFF;
 box-shadow: 0px 0px black;
 text-shadow: 0px 0px black;
}

.weather-box .weather {
  color: #FFF;
  text-transform: capitalize;
  font-family: 'Roboto', sans-serif;
  font-size: 48px;
  font-weight: 700;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.tabledesign{
  text-align: center;
  color: white;
  width: 50%;
  margin: 0 auto;
}

.tabledesign tr{
  font-family: 'Roboto', sans-serif;
  font-size: 15px;
  font-weight: 500;
}
</style>
