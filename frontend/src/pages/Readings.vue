<script setup>
import { ref, onMounted } from 'vue'
import { useVersion } from '../composables/useVersion'

const { loadVersionData } = useVersion()
const readingsData = ref(null)

onMounted(async () => {
  try {
    readingsData.value = await loadVersionData('readings.json')
  } catch (error) {
    console.error('Failed to load readings data:', error)
    readingsData.value = { topics: [] }
  }
})
</script>

<template>
  <main v-if="readingsData" class="container pb-5">
    <h1 class="h3 my-4">Course Readings</h1>

    <p v-if="!readingsData.topics.length" class="text-muted">
      Reading materials will be posted soon.
    </p>

    <!-- Top-level topics -->
    <section v-for="topic in readingsData.topics" :key="topic.id" class="mb-5">
      <h2 :id="topic.id" class="h4 mb-3">{{ topic.title }}</h2>
      <p v-if="topic.description" class="text-muted mb-3">{{ topic.description }}</p>

      <!-- Sessions within each topic -->
      <div v-for="session in topic.sessions" :key="session.id" class="card mb-3">
        <div class="card-body">
          <h5 :id="`${topic.id}-${session.id}`" class="card-title">{{ session.title }}</h5>
          <p v-if="session.description" class="card-text text-muted small">{{ session.description }}</p>

          <!-- Core readings -->
          <div v-if="session.core && session.core.length" class="mb-3">
            <h6 class="text-primary">Core Readings</h6>
            <ul class="mb-0">
              <li v-for="(material, idx) in session.core" :key="idx">
                <a :href="material.url" target="_blank" rel="noopener">{{ material.title }}</a>
                <span v-if="material.note" class="text-muted ms-1">— {{ material.note }}</span>
              </li>
            </ul>
          </div>

          <!-- Supporting materials -->
          <div v-if="session.supporting && session.supporting.length">
            <h6 class="text-secondary">Supporting Materials</h6>
            <ul class="mb-0">
              <li v-for="(material, idx) in session.supporting" :key="idx">
                <a :href="material.url" target="_blank" rel="noopener">{{ material.title }}</a>
                <span v-if="material.note" class="text-muted ms-1">— {{ material.note }}</span>
              </li>
            </ul>
          </div>

          <!-- Empty state for session -->
          <p v-if="(!session.core || !session.core.length) && (!session.supporting || !session.supporting.length)" class="text-muted mb-0 small">
            Materials coming soon.
          </p>
        </div>
      </div>

      <!-- Empty state for topic with no sessions -->
      <p v-if="!topic.sessions || !topic.sessions.length" class="text-muted">
        Sessions coming soon.
      </p>
    </section>
  </main>
</template>

<style scoped>
main { max-width: 980px; }
</style>
