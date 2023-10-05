<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        type="text"
        v-model="searchQuery"
        @input="getSearchResults"
        placeholder="Procure por cidade ou estado"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
      />
      <ul
        v-if="geocodeMapResults"
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
      >
        <p v-if="searchError">Alguma coisa deu errado, tente novamente!</p>
        <p v-if="!serverError && geocodeMapResults.length === 0">
          Nenhum resultado encontrado, tente outro nome!
        </p>
        <template v-else>
          <li
            v-for="searchResult in geocodeMapResults"
            class="py-2 cursor-pointer"
            @click="previewCity(searchResult)"
          >
            {{
              `${searchResult.name}, ${searchResult.state}, ${searchResult.country}`
            }}
          </li>
        </template>
      </ul>
    </div>
    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList />
        <template #fallback>
          <CityCardSkeleton />
        </template>
      </Suspense>
    </div>
  </main>
</template>
<script setup>
import { ref } from "vue";
import { useRouter } from "vue-router";
import CityList from "../components/CityList.vue";
import CityCardSkeleton from "../components/CityCardSkeleton.vue";

const router = useRouter();

const previewCity = (searchResult) => {
  // console.log(searchResult);
  router.push({
    name: "cityView",
    params: { state: searchResult.state, city: searchResult.name },
    query: {
      lat: searchResult.lat,
      lon: searchResult.lon,
      preview: true,
    },
  });
};

const searchQuery = ref("");
const quearyTimeout = ref(null);
const geocodeMapResults = ref(null);
const searchError = ref(null);

const getSearchResults = () => {
  clearTimeout(quearyTimeout.value);
  quearyTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== "") {
      await fetch(
        `https://api.openweathermap.org/geo/1.0/direct?q=${
          searchQuery.value
        }&limit=5&appid=${import.meta.env.VITE_OPEN_WEATHER_API_KEY}`
      )
        .then((response) => response.json())
        .then((value) => {
          geocodeMapResults.value = value;
          console.log(geocodeMapResults.value);
        })
        .catch((error) => (searchError.value = true));

      return;
    }
    geocodeMapResults.value = null;
  }, 300);
};
</script>
