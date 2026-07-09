<template>
  <div class="app">
    <div class="bg-effects">
      <div class="orb orb-1"></div>
      <div class="orb orb-2"></div>
      <div class="orb orb-3"></div>
      <div class="orb orb-4"></div>
    </div>

    <header class="app-header">
      <div class="header-content">
        <div class="header-badge">GINCANA</div>
        <h1 class="app-title">Placar das Casas</h1>
        <p class="app-subtitle">Acompanhe a pontuação em tempo real</p>
      </div>
      <div class="header-logos">
        <img
          v-for="team in sortedTeams"
          :key="team.id"
          :src="team.logo"
          :alt="team.name"
          class="header-logo-mini"
          :style="{ filter: `drop-shadow(0 2px 8px ${team.primaryColor}66)` }"
        />
      </div>
    </header>

    <main class="main-content">
      <section class="section">
        <div class="section-label">Classificação Geral</div>
        <div class="cards-grid">
          <TeamCard
            v-for="(team, idx) in sortedTeams"
            :key="team.id"
            :team="team"
            :position="idx + 1"
            :is-leader="idx === 0 && team.score > 0"
            :max-score="maxScore"
          />
        </div>
      </section>

      <section class="section chart-section">
        <div class="section-label">Gráfico de Pontuação</div>
        <div class="chart-container">
          <ScoreChart :teams="sortedTeams" />
        </div>
      </section>

      <section class="section">
        <div class="section-label">
          <button class="toggle-editor-btn" @click="showEditor = !showEditor">
            <span>{{ showEditor ? '▲ Fechar' : '▼ Abrir' }} Painel de Pontuação</span>
          </button>
        </div>
        <Transition name="slide-fade">
          <ScoreEditor
            v-if="showEditor"
            :teams="teams"
            @update:teams="updateTeams"
          />
        </Transition>
      </section>
    </main>

    <footer class="app-footer">
      <span>TSBR · Gincana {{ currentYear }}</span>
    </footer>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import TeamCard from './components/TeamCard.vue'
import ScoreChart from './components/ScoreChart.vue'
import ScoreEditor from './components/ScoreEditor.vue'
import { teams as initialTeams } from './data/teams.js'

const STORAGE_KEY = 'tsbr-gincana-scores'

function loadTeams() {
  try {
    const saved = localStorage.getItem(STORAGE_KEY)
    if (saved) {
      const parsed = JSON.parse(saved)
      return initialTeams.map((t) => {
        const found = parsed.find((p) => p.id === t.id)
        return found ? { ...t, score: found.score } : t
      })
    }
  } catch {}
  return initialTeams.map((t) => ({ ...t }))
}

const teams = ref(loadTeams())
const showEditor = ref(false)
const currentYear = new Date().getFullYear()

const sortedTeams = computed(() =>
  [...teams.value].sort((a, b) => b.score - a.score)
)

const maxScore = computed(() =>
  Math.max(...teams.value.map((t) => t.score), 1)
)

function updateTeams(updatedTeams) {
  teams.value = updatedTeams
  try {
    localStorage.setItem(
      STORAGE_KEY,
      JSON.stringify(updatedTeams.map(({ id, score }) => ({ id, score })))
    )
  } catch {}
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800;900&display=swap');

*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  -webkit-font-smoothing: antialiased;
}

body {
  background: #0a0e1a;
  color: #e2e8f0;
  min-height: 100vh;
}
</style>

<style scoped>
.app {
  min-height: 100vh;
  position: relative;
  overflow-x: hidden;
}

.bg-effects {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 0;
  overflow: hidden;
}

.orb {
  position: absolute;
  border-radius: 50%;
  filter: blur(80px);
  opacity: 0.35;
}

.orb-1 {
  width: 500px;
  height: 500px;
  background: radial-gradient(circle, #1e3a5f, transparent 70%);
  top: -200px;
  left: -100px;
  animation: drift 18s ease-in-out infinite;
}

.orb-2 {
  width: 400px;
  height: 400px;
  background: radial-gradient(circle, #4a0e15, transparent 70%);
  top: 30%;
  right: -150px;
  animation: drift 22s ease-in-out infinite reverse;
}

.orb-3 {
  width: 350px;
  height: 350px;
  background: radial-gradient(circle, #f5c51830, transparent 70%);
  bottom: 10%;
  left: 20%;
  animation: drift 25s ease-in-out infinite 4s;
}

.orb-4 {
  width: 300px;
  height: 300px;
  background: radial-gradient(circle, #8b1a2a30, transparent 70%);
  bottom: -100px;
  right: 10%;
  animation: drift 20s ease-in-out infinite 8s reverse;
}

@keyframes drift {
  0%, 100% { transform: translate(0, 0) scale(1); }
  33% { transform: translate(30px, -20px) scale(1.05); }
  66% { transform: translate(-20px, 30px) scale(0.95); }
}

.app-header {
  position: relative;
  z-index: 1;
  text-align: center;
  padding: 3rem 1.5rem 2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.5rem;
}

.header-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
}

.header-badge {
  display: inline-block;
  background: linear-gradient(135deg, #6366f1, #8b5cf6);
  color: white;
  font-size: 0.7rem;
  font-weight: 800;
  letter-spacing: 3px;
  padding: 0.35rem 1rem;
  border-radius: 99px;
  text-transform: uppercase;
}

.app-title {
  font-size: clamp(2rem, 6vw, 3.5rem);
  font-weight: 900;
  color: #f7fafc;
  letter-spacing: -1px;
  line-height: 1.1;
  background: linear-gradient(135deg, #f7fafc 0%, #a0aec0 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.app-subtitle {
  font-size: 1rem;
  color: #718096;
  font-weight: 500;
}

.header-logos {
  display: flex;
  gap: 0.75rem;
  align-items: center;
}

.header-logo-mini {
  width: 52px;
  height: 52px;
  object-fit: contain;
  transition: transform 0.3s ease;
}

.header-logo-mini:hover {
  transform: scale(1.15) translateY(-3px);
}

.main-content {
  position: relative;
  z-index: 1;
  max-width: 1100px;
  margin: 0 auto;
  padding: 0 1.5rem 4rem;
  display: flex;
  flex-direction: column;
  gap: 3rem;
}

.section {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
}

.section-label {
  font-size: 0.75rem;
  font-weight: 700;
  color: #4a5568;
  text-transform: uppercase;
  letter-spacing: 2px;
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.section-label::before,
.section-label::after {
  content: '';
  flex: 1;
  height: 1px;
  background: rgba(255, 255, 255, 0.06);
}

.cards-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1.25rem;
}

.chart-section .section-label {
  margin-bottom: 0.5rem;
}

.chart-container {
  background: linear-gradient(135deg, #1a202c 0%, #2d3748 100%);
  border: 1px solid rgba(255, 255, 255, 0.06);
  border-radius: 24px;
  padding: 1.5rem 1rem 1rem;
}

.toggle-editor-btn {
  background: rgba(99, 102, 241, 0.1);
  border: 1px dashed rgba(99, 102, 241, 0.3);
  color: #a5b4fc;
  font-size: 0.75rem;
  font-weight: 700;
  letter-spacing: 1.5px;
  text-transform: uppercase;
  padding: 0.5rem 1.2rem;
  border-radius: 99px;
  cursor: pointer;
  transition: all 0.2s;
  font-family: inherit;
  white-space: nowrap;
}

.toggle-editor-btn:hover {
  background: rgba(99, 102, 241, 0.2);
  border-color: rgba(99, 102, 241, 0.6);
  color: #c7d2fe;
}

.slide-fade-enter-active,
.slide-fade-leave-active {
  transition: all 0.35s cubic-bezier(0.4, 0, 0.2, 1);
}

.slide-fade-enter-from,
.slide-fade-leave-to {
  opacity: 0;
  transform: translateY(-12px);
}

.app-footer {
  position: relative;
  z-index: 1;
  text-align: center;
  padding: 2rem 1rem;
  font-size: 0.75rem;
  color: #2d3748;
  letter-spacing: 2px;
  font-weight: 600;
  text-transform: uppercase;
  border-top: 1px solid rgba(255, 255, 255, 0.04);
}

@media (max-width: 640px) {
  .cards-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 420px) {
  .cards-grid {
    grid-template-columns: 1fr;
  }
}
</style>
