<!-- Weather.vue -->

<template>
    <div>
      <h1>Weather App</h1>
      <div v-if="weatherData">
        <h2>{{ weatherData.name }}, {{ weatherData.sys.country }}</h2>
        <p>{{ weatherData.weather[0].description }}</p>
        <p>Temperature: {{ weatherData.main.temp }}°C</p>
        <p>Humidity: {{ weatherData.main.humidity }}%</p>
        <p>Wind Speed: {{ weatherData.wind.speed }} m/s</p>
        <div v-if="weatherImage">
            <img :src="weatherImage" alt="Weather Image" />s
        </div>
        <div v-else>
            <p>Image is not Founded.</p>
        </div>
        
      </div>
    </div>
  </template>
  
  <script>
  import { ref, onMounted } from 'vue';
  import axios from 'axios';
  
  export default {
    setup() {
      const apiKey = '49bb5244435411b758a248f4038c06a3';
      const city = ref('');
      const weatherData = ref(null);
      const weatherImage = ref('');
  
      const fetchWeather = async () => {
        try {
          // If city is not provided, try fetching user's location
          if (!city.value) {
            await fetchUserLocation();
          }
  
          const response = await axios.get(
            `https://api.openweathermap.org/data/2.5/weather?q=${city.value}&appid=${apiKey}&units=metric`
          );
          weatherData.value = response.data;
          updateWeatherImage();
        } catch (error) {
          console.error('Error fetching weather data:', error);
        }
      };
  
      const updateWeatherImage = () => {
        const weatherImages = {
          'Clear': 'https://placekitten.com/200/300',
          'Clouds': 'https://placekitten.com/200/300',
          'Rain': 'https://placekitten.com/200/300',
          'Fog': 'https://placekitten.com/200/300',
          // Add more conditions as needed
        };
  
        const mainCondition = weatherData.value.weather[0].main;
        weatherImage.value = weatherImages[mainCondition] || 'https://placehold.co/600x400';
      };
  
      const fetchUserLocation = async () => {
        try {
          const position = await getCurrentPosition();
          const { latitude, longitude } = position.coords;
          const locationData = await reverseGeocode(latitude, longitude);
          city.value = locationData.city;
        } catch (error) {
          console.error('Error fetching user location:', error);
        }
      };
  
      const getCurrentPosition = () => {
        return new Promise((resolve, reject) => {
          navigator.geolocation.getCurrentPosition(resolve, reject);
        });
      };
  
      const reverseGeocode = async (latitude, longitude) => {
        const response = await axios.get(
          `https://nominatim.openstreetmap.org/reverse?format=json&lat=${latitude}&lon=${longitude}`
        );
        return response.data.address;
      };
  
      onMounted(fetchWeather);
  
      return {
        city,
        weatherData,
        weatherImage,
      };
    },
  };
  </script>
  