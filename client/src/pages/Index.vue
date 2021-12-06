<template>
  <q-page id="app" style="overflow: hidden;" >
      <div class="date">
        {{dateBuilder()}}
      </div>
    
      <div class="whiteBox" @click="$router.push('/sixdays')">
        <div class="row">
          <div class="col">
            <span class="bold">
              {{allWeaatherData.title}}
            </span>
            <br>
            <span class="trans">
              {{allWeaatherData.parent.title}}
            </span>
          </div>
          <div class="col-2">
            <q-avatar rounded>
              <img :src="imageSrc">
            </q-avatar>
          </div>

        </div>
        <br>
        <div style="display: flex;">
          <div class="temp">{{Math.round(allWeaatherData.consolidated_weather[0].the_temp)}}<sup>°C</sup> </div>
        </div>
        <br>
        <div class="row" style="text-align: center;">
          <div class="col">
            <div class="column" style="height: 150px">
              <div class="col">
                <q-avatar rounded>
                  <img style="color: #156DF3" src="../static/wind.svg">
                </q-avatar>
              </div>
              <div class="col">
                <span class="bold">
                  {{Math.round(allWeaatherData.consolidated_weather[0].wind_speed)}}km/h
                </span>
                <br>
                <span class="trans">
                  Wind
                </span>
              </div>
            </div>
          </div>
          <div class="col">
            <div class="column" style="height: 150px">
              <div class="col">
                <q-avatar rounded>
                  <img src="../static/eye.svg">
                </q-avatar>
              </div>
              <div class="col">
                <span class="bold">
                  {{Math.round(allWeaatherData.consolidated_weather[0].visibility *10) / 10}}m
                </span>
                <br>
                <span class="trans">
                  Visibility
                </span>
              </div>
            </div>
          </div>
          <div class="col">
            <div class="column" style="height: 150px">
              <div class="col">
                <q-avatar rounded>
                  <img src="../static/droplet.svg">
                </q-avatar>
              </div>
              <div class="col">
                <span class="bold">
                  {{Math.round(allWeaatherData.consolidated_weather[0].humidity)}}%
                </span>
                <br>
                <span class="trans">
                  Humidity
                </span>
              </div>
            </div>
          </div>
        </div>
      </div>


    
  </q-page>
</template>

<script>
// import { Todo, Meta } from 'components/models';
// import ExampleComponent from 'components/CompositionComponent.vue';
// import { defineComponent, ref } from 'vue';

import cloudsnow from "../static/cloud-snow.svg"
import cloudsleet from "../static/cloud-sleet.svg"
import cloudhail from "../static/cloud-hail.svg"
import cloudlightning from "../static/cloud-lightning.svg"
import clouddrizzle from "../static/cloud-drizzle.svg"
import cloudsun from "../static/cloud-sun.svg"
import clouds1 from "../static/clouds.svg"
import sun1 from "../static/sun.svg"
import cloudrainheavy from "../static/cloud-rain-heavy.svg"

export default({
  name: 'PageIndex',
  data() {
    return {
      cloudSnow: cloudsnow,
      cloudSleet: cloudsleet,
      cloudHail: cloudhail,
      cloudLightning: cloudlightning,
      cloudDrizzle: clouddrizzle,
      cloudSun: cloudsun,
      clouds: clouds1,
      sun: sun1,
      cloudRainHeavy: cloudrainheavy,
      api_key: '',
      url: 'http://localhost:8080/https://www.metaweather.com/api/',
      query: '',
      weather: {},
      oReq: '',
      location: {coords: {latitude: null, longitude: null}},
      gettingLocation: false,
      errorStr:null,
      allWeaatherData: {title: '', parent: {title: ''},consolidated_weather: [{the_temp: '', visibility: '', wind_speed: '', humidity: ''}]},
      imageSrc: sun1,
    };
  },
  created() {
      //do we support geolocation
      if(!("geolocation" in navigator)) {
      this.errorStr = 'Geolocation is not available.';
      return;
      }

      this.gettingLocation = true;
      // get position
      navigator.geolocation.getCurrentPosition(pos => {
      this.gettingLocation = false;
      this.location = pos;
      this.getWoeData();
      console.log(pos);
      }, err => {
      this.gettingLocation = false;
      this.errorStr = err.message;
      })
  },
  methods: {
    dateBuilder(){
      let d = new Date();
      let months = ["Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"];
      let days = ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"];

      let day = days[d.getDay()];
      let date = d.getDate();
      let month = months[d.getMonth()];
      let year = d.getFullYear();

      return `${day}, ${month} ${date} ${year}`;
    },
    getWoeData() {       
        let linkStr = `http://localhost:8080/https://www.metaweather.com/api/location/search/?lattlong=${this.location.coords.latitude},${this.location.coords.longitude}`
        debugger;
        let self = this;
        this.$axios
        .get(linkStr,{
            headers: {
            "Content-Type": "application/json",
            }
        })
        .then(response => {
            debugger
            console.log("Odgovor prvi")
            console.log(response.data)
            self.getWeatherData(response.data[0].woeid)
            debugger

        })
        .catch(e => {
            //this.errors.push(e);
            console.log("Greska: " + e);
        });
    },
    getWeatherData(woeid) {       
        let linkStr = `http://localhost:8080/https://www.metaweather.com/api/location/${woeid}/`
        debugger;
        let self = this;
        this.$axios
        .get(linkStr,{
            headers: {
            "Content-Type": "application/json",
            }
        })
        .then(response => {
            debugger
            console.log("Odgovor drugi")
            console.log(response.data);
            self.allWeaatherData = response.data;

            switch (response.data.consolidated_weather[0].weather_state_abbr) {
              case 'sn':
                self.imageSrc = self.cloudSnow;
                break;
              case 'sl':
                self.imageSrc = self.cloudSleet;
                
                break;
              case 'h':
                self.imageSrc = self.cloudHail;
                
                break;
              case 't':
                self.imageSrc = self.cloudLightning;
                
                break;
              case 'hr':
                self.imageSrc = self.cloudRainHeavy;
                
                break;
              case 'lr':
                self.imageSrc = self.cloudDrizzle;
                
                break;
              case 's':
                self.imageSrc = self.cloudDrizzle;
                
                break;
              case 'hc':
                self.imageSrc = self.clouds;
                
                break;
              case 'lc':
                self.imageSrc = self.cloudSun;
                
                break;
              case 'c':
                self.imageSrc = self.sun;
                
                break;
            
              default:
                console.log("jok")
                break;
            }
            console.log(self.imageSrc)
            debugger

        })
        .catch(e => {
            //this.errors.push(e);
            console.log("Greska: " + e);
        });
    },
  },

});
</script>
<style scoped>

#app {
  background-color: #fbfdff;
  color: #2B446B;
}
#whiteBox {
  background-color: #ffffff;
  width: 80%;
}

.date {
  width:  fit-content;
  font-size: 20px;
  padding: 15px;
  margin: auto;
}

.whiteBox {
  width: 90%;
  box-shadow: rgba(0, 0, 0, 0.16) 0px 1px 4px;
  border-radius: 7px;
  margin: auto;
  padding: 15px;
}

.temp{
  font-size: 150px;
  
  text-align: center;
  display: table-cell;
  margin: auto;
}
sup {
    font-size: 30px;
    position: relative;
    top: -40px;
}
.bold {
  font-weight: bold;
  font-size: 20px;
}
.trans {
  font-size: 15px;
  color: lightgray;
}

</style>

