<template>
  <div class = "all">
    <input
      type="text"
      v-model="searchLocation"
      @input="fetchLocation"
      placeholder="Search needed location"
    />
    <ul>
      <li v-for="location in locations" :key="location.id" @click="selectLocation(location)">
        {{ `${location.name}, ${location.country}` }}
      </li>
    </ul>

    <div v-if="selectedLocation">
      <h2>{{ selectedLocation.name }}</h2>
      <p>
        Current Weather: {{ currentWeather }} {{ dailyTemperature.hourly_units.temperature_2m }}
      </p>
      <ul>
        <li v-for="(forecast, index) in dailyTemperature.hourly.time" :key="index">
          {{ forecast }} - {{ dailyTemperature.hourly.temperature_2m[index] }}
          {{ dailyTemperature.hourly_units.temperature_2m }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      searchLocation: '',
      locations: [],
      selectedLocation: null,
      currentWeather: '',
      dailyTemperature: {}
    }
  },
  methods: {
    async fetchLocation() {
      if (this.searchLocation.length > 2) {
        try {
          const response = await fetch(
            `https://geocoding-api.open-meteo.com/v1/search?name=${this.searchLocation}&count=10&language=en&format=json`
          )
          if (!response.ok) {
            throw new Error('Network response was not ok')
          }
          const data = await response.json()
          this.locations = data.results
        } catch (error) {
          console.error('Error fetching data:', error)
        }
      }
    },
    async fetchWeather(latitude, longitude) {
      try {
        const currentDate = new Date()
        let year = currentDate.getFullYear()
        let hour = currentDate.getHours()
        let month = currentDate.getMonth() + 1
        month = month < 10 ? '0' + month : month
        let day = currentDate.getDate()
        let nextday = day + 1
        day = day < 10 ? '0' + day : day
        nextday = nextday < 10 ? '0' + nextday : nextday
        const start_date = `${year}-${month}-${day}`
        const end_date = `${year}-${month}-${nextday}`
        const response = await fetch(
          `https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&hourly=temperature_2m&start_date=${start_date}&end_date=${end_date}`
        )
        if (!response.ok) {
          throw new Error('Network response was not ok')
        }
        const data = await response.json()
        this.dailyTemperature = data
        this.currentWeather = data.hourly.temperature_2m[hour]
      } catch (error) {
        console.error('Error fetching data:', error)
      }
    },
    selectLocation(location) {
      this.selectedLocation = location
      this.fetchWeather(location.latitude, location.longitude)
    }
  }
}
</script>

<style>
div .all {
  position:absolute;
  left:10px;
  top:10px;
  color:blue;
}
</style>
