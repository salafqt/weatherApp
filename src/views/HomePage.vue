<template>
  <ion-page>
    <ion-header translucent>
      <ion-toolbar>
        <ion-title>Weather</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content fullscreen :style="bgStyle">
      <div class="glass-container">
        <div class="weather-header glass-effect">
          <h1>{{ city }}</h1>
          <div class="current-temp">{{ currentTemp.toFixed(0) }}°</div>
          <h2>{{ currentCondition }}</h2>
          <div class="weather-details">
            <div class="detail-item">
              <ion-icon :icon="waterOutline"></ion-icon>
              <span>{{ humidity }}%</span>
            </div>
            <div class="detail-item">
              <ion-icon :icon="speedometerOutline"></ion-icon>
              <span>{{ windSpeed }} km/h</span>
            </div>
          </div>
        </div>

        <div class="forecast-section glass-effect">
          <h3>Hourly Forecast for {{ selectedDate }}</h3>
          <div class="forecast-navigation">
            <ion-button @click="prevDay" :disabled="currentDayIndex === 0">
              &lt; Previous Day
            </ion-button>
            <ion-button
              @click="nextDay"
              :disabled="currentDayIndex === forecast.length - 1"
            >
              Next Day &gt;
            </ion-button>
          </div>
          <div class="forecast-grid">
            <WeatherCard
              v-for="(item, i) in hourlyForecast"
              :key="i"
              :time="item.time"
              :temperature="item.temperature"
              :code="item.code"
              :date="item.date"
            />
          </div>
        </div>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import WeatherCard from "@/components/WeatherCard.vue";
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonIcon,
  IonButton,
} from "@ionic/vue";
import { waterOutline, speedometerOutline } from "ionicons/icons";

const city = "Jakarta";
const forecast = ref([]);
const currentTemp = ref(0);
const currentCondition = ref("Loading...");
const currentCode = ref(0);
const humidity = ref(0);
const windSpeed = ref(0);
const currentDayIndex = ref(0); // Track the current day index

const selectedDate = computed(() => {
  if (forecast.value.length > 0) {
    return new Date(
      forecast.value[currentDayIndex.value].date
    ).toLocaleDateString("en-US", {
      weekday: "long",
      month: "long",
      day: "numeric",
    });
  }
  return "";
});

const hourlyForecast = computed(() => {
  if (forecast.value.length > 0) {
    return forecast.value[currentDayIndex.value].hourly; // Get hourly data for the selected day
  }
  return [];
});

onMounted(async () => {
  const res = await fetch(
    "https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&daily=temperature_2m_max,temperature_2m_min,weathercode&hourly=temperature_2m,weathercode,relativehumidity_2m,windspeed_10m&current_weather=true"
  );
  const data = await res.json();

  const times = data.hourly.time;
  const temps = data.hourly.temperature_2m;
  const codes = data.hourly.weathercode;
  const humidityData = data.hourly.relativehumidity_2m;
  const windSpeedData = data.hourly.windspeed_10m;

  // Update current weather
  currentTemp.value = data.current_weather.temperature;
  currentCode.value = data.current_weather.weathercode;
  currentCondition.value = mapWeatherCode(data.current_weather.weathercode);
  humidity.value = humidityData[0];
  windSpeed.value = windSpeedData[0];

  // Group hourly data by day
  const dailyForecast = data.daily.time.map((date, index) => {
    const hourlyData = times
      .slice(index * 24, (index + 1) * 24) // Get 24 hours for each day
      .map((t, i) => ({
        time: new Date(t).toLocaleTimeString("id-ID", {
          hour: "2-digit",
          minute: "2-digit",
        }),
        temperature: temps[index * 24 + i],
        code: codes[index * 24 + i],
        date: t,
      }));
    return {
      date: date,
      hourly: hourlyData,
    };
  });

  forecast.value = dailyForecast; // Store the daily forecast
});

function prevDay() {
  if (currentDayIndex.value > 0) {
    currentDayIndex.value--;
  }
}

function nextDay() {
  if (currentDayIndex.value < forecast.value.length - 1) {
    currentDayIndex.value++;
  }
}

function mapWeatherCode(code) {
  if (code === 0) return "Clear";
  if (code >= 1 && code <= 3) return "Partly Cloudy";
  if (code >= 45 && code <= 48) return "Foggy";
  if (code >= 51 && code <= 67) return "Rain";
  if (code >= 71 && code <= 77) return "Snow";
  if (code >= 80 && code <= 82) return "Showers";
  if (code >= 95) return "Thunderstorm";
  return "Unknown";
}

const bgStyle = computed(() => {
  const c = currentCode.value;
  const time = new Date().getHours();
  const isDaytime = time > 6 && time < 18;

  const baseGradients = {
    clear: isDaytime
      ? "linear-gradient(135deg, #72EDF2 10%, #5151E5 100%)"
      : "linear-gradient(135deg, #090947 10%, #5A585A 100%)",
    cloudy: "linear-gradient(135deg, #6B73FF 10%, #000DFF 100%)",
    foggy: "linear-gradient(135deg, #757F9A 10%, #D7DDE8 100%)",
    rainy: "linear-gradient(135deg, #000046 10%, #1CB5E0 100%)",
    storm: "linear-gradient(135deg, #000000 10%, #434343 100%)",
  };

  if (c === 0) return `background: ${baseGradients.clear};`;
  if (c >= 1 && c <= 3) return `background: ${baseGradients.cloudy};`;
  if (c >= 45 && c <= 48) return `background: ${baseGradients.foggy};`;
  if (c >= 51 && c <= 82) return `background: ${baseGradients.rainy};`;
  if (c >= 95) return `background: ${baseGradients.storm};`;
  return "background: #1e1e1e;";
});
</script>

<style scoped>
.glass-container {
  padding: 1rem;
  min-height: 100%;
}

.glass-effect {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 24px;
  box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.15);
}

.forecast-navigation {
  display: flex;
  justify-content: space-between;
  margin: 1rem 0;
}

.weather-header {
  text-align: center;
  color: white;
  padding: 2rem;
  margin-bottom: 1rem;
}

.weather-details {
  display: flex;
  justify-content: center;
  gap: 2rem;
  margin-top: 1.5rem;
}

.detail-item {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 1.1rem;
}

.forecast-section {
  padding: 1.5rem;
}

.forecast-section h3 {
  margin: 0 0 1rem 0;
  color: white;
  font-weight: 400;
  font-size: 1.2rem;
}

.weather-header h1 {
  margin: 0;
  font-size: 2rem;
  font-weight: 400;
  letter-spacing: -0.5px;
}

.current-temp {
  font-size: 5rem;
  font-weight: 200;
  margin: 0.5rem 0;
}

.weather-header h2 {
  margin: 0;
  font-size: 1.2rem;
  font-weight: 400;
  color: rgba(255, 255, 255, 0.8);
}

.forecast-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
  gap: 1rem;
}

ion-toolbar {
  --background: transparent;
  --border-style: none;
}

ion-header {
  background: transparent;
}

ion-content::part(scroll) {
  background: transparent;
}

ion-icon {
  font-size: 1.2rem;
  opacity: 0.9;
}
</style>
