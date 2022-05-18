<template>
  <div v-if="data" class="h-screen relative overflow-hidden">
    <img :src="background" />
    <div class="absolute w-full h-full top-0 overlay" />
    <div class="absolute w-full h-full top-0 p-48">
      <div class="flex justify-between">
        <div>
          <h1 class="text-7xl text-white">{{ data.name }}</h1>
          <p class="font-extralight text-2xl mt-2 text-white">{{ today }}</p>
          <img
            :src="`https://openweathermap.org/img/wn/${data.weather[0].icon}@4x.png`"
            class="w-56 icon"
          />
        </div>
        <div>
          <p class="text-9xl text-white font-extralight">
            {{ data.main.temp }}°
          </p>
        </div>
      </div>
      <div class="mt-20">
        <input
          type="text"
          class="w-1/2 h-11"
          placeholder="Search a city..."
          v-model="input"
        />
        <button class="bg-sky-400 w-20 text-white h-10" @click="handleClick">
          Search
        </button>
      </div>
    </div>
  </div>
  <div v-else class="p-10">
    <h1 class="text-7xl">Can't get data</h1>
    <h2 class="text-4xl">{{ error }}</h2>
    <button class="mt-5 bg-sky-400 px-10 w-50 text-white h-10" @click="goBack">
      Go Back
    </button>
  </div>
</template>

<script setup lang="ts">
const config = useRuntimeConfig()
const cookie = useCookie('city')
if (!cookie.value) cookie.value = 'Toronto'

const search = ref(cookie.value)
const input = ref('')
const background = ref('')
// const { data, error } = useFetch(
//   () =>
//     `https://api.openweathermap.org/data/2.5/weather?q=${search.value}&units=metric&appid=api_key`
// )

const { data, error } = useAsyncData(
  'city',
  async () => {
    let response

    try {
      response = await $fetch(
        `https://api.openweathermap.org/data/2.5/weather?q=${search.value}`,
        {
          params: {
            units: 'metric',
            appid: config.WEATHER_APP_SECRET,
          },
        }
      )
      cookie.value = search.value

      const temp = response.main.temp
      if (temp <= -10) {
        background.value =
          'https://images.unsplash.com/photo-1483664852095-d6cc6870702d?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=3540&q=80'
      }
      if (temp > -10 && temp <= 0) {
        background.value =
          'https://images.unsplash.com/photo-1476820865390-c52aeebb9891?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=3540&q=80'
      }
      if (temp > 0 && temp <= 10) {
        background.value =
          'https://images.unsplash.com/photo-1560258018-c7db7645254e?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=4032&q=80'
      }
      if (temp > 10) {
        background.value =
          'https://images.unsplash.com/photo-1507525428034-b723cf961d3e?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=3546&q=80'
      }
    } catch (err) {
      console.log(error.value)
    }
    return response
  },
  {
    watch: [search],
  }
)

const today = new Date().toLocaleDateString('pl-PL', {
  weekday: 'long',
  year: 'numeric',
  month: 'long',
  day: 'numeric',
})

const handleClick = () => {
  const formatedSearch = input.value.trim().replace(' ', '+')
  search.value = formatedSearch
  input.value = ''
}

const goBack = () => {
  search.value = cookie.value
}
</script>

<style scoped>
.overlay {
  background-color: rgba(0, 0, 0, 0.5);
}
.icon {
  margin-left: -2.75rem;
  margin-top: -2.75rem;
}
</style>
