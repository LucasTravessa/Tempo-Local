<template>
  <div class="flex flex-col flex-1 items-center">
    <!-- Banner -->
    <div
      v-if="route.query.preview"
      class="text-white p-4 bg-weather-secondary w-full text-center"
    >
      <p>
        Você está atualmente somente dando uma olhadinha, clique no "+" para
        acompanhar a cidade
      </p>
    </div>
    <!-- Weather Overview -->
    <div class="flex flex-col items-center text-white py-12">
      <h1 class="text-4xl mb-2">{{ weatherData.city.name }}</h1>
      <p class="text-sm mb-12 capitalize">
        {{
          new Date(Date.now()).toLocaleDateString("pt-br", {
            weekday: "long",
            day: "2-digit",
            month: "long",
          })
        }}
      </p>
      <p class="text-8xl mb-8">
        {{ Math.round(weatherData.list[0].main.temp) }}&deg;C
      </p>
      <p>
        Sensação termica
        {{ Math.round(weatherData.list[0].main.feels_like) }}&deg;C
      </p>
      <p class="capitalize">
        {{ weatherData.list[0].weather[0].description }}
      </p>
      <img
        class="w-[150px] h-auto"
        :src="`http://openweathermap.org/img/wn/${weatherData.list[0].weather[0].icon}@2x.png`"
        alt=""
      />
    </div>
    <hr class="border-white border-opacity-10 border w-full" />
    <!-- Hourly Weather -->
    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">Ao longo do dia</h2>
        <div class="flex gap-10 overflow-x-scroll">
          <div
            v-for="hourData in weatherData.list"
            :key="hourData.dt"
            class="flex flex-col gap-4 items-center"
          >
            <p class="whitespace-nowrap text-md">
              {{
                new Date(hourData.dt_txt).toLocaleTimeString("pt-br", {
                  hour: "numeric",
                })
              }}h
            </p>
            <img
              class="w-auto h-[50px] object-cover"
              :src="`http://openweathermap.org/img/wn/${hourData.weather[0].icon}@2x.png`"
              alt=""
            />
            <p class="text-xl">{{ Math.round(hourData.main.temp) }}&deg;C</p>
          </div>
        </div>
      </div>
    </div>
    <hr class="border-white border-opacity-10 border w-full" />
    <!-- Weekly Weather -->
    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">Nos proximos dias</h2>
        <div
          v-for="day in weatherData.list"
          :key="day.dt"
          class="flex items-center"
        >
          <p class="flex-1">
            {{
              new Date(day.dt * 1000).toLocaleDateString("pt-br", {
                weekday: "long",
              })
            }}
          </p>
          <img
            class="w-[50] h-[50px] object-cover"
            :src="`http://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png`"
            alt=""
          />
          <div class="flex gap-2 flex-1 justify-end">
            <p class="text-xl">{{ Math.round(day.main.temp) }}&deg;C</p>
          </div>
        </div>
      </div>
    </div>

    <div
      class="flex items-center gap-2 py-12 text-white cursor-pointer duration-150 hover:text-red-500"
      @click="removeCity"
    >
      <i class="fa-solid fa-trash"></i>
      <p>Remover cidade</p>
    </div>
  </div>
</template>

<script setup>
import { useRoute, useRouter } from "vue-router";

const route = useRoute();

const getWeatherData = async () => {
  try {
    let weatherData;
    await fetch(
      `https://api.openweathermap.org/data/2.5/forecast?lat=${
        route.query.lat
      }&lon=${route.query.lon}&appid=${
        import.meta.env.VITE_OPEN_WEATHER_API_KEY
      }&units=metric&lang=pt_br`
    )
      .then((response) => response.json())
      .then((value) => {
        weatherData = value;
      });
    await new Promise((res) => setTimeout(res, 200));

    return weatherData;
  } catch (error) {
    console.log(error);
  }
};

const weatherData = await getWeatherData();

const router = useRouter();
const removeCity = () => {
  const cities = JSON.parse(localStorage.getItem("savedCities"));
  const updatedCities = cities.filter((city) => city.id !== route.query.id);
  localStorage.setItem("savedCities", JSON.stringify(updatedCities));
  router.push({
    name: "home",
  });
};
</script>
