<template>
  <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
    <div class="bg-white dark:bg-gray-900 rounded-2xl shadow-2xl p-6 w-full max-w-3xl relative">
      <button
        @click="$emit('close')"
        class="absolute top-4 right-4 text-gray-500 hover:text-red-600 text-2xl"
      >
        &times;
      </button>

      <h2 class="text-2xl font-bold mb-2">{{ launch.name }}</h2>
      <p class="text-sm text-gray-600 dark:text-gray-300 mb-2">
        {{ formatDate(launch.date_utc) }}
      </p>

      <img
        v-if="launch.links.patch.small"
        :src="launch.links.patch.small"
        alt="Patch mission"
        class="w-24 h-24 object-contain mb-4"
      />

      <p class="mb-4">{{ launch.details || 'No details available.' }}</p>

      <a
        v-if="launch.links.article"
        :href="launch.links.article"
        target="_blank"
        class="text-blue-500 underline mb-4 block"
      >
        Lire l'article
      </a>

      <!-- Switch pour la vidéo -->
      <div class="flex items-center gap-2 mb-4">
        <label class="font-medium">Voir la vidéo :</label>
        <input type="checkbox" v-model="showVideo" />
      </div>

      <!-- Vidéo YouTube -->
      <div v-if="showVideo && launch.links.youtube_id" class="aspect-w-16 aspect-h-9 mb-4">
        <iframe
          class="w-full h-64"
          :src="`https://www.youtube.com/embed/${launch.links.youtube_id}`"
          frameborder="0"
          allowfullscreen
        ></iframe>
      </div>

      <!-- Données supplémentaires -->
      <div v-if="launch.launchpad" class="mt-4">
        <p><strong>Lieu de lancement :</strong> {{ launch.launchpad.name }}</p>
      </div>

      <div v-if="launch.payloads && launch.payloads.length">
        <p><strong>Chargements :</strong> {{ launch.payloads.map(p => p.name).join(', ') }}</p>
        <p><strong>Clients :</strong> {{ getClients(launch.payloads).join(', ') }}</p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'

const props = defineProps<{
  launch: any
}>()
const emit = defineEmits(['close'])

const showVideo = ref(false)

// Formatage de la date
const formatDate = (date: string) => {
  const d = new Date(date)
  return d.toLocaleDateString('fr-FR', {
    day: '2-digit',
    month: '2-digit',
    year: 'numeric',
  })
}

// Récupération des infos supplémentaires : payloads + launchpad
const fetchAdditionalData = async () => {
  const payloads = await Promise.all(
    props.launch.payloads.map(async (id: string) => {
      const res = await fetch(`https://api.spacexdata.com/v4/payloads/${id}`)
      return await res.json()
    })
  )

  const launchpadRes = await fetch(`https://api.spacexdata.com/v4/launchpads/${props.launch.launchpad}`)
  const launchpad = await launchpadRes.json()

  props.launch.payloads = payloads
  props.launch.launchpad = launchpad
}

onMounted(() => {
  fetchAdditionalData()
})

// Extraire les clients
const getClients = (payloads: any[]) => {
  const clients = payloads.flatMap(p => p.customers || [])
  return [...new Set(clients)] // Suppression des doublons
}
</script>

<style scoped>
.aspect-w-16 {
  aspect-ratio: 16 / 9;
}
</style>
