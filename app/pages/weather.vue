<script setup lang="ts">
import { ref, computed } from 'vue'

const city = ref('San Fernando')
const searchCity = ref('')
const loading = ref(false)
const errorMessage = ref('')

const weather = ref<any>(null)
const forecast = ref<any>(null)

const weatherCodes: Record<number, string> = {
  0: 'Clear sky',
  1: 'Mainly clear',
  2: 'Partly cloudy',
  3: 'Overcast',
  45: 'Foggy',
  48: 'Foggy',
  51: 'Light drizzle',
  53: 'Drizzle',
  55: 'Heavy drizzle',
  61: 'Light rain',
  63: 'Rainy',
  65: 'Heavy rain',
  71: 'Light snow',
  80: 'Rain showers',
  81: 'Rain showers',
  82: 'Heavy showers',
  95: 'Thunderstorm',
  96: 'Thunderstorm',
  99: 'Thunderstorm'
}

const weatherIcon = (code: number, isDay = true) => {
  if (code === 0) return isDay ? '☀️' : '🌙'
  if (code === 1 || code === 2) return isDay ? '🌤️' : '☁️'
  if (code === 3) return '☁️'
  if (code === 45 || code === 48) return '🌫️'
  if (code >= 51 && code <= 67) return '🌧️'
  if (code >= 71 && code <= 77) return '❄️'
  if (code >= 80 && code <= 82) return '🌦️'
  if (code >= 95) return '⛈️'
  return '🌤️'
}

const condition = computed(() => {
  if (!weather.value) return 'Loading weather...'
  return weatherCodes[weather.value.weather_code] || 'Cloudy'
})

const today = computed(() =>
  new Intl.DateTimeFormat('en-US', {
    weekday: 'long',
    month: 'long',
    day: 'numeric'
  }).format(new Date())
)

const forecastDays = computed(() => {
  if (!forecast.value) return []

  return forecast.value.daily.time.map((date: string, index: number) => ({
    date,
    day: new Intl.DateTimeFormat('en-US', {
      weekday: 'short'
    }).format(new Date(`${date}T12:00:00`)),
    icon: weatherIcon(forecast.value.daily.weather_code[index]),
    high: Math.round(forecast.value.daily.temperature_2m_max[index]),
    low: Math.round(forecast.value.daily.temperature_2m_min[index])
  }))
})

async function getWeather(location = city.value) {
  if (!location.trim()) return

  loading.value = true
  errorMessage.value = ''

  try {
    const geo = await $fetch<any>(
      'https://geocoding-api.open-meteo.com/v1/search',
      {
        query: {
          name: location.trim(),
          count: 1,
          language: 'en',
          format: 'json'
        }
      }
    )

    if (!geo.results?.length) {
      throw new Error('City not found. Try another location.')
    }

    const place = geo.results[0]
    city.value = `${place.name}${place.country ? ', ' + place.country : ''}`

    const data = await $fetch<any>(
      'https://api.open-meteo.com/v1/forecast',
      {
        query: {
          latitude: place.latitude,
          longitude: place.longitude,
          current: [
            'temperature_2m',
            'relative_humidity_2m',
            'apparent_temperature',
            'is_day',
            'precipitation',
            'weather_code',
            'wind_speed_10m'
          ].join(','),
          daily: [
            'weather_code',
            'temperature_2m_max',
            'temperature_2m_min',
            'precipitation_probability_max'
          ].join(','),
          timezone: 'auto',
          forecast_days: 7
        }
      }
    )

    weather.value = data.current
    forecast.value = data
    searchCity.value = ''
  } catch (error: any) {
    errorMessage.value =
      error?.data?.message ||
      error?.message ||
      'Unable to load weather. Please try again.'
  } finally {
    loading.value = false
  }
}

function searchWeather() {
  getWeather(searchCity.value)
}

function useMyLocation() {
  if (!navigator.geolocation) {
    errorMessage.value = 'Geolocation is not supported by your browser.'
    return
  }

  loading.value = true
  errorMessage.value = ''

  navigator.geolocation.getCurrentPosition(
    async (position) => {
      try {
        const data = await $fetch<any>(
          'https://api.open-meteo.com/v1/forecast',
          {
            query: {
              latitude: position.coords.latitude,
              longitude: position.coords.longitude,
              current:
                'temperature_2m,relative_humidity_2m,apparent_temperature,is_day,precipitation,weather_code,wind_speed_10m',
              daily:
                'weather_code,temperature_2m_max,temperature_2m_min,precipitation_probability_max',
              timezone: 'auto',
              forecast_days: 7
            }
          }
        )

        weather.value = data.current
        forecast.value = data

        const geo = await $fetch<any>(
          'https://geocoding-api.open-meteo.com/v1/search',
          {
            query: {
              name: 'San Fernando',
              count: 1,
              language: 'en'
            }
          }
        )

        city.value = `Your location (${position.coords.latitude.toFixed(2)}, ${position.coords.longitude.toFixed(2)})`
      } catch {
        errorMessage.value = 'Unable to load your local weather.'
      } finally {
        loading.value = false
      }
    },
    () => {
      errorMessage.value = 'Location access denied. Please allow location access.'
      loading.value = false
    }
  )
}

getWeather('San Fernando')
</script>

<template>
  <main class="weather-page">
    <div class="weather-container">
      <header class="topbar">
        <div class="brand">
          <div class="brand-icon">☀</div>
          <span>Weatherly</span>
        </div>

        <button class="location-button" @click="useMyLocation">
          <span>◎</span> My Location
        </button>
      </header>

      <section class="search-area">
        <p class="eyebrow">WEATHER FORECAST</p>
        <h1>Discover the weather.</h1>
        <p class="subtitle">Stay ahead of the weather, wherever you go.</p>

        <form class="search-box" @submit.prevent="searchWeather">
          <span class="search-icon">⌕</span>
          <input
            v-model="searchCity"
            placeholder="Search city..."
            aria-label="Search city"
          />
          <button type="submit" :disabled="loading">
            {{ loading ? 'Loading...' : 'Search' }}
          </button>
        </form>

        <p v-if="errorMessage" class="error-message">
          {{ errorMessage }}
        </p>
      </section>

      <section v-if="weather" class="current-card">
        <div class="current-left">
          <div class="current-location">
            <span>📍</span> {{ city }}
          </div>
          <p class="current-date">{{ today }}</p>

          <div class="temperature">
            {{ Math.round(weather.temperature_2m) }}<span>°</span>
          </div>

          <h2>{{ condition }}</h2>
          <p class="feels-like">
            Feels like {{ Math.round(weather.apparent_temperature) }}°C
          </p>
        </div>

        <div class="current-right">
          <div class="weather-art">
            {{ weatherIcon(weather.weather_code, weather.is_day === 1) }}
          </div>
          <div class="art-caption">Today's weather</div>
        </div>
      </section>

      <section v-if="weather" class="details-grid">
        <article class="detail-card">
          <div class="detail-icon humidity">💧</div>
          <div>
            <p>Humidity</p>
            <h3>{{ weather.relative_humidity_2m }}<span>%</span></h3>
          </div>
        </article>

        <article class="detail-card">
          <div class="detail-icon wind">💨</div>
          <div>
            <p>Wind speed</p>
            <h3>{{ weather.wind_speed_10m }} <span>km/h</span></h3>
          </div>
        </article>

        <article class="detail-card">
          <div class="detail-icon rain">🌧️</div>
          <div>
            <p>Precipitation</p>
            <h3>{{ weather.precipitation }} <span>mm</span></h3>
          </div>
        </article>

        <article class="detail-card">
          <div class="detail-icon feels">🌡️</div>
          <div>
            <p>Feels like</p>
            <h3>{{ Math.round(weather.apparent_temperature) }}<span>°C</span></h3>
          </div>
        </article>
      </section>

      <section v-if="forecastDays.length" class="forecast-section">
        <div class="section-heading">
          <div>
            <p class="eyebrow">PLAN YOUR WEEK</p>
            <h2>7-day forecast</h2>
          </div>
          <span class="week-label">Next 7 days</span>
        </div>

        <div class="forecast-list">
          <article
            v-for="(day, index) in forecastDays"
            :key="day.date"
            class="forecast-day"
            :class="{ 'today-row': index === 0 }"
          >
            <span class="forecast-name">
              {{ index === 0 ? 'Today' : day.day }}
            </span>
            <span class="forecast-icon">{{ day.icon }}</span>
            <div class="forecast-condition">
              {{ weatherCodes[forecast.daily.weather_code[index]] || 'Cloudy' }}
            </div>
            <div class="forecast-temperatures">
              <strong>{{ day.high }}°</strong>
              <span>{{ day.low }}°</span>
            </div>
          </article>
        </div>
      </section>

      <footer class="footer">
        <span>☀ Weatherly</span>
        <span>Weather data by Open-Meteo</span>
      </footer>
    </div>
  </main>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;600;700&family=Plus+Jakarta+Sans:wght@400;500;600;700;800&display=swap');

:root {
  font-family: 'DM Sans', sans-serif;
  color: #172d4b;
  background: #f4f7fc;
  font-synthesis: none;
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
}

button,
input {
  font: inherit;
}

button {
  cursor: pointer;
}

.weather-page {
  min-height: 100vh;
  padding: 24px;
  background:
    radial-gradient(ellipse at 85% 0%, lightblue 0%, transparent 35%),
    lightblue;
}

.weather-container {
  width: min(100%, 1000px);
  margin: 0 auto;
}

.topbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 6px 0 36px;
}

.brand {
  display: flex;
  align-items: center;
  gap: 10px;
  font-family: 'Plus Jakarta Sans', sans-serif;
  font-size: 22px;
  font-weight: 800;
  letter-spacing: -0.8px;
}

.brand-icon {
  display: grid;
  place-items: center;
  width: 38px;
  height: 38px;
  border-radius: 13px;
  background: #e9efff;
  color: #5278ec;
  font-size: 27px;
}

.location-button {
  padding: 11px 16px;
  border: 1px solid lightblue;
  border-radius: 12px;
  background: lightblue;
  color: #314967;
  font-size: 13px;
  font-weight: 600;
}

.location-button span {
  margin-right: 6px;
  color: #5479ec;
}

.search-area {
  margin-bottom: 30px;
}

.eyebrow {
  margin: 0 0 10px;
  color: #6681b7;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 2px;
}

.search-area h1 {
  margin: 0;
  font-family: 'Plus Jakarta Sans', sans-serif;
  font-size: clamp(29px, 5vw, 43px);
  font-weight: 800;
  letter-spacing: -1.8px;
}

.subtitle {
  margin: 10px 0 23px;
  color: #7a8ba5;
  font-size: 14px;
}

.search-box {
  display: flex;
  align-items: center;
  gap: 12px;
  width: min(100%, 550px);
  padding: 6px;
  border: 1px solid lightblue;
  border-radius: 15px;
  background: white;
  box-shadow: 0 8px 25px #243d6208;
}

.search-icon {
  padding-left: 12px;
  color: #8c9bb2;
  font-size: 26px;
}

.search-box input {
  width: 100%;
  min-width: 0;
  border: 0;
  outline: none;
  background: transparent;
  color: #172d4b;
  font-size: 14px;
}

.search-box button {
  flex-shrink: 0;
  padding: 12px 20px;
  border: 0;
  border-radius: 10px;
  background: #5378ed;
  color: lightblue;
  font-size: 13px;
  font-weight: 700;
}

.search-box button:disabled {
  opacity: 0.6;
}

.error-message {
  color: #c13d49;
  font-size: 13px;
}

.current-card {
  position: relative;
  display: flex;
  justify-content: space-between;
  min-height: 270px;
  padding: 32px 38px;
  overflow: hidden;
  border-radius: 25px;
  background: linear-gradient(115deg, #5478ee, #7595f5 65%, #8aa7fb);
  color: lightblue;
  box-shadow: 0 18px 40px #5478ee25;
}

.current-card::after {
  position: absolute;
  right: 100px;
  bottom: -155px;
  width: 330px;
  height: 330px;
  border: 1px solid #ffffff20;
  border-radius: 50%;
  content: '';
}

.current-left,
.current-right {
  position: relative;
  z-index: 1;
}

.current-location {
  font-size: 15px;
  font-weight: 700;
}

.current-date {
  margin: 8px 0 0;
  color: lightblue;
  font-size: 13px;
}

.temperature {
  margin-top: 20px;
  font-family: 'Plus Jakarta Sans', sans-serif;
  font-size: 76px;
  font-weight: 700;
  letter-spacing: -5px;
  line-height: 1;
}

.temperature span {
  font-size: 49px;
  vertical-align: top;
}

.current-left h2 {
  margin: 10px 0 0;
  font-size: 17px;
  font-weight: 600;
}

.feels-like {
  margin: 6px 0 0;
  color: lightblue;
  font-size: 12px;
}

.current-right {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding-right: 24px;
}

.weather-art {
  font-size: clamp(65px, 10vw, 105px);
  filter: drop-shadow(0 12px 15px #294da72b);
}

.art-caption {
  margin-top: 8px;
  color: lightblue;
  font-size: 12px;
}

.details-grid {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 15px;
  margin-top: 20px;
}

.detail-card {
  display: flex;
  align-items: center;
  gap: 13px;
  min-width: 0;
  padding: 19px 15px;
  border: 1px solid lightblue;
  border-radius: 17px;
  background: lightblue;
}

.detail-icon {
  display: grid;
  flex-shrink: 0;
  place-items: center;
  width: 40px;
  height: 40px;
  border-radius: 13px;
  font-size: 19px;
}

.humidity {
  background: lightblue;
}

.wind {
  background: lightblue;
}

.rain {
  background: lightblue;
}

.feels {
  background: lightblue;
}

.detail-card p {
  margin: 0 0 7px;
  color: #8190a7;
  font-size: 11px;
}

.detail-card h3 {
  margin: 0;
  font-family: 'Plus Jakarta Sans', sans-serif;
  font-size: 19px;
  font-weight: 800;
  white-space: nowrap;
}

.detail-card h3 span {
  font-size: 11px;
  font-weight: 600;
}

.forecast-section {
  margin-top: 35px;
}

.section-heading {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 17px;
}

.section-heading .eyebrow {
  margin-bottom: 6px;
}

.section-heading h2 {
  margin: 0;
  font-family: 'Plus Jakarta Sans', sans-serif;
  font-size: 23px;
  font-weight: 800;
  letter-spacing: -0.7px;
}

.week-label {
  padding: 8px 12px;
  border-radius: 10px;
  background: lightblue;
  color: #5375d9;
  font-size: 11px;
  font-weight: 700;
}

.forecast-list {
  overflow: hidden;
  border: 1px solid lightblue;
  border-radius: 18px;
  background: lightblue;
}

.forecast-day {
  display: grid;
  grid-template-columns: 1fr 50px 1.2fr 80px;
  align-items: center;
  gap: 12px;
  min-height: 66px;
  padding: 10px 22px;
  border-bottom: 1px solid lightblue;
}

.forecast-day:last-child {
  border-bottom: 0;
}

.forecast-name {
  font-size: 13px;
  font-weight: 700;
}

.forecast-icon {
  font-size: 25px;
  text-align: center;
}

.forecast-condition {
  color: #8190a7;
  font-size: 12px;
}

.forecast-temperatures {
  display: flex;
  justify-content: flex-end;
  gap: 14px;
  font-size: 13px;
}

.forecast-temperatures span {
  color: #94a0b4;
}

.today-row {
  background: lightblue;
}

.footer {
  display: flex;
  justify-content: space-between;
  gap: 12px;
  margin: 30px 0 10px;
  color: #94a0b4;
  font-size: 11px;
}

.footer span:first-child {
  color: #6178bd;
  font-weight: 700;
}

@media (max-width: 760px) {
  .weather-page {
    padding: 16px;
  }

  .topbar {
    padding-bottom: 30px;
  }

  .current-card {
    min-height: 235px;
    padding: 26px 22px;
    border-radius: 21px;
  }

  .temperature {
    font-size: 65px;
  }

  .temperature span {
    font-size: 40px;
  }

  .current-right {
    padding-right: 0;
  }

  .details-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 11px;
  }

  .detail-card {
    padding: 15px 12px;
  }

  .forecast-day {
    grid-template-columns: 0.8fr 35px 1fr 65px;
    gap: 7px;
    padding: 10px 12px;
  }
}

@media (max-width: 390px) {
  .brand {
    font-size: 19px;
  }

  .location-button {
    padding: 10px;
    font-size: 11px;
  }

  .current-card {
    padding: 23px 16px;
  }

  .current-right {
    max-width: 95px;
  }

  .weather-art {
    font-size: 65px;
  }

  .forecast-condition {
    font-size: 10px;
  }

  .forecast-day {
    grid-template-columns: 0.8fr 30px 1fr 58px;
  }

  .forecast-temperatures {
    gap: 8px;
    font-size: 12px;
  }
}
</style>


