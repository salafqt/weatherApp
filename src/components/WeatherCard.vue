<template>
  <div class="weather-card">
    <div class="card-content">
      <span class="day">{{ day }}</span>
      <h3>{{ time }}</h3>
      <p class="temperature">{{ temperature.toFixed(0) }}°</p>
      <span class="weather-info">{{ weatherInfo }}</span>
    </div>
  </div>
</template>

<script setup>
import { computed } from "vue";

const props = defineProps({
  time: String,
  temperature: Number,
  code: Number,
  date: String,
});

const day = computed(() => {
  const date = new Date(props.date);
  return date.toLocaleDateString("en-US", { weekday: "short" });
});

const weatherInfo = computed(() => {
  const c = props.code;
  if (c === 0) return "Clear";
  if (c >= 1 && c <= 3) return "Partly Cloudy";
  if (c >= 45 && c <= 48) return "Foggy";
  if (c >= 51 && c <= 67) return "Rain";
  if (c >= 71 && c <= 77) return "Snow";
  if (c >= 80 && c <= 82) return "Showers";
  if (c >= 95) return "Thunderstorm";
  return "Unknown";
});
</script>

<style scoped>
.weather-card {
  border-radius: 20px;
  padding: 1rem;
  height: 140px;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.weather-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.1);
}

.card-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  height: 100%;
  gap: 0.5rem;
}

.day {
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.8);
  text-transform: uppercase;
  letter-spacing: 1px;
  font-weight: 500;
}

h3 {
  margin: 0;
  font-size: 1.1rem;
  font-weight: 400;
  color: rgba(255, 255, 255, 0.9);
}

.temperature {
  margin: 0;
  font-size: 2rem;
  font-weight: 300;
  color: white;
}

.weather-info {
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.7);
  font-weight: 400;
}
</style>
