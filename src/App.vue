<template>
  <div class="content">
    <button @click="getCurrentLocation">GET LOCATION</button>
    <p>CurrentLocation</p>
    <ul>
      <li>lat: {{ lat }}</li>
      <li>lng:{{ lng }}</li>
    </ul>
    <button @click="setWeatherReports">GET WEATHER</button>
    <p>WeatherReports</p>
    <ul>
      <li v-for="(item, index) in weatherReports" :key="item">{{ index }}:{{ item }}</li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      lat: 0,
      lng: 0,
      weatherReports: {}
    }
  },
  name: 'App',
  methods: {
    getCurrentLocation() {
      if (navigator.geolocation) {
        const options = {
          enableHighAccuracy: false,
          timeout: 5000,
          maximumAge: 0
        }

        navigator.geolocation.getCurrentPosition(this.success, this.error, options)
      } else {
        console.error("Geolocation not valid")
      }
    },
    success(pos) {
      this.lat = pos.coords.latitude
      this.lng = pos.coords.longitude
    },
    error(err) {
      console.error(`ERROR(${err.code}):${err.message}`)
    },
    callWeatherAPI: async function () {
      if (this.lat && this.lng) {
        const url = `https://api.open-meteo.com/v1/forecast?latitude=${this.lat}&longitude=${this.lng}&daily=weathercode&timezone=auto`
        const res = await (await fetch(url)).json()
        return res.daily.weathercode
      } else {
        console.error("Please enter lat & lng")
        return "err"
      }
    },
    setWeatherReports: async function () {
      const res = await this.callWeatherAPI()
      if (res != "err") {
        let weatherReports = {
          day1: undefined,
          day2: undefined,
          day3: undefined,
          day4: undefined,
          day5: undefined,
          day6: undefined,
          day7: undefined
        }
        for (let i = 0; i < res.length; i++) {
          const weathercode = res[i];
          switch (weathercode) {
            case 0:
              weatherReports[`day${i + 1}`] = "Clear sky"
              break
            case 1: case 2: case 3:
              weatherReports[`day${i + 1}`] = "Mainly clear, partly cloudy, and overcast"
              break
            case 45: case 48:
              weatherReports[`day${i + 1}`] = "Fog and depositing rime fog"
              break
            case 51: case 53: case 55:
              weatherReports[`day${i + 1}`] = "Drizzle: Light, moderate, and dense intensity"
              break
            case 56: case 57:
              weatherReports[`day${i + 1}`] = "Freezing Drizzle: Light and dense intensity"
              break
            case 61: case 63: case 65:
              weatherReports[`day${i + 1}`] = "Rain: Slight, moderate and heavy intensity"
              break
            case 66: case 67:
              weatherReports[`day${i + 1}`] = "Freezing Rain: Light and heavy intensity"
              break
            case 71: case 73: case 75:
              weatherReports[`day${i + 1}`] = "Snow fall: Slight, moderate, and heavy intensity"
              break
            case 77:
              weatherReports[`day${i + 1}`] = "Snow grains"
              break
            case 80: case 81: case 82:
              weatherReports[`day${i + 1}`] = "Rain showers: Slight, moderate, and violent"
              break
            case 85: case 86:
              weatherReports[`day${i + 1}`] = "Snow showers slight and heavy"
              break
            case 95:
              weatherReports[`day${i + 1}`] = "Thunderstorm: Slight or moderate"
              break
            case 96: case 99:
              weatherReports[`day${i + 1}`] = "Thunderstorm with slight and heavy hail"
              break

            default:
              break;
          }
        }
        this.weatherReports = weatherReports
      }
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
}

li {
  list-style: none;
  margin-top: 7px;
}

.content {
  width: 80%;
  margin: 0 auto;
}
</style>
