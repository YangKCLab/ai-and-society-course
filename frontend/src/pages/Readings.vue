<script setup>
import { ref, computed, onMounted } from 'vue'
import { useVersion } from '../composables/useVersion'

const { loadVersionData } = useVersion()
const readingsData = ref(null)

onMounted(async () => {
  try {
    readingsData.value = await loadVersionData('readings.json')
  } catch (error) {
    console.error('Failed to load readings data:', error)
    readingsData.value = { groups: [] }
  }
})

const displayGroups = computed(() => {
  if (!readingsData.value) return []
  if (readingsData.value.groups) {
    // Normalize: flatten topics[].sessions into sessions[] if needed
    return readingsData.value.groups.map(g => {
      if (g.sessions) return g
      const sessions = (g.topics || []).flatMap(t => t.sessions || [])
      return { ...g, sessions }
    })
  }
  // Backward compat: _template has { topics: [] }
  return [{ id: 'all', title: '', sessions: [] }]
})
</script>

<template>
  <main v-if="readingsData" class="container pb-5">
    <h1 class="h3 my-4">Course Readings</h1>
    <p v-if="readingsData.lastUpdated" class="text-muted small mb-4">
      Last updated: {{ readingsData.lastUpdated }}
    </p>

    <p v-if="!displayGroups.length || displayGroups.every(g => !g.sessions.length)" class="text-muted">
      Reading materials will be posted soon.
    </p>

    <!-- Table of Contents -->
    <nav v-if="displayGroups.some(g => g.sessions.length)" class="mb-4">
      <ul class="list-unstyled mb-0">
        <template v-for="group in displayGroups" :key="'toc-' + group.id">
          <li v-if="group.title" class="mb-1">
            <details class="toc-group">
              <summary>
                <a :href="'#' + group.id" class="fw-semibold text-decoration-none">{{ group.title }}</a>
              </summary>
              <ul class="list-unstyled ms-4 mt-1">
                <li v-for="session in group.sessions" :key="'toc-' + session.id">
                  <a :href="'#' + session.id" class="text-decoration-none small">{{ session.title }}</a>
                </li>
              </ul>
            </details>
          </li>
        </template>
      </ul>
    </nav>

    <!-- Groups -->
    <div v-for="(group, groupIdx) in displayGroups" :key="group.id">
      <hr v-if="group.title && groupIdx > 0" class="mt-5 mb-4">
      <h2 v-if="group.title" :id="group.id" class="h4 mb-3">
        {{ group.title }}
      </h2>

      <!-- Sessions within each group -->
      <div v-for="session in group.sessions" :key="session.id" class="card mb-3">
        <div class="card-body">
          <h5 :id="session.id" class="card-title">{{ session.title }}</h5>
          <p v-if="session.description" class="card-text text-muted small">{{ session.description }}</p>

          <!-- Core readings -->
          <div v-if="session.core?.length" class="mb-3">
            <h6 class="text-primary">Core Readings</h6>
            <ul class="mb-0">
              <li v-for="(material, idx) in session.core" :key="idx">
                <template v-if="material.links && material.links.length">
                  {{ material.title }}
                  <span v-if="material.note" class="text-muted ms-1">— {{ material.note }}</span>
                  <ul class="mb-0 ms-3">
                    <li v-for="(link, linkIdx) in material.links" :key="linkIdx">
                      <a :href="link.url" target="_blank" rel="noopener">{{ link.label }}</a>
                    </li>
                  </ul>
                </template>
                <template v-else>
                  <a :href="material.url" target="_blank" rel="noopener">{{ material.title }}</a>
                  <span v-if="material.note" class="text-muted ms-1">— {{ material.note }}</span>
                </template>
              </li>
            </ul>
          </div>

          <!-- Supporting materials -->
          <div v-if="session.supporting?.length">
            <h6 class="text-secondary">Supporting Materials</h6>
            <ul class="mb-0">
              <li v-for="(material, idx) in session.supporting" :key="idx">
                <template v-if="material.links && material.links.length">
                  {{ material.title }}
                  <span v-if="material.note" class="text-muted ms-1">— {{ material.note }}</span>
                  <ul class="mb-0 ms-3">
                    <li v-for="(link, linkIdx) in material.links" :key="linkIdx">
                      <a :href="link.url" target="_blank" rel="noopener">{{ link.label }}</a>
                    </li>
                  </ul>
                </template>
                <template v-else>
                  <a :href="material.url" target="_blank" rel="noopener">{{ material.title }}</a>
                  <span v-if="material.note" class="text-muted ms-1">— {{ material.note }}</span>
                </template>
              </li>
            </ul>
          </div>

          <!-- Empty state for session -->
          <p v-if="!session.core?.length && !session.supporting?.length" class="text-muted mb-0 small">
            Materials coming soon.
          </p>
        </div>
      </div>
    </div>
  </main>
</template>

<style scoped>
main { max-width: 980px; }
.toc-group summary {
  cursor: pointer;
}
</style>
