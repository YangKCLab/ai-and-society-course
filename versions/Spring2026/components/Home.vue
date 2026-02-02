<script setup>
import { ref, onMounted, computed } from 'vue'
import { useVersion } from '../../../frontend/src/composables/useVersion'

const { loadVersionData, currentVersion } = useVersion()
const homeData = ref(null)

// Base URL for assets
const baseUrl = import.meta.env.BASE_URL

// Computed syllabus PDF URL
const syllabusPdfUrl = computed(() => `${baseUrl}versions/${currentVersion.value}/content/syllabus.pdf`)

// Format date without timezone conversion
const formatDate = (dateStr) => {
  if (!dateStr) return ''
  const [year, month, day] = dateStr.split('-')
  return `${parseInt(month)}/${parseInt(day)}/${year}`
}

onMounted(async () => {
  try {
    homeData.value = await loadVersionData('home.json')
  } catch (error) {
    console.error('Failed to load home data:', error)
    homeData.value = {
      semester: 'Spring 2026',
      classTime: 'Monday & Wednesday 5:00pm-6:30pm',
      location: 'Academic Building A, Room G023',
      grading: { participation: 50, project: 40, essay: 10 },
      gradingScale: { A: '100–90', B: '89–80', C: '79–70', D: '69–60', F: '59–0' }
    }
  }
})
</script>

<template>
  <main class="container pb-5">
    <section class="mb-4">
      <h1 class="display-6">CS 481E/581E AI and Society</h1>
      <p class="lead" id="description">
        This course examines the transformative impact of artificial intelligence—particularly large language models (LLMs)—on society.
        Through seminar discussions, research paper readings, and hands-on projects, students will explore how AI systems
        are reshaping labor markets, education, scientific research, healthcare, information ecosystems, and governance.
        The course emphasizes critical analysis of both the opportunities and challenges posed by AI technologies.
      </p>
    </section>

    <section v-if="homeData" class="row gy-3 mb-5">
      <div class="col-md-12">
        <div class="card h-100">
          <div class="card-body">
            <h5 class="card-title" id="schedule">Time and Place</h5>
            <ul class="mb-0">
              <li>{{ homeData.semester }}</li>
              <li>{{ homeData.classTime }}</li>
              <li>Location: {{ homeData.location }}</li>
            </ul>
          </div>
        </div>
      </div>
    </section>

    <section class="mb-5">
      <h3>Course Objectives</h3>
      <p>
        Upon successful completion of this course, you will be able to:
      </p>
      <ul>
        <li>Understand the technical foundations of modern AI systems, particularly large language models.</li>
        <li>Critically analyze AI's impact across different sectors of society.</li>
        <li>Evaluate ethical considerations and governance challenges in AI deployment.</li>
        <li>Engage with current research literature on AI and society.</li>
        <li>Conduct independent research on AI-related societal issues.</li>
      </ul>
    </section>

    <section class="mb-5">
      <h3 id="syllabus">Syllabus</h3>
      <p>
        Download the most up-to-date syllabus (PDF):
        <a
          class="btn btn-outline-primary btn-sm ms-1"
          :href="syllabusPdfUrl"
          target="_blank"
          rel="noopener"
        >
          View Syllabus
        </a>
        <span v-if="homeData?.syllabusLastUpdated" class="text-muted ms-2">
          (Last updated: {{ formatDate(homeData.syllabusLastUpdated) }})
        </span>
      </p>
      <p>
        The syllabus is subject to change, so please check it regularly for updates.
      </p>
    </section>


    <section class="mb-5">
      <h3 id="materials">Course Materials</h3>
      <p>There is no textbook for this course.</p>

      <p>Reading assignments (research papers and articles) will be made available on the <a href="https://docs.google.com/spreadsheets/d/1AMe7DZp6VKG5EYCbvVe6EJGatuYZ7ehWqhz93GR6FhE" target="_blank" rel="noopener">Schedule</a>.</p>

      <p>Slides and other course materials will be made available via Brightspace.</p>
    </section>

    <section class="mb-5">
      <h3>Course Format and Topics</h3>
      <p>
        This seminar-style course combines brief lectures with extensive discussion of research papers and current events.
        Students will actively participate in discussions and present papers to the class.

        Please refer to the <a href="https://docs.google.com/spreadsheets/d/1AMe7DZp6VKG5EYCbvVe6EJGatuYZ7ehWqhz93GR6FhE" target="_blank" rel="noopener">Schedule</a> for the detailed topics and reading materials.
      </p>
      <div class="row g-3">
        <div class="col-md-6">
          <div class="card h-100">
            <div class="card-body">
              <h6 class="card-title">Seminar Topics</h6>
              <ul class="mb-0">
                <li>Introduction to LLMs and AI foundations</li>
                <li>AI and labor markets</li>
                <li>AI in education</li>
                <li>AI in scientific research</li>
                <li>AI in healthcare</li>
                <li>AI and information ecosystems</li>
                <li>AI governance and ethics</li>
              </ul>
            </div>
          </div>
        </div>
        <div class="col-md-6">
          <div class="card h-100">
            <div class="card-body">
              <h6 class="card-title">Key Themes</h6>
              <ul class="mb-0">
                <li>Capabilities and limitations of AI systems</li>
                <li>Societal benefits and risks</li>
                <li>Bias, fairness, and accountability</li>
                <li>Privacy and security concerns</li>
                <li>Policy and regulatory frameworks</li>
              </ul>
            </div>
          </div>
        </div>
      </div>
    </section>

    <section class="mb-5">
      <h3>Prerequisites</h3>
      <ul>
        <li>CS 436/536 Introduction to Machine Learning, or</li>
        <li>CS 465/565 Introduction to Artificial Intelligence</li>
        <li>Familiarity with Python programming</li>
      </ul>
    </section>

    <section v-if="homeData" class="mb-5">
      <h3 id="grading">Method of Assessment</h3>
      <ul>
        <li>Participation (discussions and presentations): {{ homeData.grading.participation }}%</li>
        <li>Course project: {{ homeData.grading.project }}%</li>
        <li>Final essay: {{ homeData.grading.essay }}%</li>
      </ul>
      <h4 class="mt-3">Grading Scale</h4>
      <ul>
        <li>A: {{ homeData.gradingScale.A }}</li>
        <li>B: {{ homeData.gradingScale.B }}</li>
        <li>C: {{ homeData.gradingScale.C }}</li>
        <li>D: {{ homeData.gradingScale.D }}</li>
        <li>F: {{ homeData.gradingScale.F }}</li>
      </ul>
    </section>

    <footer class="text-center text-muted">
      <hr />
      <div>Last updated {{ new Date().toLocaleDateString() }}.</div>
      <div>
        Source on <a href="https://github.com/YangKCLab/ai-and-society-course" target="_blank" rel="noopener">GitHub</a>.
      </div>
    </footer>
  </main>
</template>

<style scoped>
main {
  max-width: 980px;
}
.card-title {
  margin-bottom: 0.5rem;
}
</style>
