<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const launch = ref<any>(null)
const countdown = ref<number>(0)
let intervalId: number

const fetchNextLaunch = async () => {
  const res = await fetch('https://api.spacexdata.com/v5/launches/next')
  launch.value = await res.json()
  updateCountdown()
  intervalId = setInterval(updateCountdown, 1000)
}

const updateCountdown = () => {
  if (!launch.value?.date_utc) return
  const launchTime = new Date(launch.value.date_utc).getTime()
  const now = Date.now()
  countdown.value = Math.max(Math.floor((launchTime - now) / 1000), 0)
}

onMounted(fetchNextLaunch)
onUnmounted(() => clearInterval(intervalId))
</script>

<template>
  <section class="bg-gray-900 text-white p-6 rounded-2xl shadow-xl">
    <h2 class="text-2xl font-bold mb-4"> Prochain lancement</h2>
    <div v-if="launch">
      <p><strong>Nom :</strong> {{ launch.name }}</p>
      <p><strong>Date :</strong> {{ new Date(launch.date_utc).toLocaleString() }}</p>
      <p><strong>Décompte :</strong> {{ countdown }} secondes</p>
    </div>
    <p v-else>Chargement...</p>
  </section>
</template>

<style scoped>

</style>
