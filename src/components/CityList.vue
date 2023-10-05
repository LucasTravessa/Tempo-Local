<template>
  <div v-for="city in savedCities" :key="city.id">
    <CityCard :city="city" @click="goToCityView(city)" />
  </div>
  <p v-if="savedCities.length === 0">
    Nenhum local adicionado. Para começar a acompanhar algum local, procure no
    campo acima.
  </p>
</template>

<script setup>
import { ref } from "vue";
import CityCard from "./CityCard.vue";
import { useRouter } from "vue-router";

const savedCities = ref([]);
const getCities = async () => {
  if (localStorage.getItem("savedCities")) {
    savedCities.value = JSON.parse(localStorage.getItem("savedCities"));
    const requests = [];
    savedCities.value.forEach((city) => {
      requests.push(
        fetch(
          `https://api.openweathermap.org/data/2.5/weather?lat=${
            city.coords.lat
          }&lon=${city.coords.lon}&appid=${
            import.meta.env.VITE_OPEN_WEATHER_API_KEY
          }&units=metric&lang=pt_br`
        ).then((response) => response.json())
      );
    });

    const weatherData = await Promise.all(requests);
    await new Promise((res) => setTimeout(res, 200));
    weatherData.forEach((value, index) => {
      savedCities.value[index].weather = value;
    });
  }
};

await getCities();

const router = useRouter();
const goToCityView = (city) => {
  router.push({
    name: "cityView",
    params: { state: city.state, city: city.city },
    query: { id: city.id, lat: city.coords.lat, lon: city.coords.lon },
  });
};
</script>
