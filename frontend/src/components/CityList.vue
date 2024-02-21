<template>
  <div>
    <div v-for="city in saveCities" :key="city.id">
      <CityCard :city="city" @click="goToCityView(city)" />
    </div>

    <p v-if="saveCities.length === 0">
      No locations added. To start tracking a location, search in the field
      above.
    </p>
  </div>
</template>

<script setup>
import axios from "axios";
import { ref } from "vue";
import { useRouter } from "vue-router";
import CityCard from "./CityCard.vue";

const saveCities = ref([]);
const getCities = async () => {
  if (localStorage.getItem("saveCities")) {
    saveCities.value = JSON.parse(localStorage.getItem("saveCities"));

    const requests = [];

    saveCities.value.forEach((city) => {
      requests.push(
        axios.get(
          `https://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}&lon=${city.coords.lng}&appid=7efa332cf48aeb9d2d391a51027f1a71`
        )
      );
    });

    const weatherData = await Promise.all(requests);

    //Flicker Delay
    await new Promise((res) => setTimeout(res, 300));

    weatherData.forEach((value, index) => {
      saveCities.value[index].weather = value.data;
    });
  }
};
await getCities();

const router = useRouter();
const goToCityView = (city) => {
  router.push({
    name: "cityView",
    params: { state: city.state, city: city.city },
    query: {
      id: city.id,
      lat: city.coords.lat,
      lng: city.coords.lng,
    },
  });
};
</script>
