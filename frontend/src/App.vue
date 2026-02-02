<script setup>
import { ref, onMounted, computed } from 'vue'
import { RouterLink, RouterView, useRoute } from 'vue-router'

const route = useRoute()
const versions = ref([])

// Load available versions from config
onMounted(async () => {
  try {
    const response = await fetch(`${import.meta.env.BASE_URL}versions/config.json`)
    const config = await response.json()
    versions.value = config.versions
  } catch (error) {
    console.error('Failed to load version config:', error)
  }
})

const currentVersion = computed(() => route.params.version || 'Spring2026')

// Get current version config
const currentVersionConfig = computed(() => {
  const match = versions.value.find(v => v.id === currentVersion.value)
  if (!match && currentVersion.value) {
    console.warn(`No config found for version: ${currentVersion.value}`)
  }
  return match || {}
})

// Get navigation settings with defaults
const navigation = computed(() => {
  return currentVersionConfig.value.navigation || {
    home: true,
    schedule: true,
    readings: true,
    resources: true,
    staff: true
  }
})
</script>

<template>
  <header class="navbar navbar-expand-lg navbar-light bg-light border-bottom mb-4">
    <div class="container">
      <RouterLink class="navbar-brand fw-semibold" :to="`/${currentVersion}/`">CS 481E/581E — AI and Society</RouterLink>
      <button
        class="navbar-toggler"
        type="button"
        data-bs-toggle="collapse"
        data-bs-target="#navbarNav"
        aria-controls="navbarNav"
        aria-expanded="false"
        aria-label="Toggle navigation"
      >
        <span class="navbar-toggler-icon"></span>
      </button>
      <nav id="navbarNav" class="collapse navbar-collapse">
        <ul class="navbar-nav ms-auto">
          <li v-if="navigation.home" class="nav-item"><RouterLink class="nav-link" :to="`/${currentVersion}/`">Home</RouterLink></li>
          <li v-if="navigation.schedule" class="nav-item"><a class="nav-link" href="https://docs.google.com/spreadsheets/d/1AMe7DZp6VKG5EYCbvVe6EJGatuYZ7ehWqhz93GR6FhE" target="_blank" rel="noopener">Schedule</a></li>
          <li v-if="navigation.readings" class="nav-item"><RouterLink class="nav-link" :to="`/${currentVersion}/readings`">Readings</RouterLink></li>
          <li v-if="navigation.staff" class="nav-item"><RouterLink class="nav-link" :to="`/${currentVersion}/staff`">Staff</RouterLink></li>
          <li v-if="versions.filter(v => v.active).length > 1" class="nav-item dropdown">
            <a
              class="nav-link dropdown-toggle"
              href="#"
              role="button"
              data-bs-toggle="dropdown"
              aria-expanded="false"
            >
              {{ versions.find(v => v.id === currentVersion)?.displayName || currentVersion }}
            </a>
            <ul class="dropdown-menu dropdown-menu-end">
              <li v-for="version in versions.filter(v => v.active)" :key="version.id">
                <RouterLink
                  class="dropdown-item"
                  :class="{ active: version.id === currentVersion }"
                  :to="`/${version.path}/`"
                >
                  {{ version.displayName }}
                </RouterLink>
              </li>
            </ul>
          </li>
        </ul>
      </nav>
    </div>
  </header>
  <RouterView />
</template>

<style scoped>
/* Navbar spacing and minor tweaks */
</style>
