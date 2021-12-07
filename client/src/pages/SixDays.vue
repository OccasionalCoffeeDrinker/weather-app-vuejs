<template>
  <q-page id="app" style="overflow: hidden;" >
      <q-btn flat style="position: absolute; top: 10px; left: 10px;" color="primary" @click="$router.push('/')">
      <q-avatar size="15px" style="margin-right: 4px;">
        <img :src="arrowLeft">
      </q-avatar>
      <span>Home</span>
      </q-btn>
      <!-- here we get date -->
      <div class="date">
        {{dateBuilder()}}
      </div>

      <div>
        <!-- we use for loop here to get data for weather, day, icons and max and min temp -->
            <div style="padding: 20px; box-shadow: rgba(0, 0, 0, 0.16) 0px 1px 4px; border-radius: 7px;" v-if="getDone">
                <div style="margin-bottom: 10px" class="row box" v-for="i in allWeaatherData.consolidated_weather.length" :key="i">
                    <div class="col">
                        {{getDay(i-1)}}
                    </div>
                    <div style="text-align: center" class="col">
                        <q-avatar size="24px" rounded>
                            <img :src="imagesSrc[i-1]">
                        </q-avatar>
                    </div>
                    <div  style="text-align: right" class="col"> 
                        {{Math.round(allWeaatherData.consolidated_weather[i-1].max_temp)}} / {{Math.round(allWeaatherData.consolidated_weather[i-1].min_temp)}}
                    </div>
                </div>
            </div>
        </div>
      <div class="btn">
        <!-- 2 buttons, one for text size and other for reading text if someone is having hard time to use app -->
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
import arrowleft from "../static/arrow-left.svg"

export default({
  name: 'PageIndex',
  data() {
    return {
      getDone: 0,
      cloudSnow: cloudsnow,
      cloudSleet: cloudsleet,
      cloudHail: cloudhail,
      cloudLightning: cloudlightning,
      cloudDrizzle: clouddrizzle,
      cloudSun: cloudsun,
      clouds: clouds1,
      arrowLeft: arrowleft,
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
      allWeaatherData: {title: '', parent: {title: ''},consolidated_weather: [{the_temp: '', visibility: '', wind_speed: '', humidity: '', max_temp: '', min_temp: ''}]},
      imageSrc: "https://cdn.quasar.dev/img/avatar.png",
      imagesSrc: [],
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
        var elements = document.getElementsByClassName("col");
          for (var i = 0, len = elements.length; i < len; i++) {
            elements[i].style.fontSize = "40px";
        }
      }else{
        this.btnName = "200% text"
        document.getElementById("btn1").style.fontSize = "20px";
        document.getElementById("btn2").style.fontSize = "20px";
        document.getElementsByClassName("date")[0].style.fontSize = "15px";
        var elements = document.getElementsByClassName("col");
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
    getDay(a){
        let d = new Date();
        let days = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
        let day = days[(d.getDay() + a)%7];
        return day;

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
            self.textForReading = "Today is "+ self.dateBuilder() +". This is weather for next 6 days. "+ self.getDay(0)+"Max temperature is"+ Math.round(self.allWeaatherData.consolidated_weather[0].max_temp) + "degrees, min temperature is"+Math.round(self.allWeaatherData.consolidated_weather[0].min_temp) +"degrees"
                                  + self.getDay(1)+", Max temperature is "+ Math.round(self.allWeaatherData.consolidated_weather[1].max_temp) + " degrees, min temperature is "+Math.round(self.allWeaatherData.consolidated_weather[1].min_temp) +" degrees"
                                  + self.getDay(2)+", Max temperature is "+ Math.round(self.allWeaatherData.consolidated_weather[2].max_temp) + " degrees, min temperature is "+Math.round(self.allWeaatherData.consolidated_weather[2].min_temp) +" degrees"
                                  + self.getDay(3)+", Max temperature is "+ Math.round(self.allWeaatherData.consolidated_weather[3].max_temp) + " degrees, min temperature is "+Math.round(self.allWeaatherData.consolidated_weather[3].min_temp) +" degrees"
                                  + self.getDay(4)+", Max temperature is "+ Math.round(self.allWeaatherData.consolidated_weather[4].max_temp) + " degrees, min temperature is "+Math.round(self.allWeaatherData.consolidated_weather[4].min_temp) +" degrees"
                                  + self.getDay(5)+", Max temperature is "+ Math.round(self.allWeaatherData.consolidated_weather[5].max_temp) + " degrees, min temperature is "+Math.round(self.allWeaatherData.consolidated_weather[5].min_temp) +" degrees"
            self.imagesSrc = [];
            self.allWeaatherData.consolidated_weather.forEach(entry => {
                // self.imagesSrc.push(entry.weather_state_abbr)
                switch (entry.weather_state_abbr) {
                    case 'sn':
                        self.imagesSrc.push(self.cloudSnow);
                        break;
                    case 'sl':
                        self.imagesSrc.push(self.cloudSleet);
                        
                        break;
                    case 'h':
                        self.imagesSrc.push(self.cloudHail);
                        
                        break;
                    case 't':
                        self.imagesSrc.push(self.cloudLightning);
                        
                        break;
                    case 'hr':
                        self.imagesSrc.push(self.cloudRainHeavy);
                        
                        break;
                    case 'lr':
                        self.imagesSrc.push(self.cloudDrizzle);
                        
                        break;
                    case 's':
                        self.imagesSrc.push(self.cloudDrizzle);
                        
                        break;
                    case 'hc':
                        self.imagesSrc.push(self.clouds);
                        
                        break;
                    case 'lc':
                        self.imagesSrc.push(self.cloudSun);
                        
                        break;
                    case 'c':
                        self.imagesSrc.push(self.sun);
                        
                        break;
                    
                    default:
                        console.log("jok")
                        break;
                    }
            });
                        console.log("slikeee    ");
                        console.log(self.imagesSrc);

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
            self.getDone += 1;
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
  margin: auto;
  font-size: 17px;
  padding: 0px 20px 20px 20px;
}
#whiteBox {
  background-color: #ffffff;
  width: 80%;
}

.date {
  width:  fit-content;
  padding: 15px;
  margin: auto;
}

.whiteBox {
  width: 90%;
  border: 1px solid black;
  box-shadow: 3px #56585a;
  border-radius: 5px;
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
  font-weight: 400;
  font-size: 20px;
}
.trans {
  font-size: 15px;
  color: lightgray;
}
.box{
    padding: 10px;
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