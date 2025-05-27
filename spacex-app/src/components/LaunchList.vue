<script setup lang="ts">
import { ref, watchEffect } from 'vue'
import LaunchModal from './LaunchModal.vue'

type Launch = {
  id: string
  name: string
  date_utc: string
  success: boolean
  links: {
    patch: { small: string | null }
  }
}

const launches = ref<Launch[]>([])
const filter = ref<'all' | 'success' | 'fail'>('all')
const filteredLaunches = ref<Launch[]>([])
const loading = ref(true)

// Modal
const selectedLaunch = ref<Launch | null>(null)
const showModal = ref(false)

const fetchLaunches = async () => {
  loading.value = true
  const res = await fetch('https://api.spacexdata.com/v5/launches')
  const data = await res.json()
  launches.value = data.reverse().slice(0, 200)
  updateFilter()
  loading.value = false
}

const updateFilter = () => {
  if (filter.value === 'success') {
    filteredLaunches.value = launches.value.filter(l => l.success).slice(0, 10)
  } else if (filter.value === 'fail') {
    filteredLaunches.value = launches.value.filter(l => l.success === false).slice(0, 10)
  } else {
    filteredLaunches.value = launches.value.slice(0, 10)
  }
}

watchEffect(updateFilter)

fetchLaunches()
</script>

<template>
  <section class="bg-white dark:bg-gray-900 text-gray-800 dark:text-white p-6 rounded-2xl shadow-xl mt-8">
    <h2 class="text-2xl font-bold mb-4">Derniers lancements</h2>

    <label class="block mb-4">
      <span class="text-sm font-semibold">Filtrer :</span>
      <select
        v-model="filter"
        class="mt-1 p-2 rounded border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-800"
      >
        <option value="all">Tous les lancements</option>
        <option value="success">Lancements réussis</option>
        <option value="fail">Lancements échoués</option>
      </select>
    </label>

    <div v-if="loading">Chargement...</div>
    <div v-else class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
      <div
        v-for="launch in filteredLaunches"
        :key="launch.id"
        class="p-4 border rounded-lg hover:shadow-lg transition cursor-pointer bg-gray-100 dark:bg-gray-800"
        @click="selectedLaunch = launch; showModal = true"
    >

        <h3 class="font-bold text-lg">{{ launch.name }}</h3>
        <p class="text-sm text-gray-600 dark:text-gray-400">
          {{ new Date(launch.date_utc).toLocaleDateString() }}
        </p>
       <!-- <p :class="launch.success ? 'text-green-500': 'text-red-500'">
          {{ launch.success ? 'Succès' : 'Échec' }}
        </p> -->
        <p v-if="launch.success !== null" class="text-sm text-gray-600 dark:text-gray-400">
      Statut: 
      <span :class="launch.success ? 'text-green-600' : 'text-red-600'">
        {{ launch.success ? 'Réussi' : 'Échoué' }}
      </span>
    </p>
    <p v-else class="text-sm text-gray-600 dark:text-gray-400">Pas effectué</p>
      </div>
    </div>

    <!-- Modal -->
    <LaunchModal
      v-if="showModal"
      :launch="selectedLaunch"
      @close="showModal = false"
    />
  </section>
</template>
