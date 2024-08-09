<script setup lang="ts">
import { computed, onMounted, ref, type Ref } from 'vue'
import { Search } from 'lucide-vue-next'

const weatherData: Ref<any> = ref(null)

const dailyWeather = ref([])

const hourlyWeather: Ref<any[]> = ref([])

const city: Ref<string> = ref('')

const apiKey = 'api_key'

onMounted(getCurrentPositionWeather)

async function getCityWeather() {
    const response = await fetch(
        `http://api.openweathermap.org/data/2.5/weather?q=${city.value}&appid=${apiKey}`
    )
    weatherData.value = await response.json()
    city.value = ''
    getHourlyForecast(weatherData.value.name)
}

async function getHourlyForecast(cityName: string) {
    const response = await fetch(
        `http://api.openweathermap.org/data/2.5/forecast?q=${cityName}&appid=${apiKey}`
    )
    const cityData = await response.json()
    hourlyWeather.value = cityData.list.filter(
        (forecast: any) => new Date(forecast.dt * 1000).getDate() === new Date().getDate()
    )
}

async function getCurrentPositionWeather() {
    if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(async (position) => {
            weatherData.value = null
            const { latitude, longitude } = position.coords
            const response = await fetch(
                `http://api.openweathermap.org/data/2.5/weather?lat=${latitude}&lon=${longitude}&appid=${apiKey}`
            )
            weatherData.value = await response.json()

            getHourlyForecast(weatherData.value.name)
        })
    } else {
        alert('Non è possibile recuperare la posizione corrente')
    }
}

const icon = computed(
    () => `http://api.openweathermap.org/img/w/${weatherData.value.weather[0].icon}@2x.png`
)
const temperature = computed(() => {
    return formatTemperature(weatherData.value.main.temp)
})

function formatTemperature(temperature: number): number {
    return Math.round((temperature - 273.15) * 10) / 10
}
</script>

<template>
    <!--Search box-->
    <div class="max-w-sm mx-auto relative mt-2 rounded-md shadow-sm">
        <form @submit.prevent="getCityWeather">
            <input
                type="text"
                v-model="city"
                required
                placeholder="Inserisci il nome della città"
                class="block w-full rounded-md border-0 py-1.5 pl-7 pr-20 text-gray-900 ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600 sm:text-sm sm:leading-6 hover:shadow-inner"
            />
            <div class="absolute inset-y-0 right-0 flex items-center">
                <button
                    class="h-full rounded-md border-0 bg-transparent py-0 px-2 text-gray-500 focus:ring-2 focus:ring-inset focus:ring-indigo-600 sm:text-sm hover:bg-gray-300"
                >
                    <Search />
                </button>
            </div>
        </form>
    </div>
    <!--Header-->
    <div v-if="weatherData" class="flex w-full mx-auto justify-center items-center">
        <h1 class="text-3xl">{{ weatherData.name }}, {{ weatherData.sys.country }}</h1>
        <img :src="icon" />
        <p class="pl-3">{{ temperature }} °C</p>
        <span class="italic text-gray-500 dark:text-gray-300 pl-3">{{
            weatherData.weather[0].description
        }}</span>
        <span class="pl-3">{{ weatherData.wind.speed }} m/s</span>
    </div>
    <div v-else>Loading...</div>
    <!--Forecast-->
    <div v-if="dailyWeather.length">
        <div>Meteo giornaliero</div>
    </div>
    <!--Daily Detail-->
    <div class="relative overflow-x-auto">
        <table v-if="hourlyWeather.length" class="table-auto w-full text-center items-center">
            <thead>
                <th>Ora</th>
                <th>Tempo</th>
                <th>Temperatura</th>
                <th>Cielo</th>
                <th>Vento</th>
            </thead>
            <tbody>
                <tr
                    v-for="(forecast, index) in hourlyWeather"
                    :key="index"
                    class="odd:bg-white odd:dark:bg-gray-900 even:bg-gray-50 even:dark:bg-gray-800 border-b dark:border-gray-700 dark:text-white"
                >
                    <td>{{ new Date(forecast.dt * 1000).getHours() }}</td>
                    <td class="flex justify-center items-center">
                        <img
                            :src="`http://api.openweathermap.org/img/w/${forecast.weather[0].icon}.png`"
                        />
                    </td>
                    <td>{{ formatTemperature(forecast.main.temp) }} °C</td>
                    <td class="italic text-gray-500 dark:text-gray-300">
                        {{ forecast.weather[0].description }}
                    </td>
                    <td>{{ forecast.wind.speed }} m/s</td>
                </tr>
            </tbody>
        </table>
    </div>
</template>

<style scoped></style>
