<template>
  <q-page id="app" style="overflow: hidden;" >
      <div class="date">
        {{dateBuilder()}}
      </div>
    <!-- use whole div as a button and router to go to 6 days weather page -->
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
      <!-- 2 buttons, one for text size and other for reading text if someone is having hard time to use app -->
      <div class="btn">
          <q-btn id="btn1" @click="toggleSize()" unelevated rounded color="primary" :label="this.btnName" />
          <q-btn id="btn2" @click="read()" unelevated rounded color="primary" label="Read text" />
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
      btnName : "200% text",
      textForReading: "",
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
    //we get data from api and use this to read msg we prepered in getWeatherData function bellow
    read(){
      var msg = new SpeechSynthesisUtterance();
      msg.text = this.textForReading;
      window.speechSynthesis.speak(msg);
    },
    // this function change size of text and label on button
    toggleSize(){
      if (this.btnName === "200% text"){
        this.btnName = "100% text"
        document.getElementById("btn1").style.fontSize = "40px";
        document.getElementById("btn2").style.fontSize = "40px";
        document.getElementsByClassName("date")[0].style.fontSize = "30px";
        document.getElementsByClassName("temp")[0].style.fontSize = "300px";
        var elements = document.getElementsByClassName("trans");
          for (var i = 0, len = elements.length; i < len; i++) {
            elements[i].style.fontSize = "30px";
        }
        var elements = document.getElementsByClassName("bold");
          for (var i = 0, len = elements.length; i < len; i++) {
            elements[i].style.fontSize = "40px";
        }
      }else{
        this.btnName = "200% text"
        document.getElementById("btn1").style.fontSize = "20px";
        document.getElementById("btn2").style.fontSize = "20px";
        document.getElementsByClassName("date")[0].style.fontSize = "15px";
        document.getElementsByClassName("temp")[0].style.fontSize = "150px";
        var elements = document.getElementsByClassName("trans");
          for (var i = 0, len = elements.length; i < len; i++) {
            elements[i].style.fontSize = "15px";
        }
        var elements = document.getElementsByClassName("bold");
          for (var i = 0, len = elements.length; i < len; i++) {
            elements[i].style.fontSize = "20px";
        }
      }
    },
    //we are getting date in this function
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
    //we get lat and lon and then we can get woeid that we need to get info
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
            self.textForReading = "Today is "+ self.dateBuilder() +". You're in "+ self.allWeaatherData.title+", "+ self.allWeaatherData.parent.title + " ,It is "+Math.round(self.allWeaatherData.consolidated_weather[0].the_temp)+" degrees, Wind speed is "+Math.round(self.allWeaatherData.consolidated_weather[0].wind_speed )+" kilometers per hour, Visibility is "+Math.round(self.allWeaatherData.consolidated_weather[0].visibility*10)/10+" meters, Humidity is "+self.allWeaatherData.consolidated_weather[0].humidity+" %"
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
.btn{
  display: flex;
  justify-content: center;
  align-items: center;
  color: #2B446B;
}

#btn1 {
  padding: 15px;
  margin: 15px;
  width: 40%;
  font-size: 20px;
  
}
#btn2 {
  padding: 15px;
  margin: 15px;
  width: 40%;
  font-size: 20px;
  
}

</style>

