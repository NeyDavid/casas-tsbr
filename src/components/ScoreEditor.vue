<template>
  <div class="editor-panel">
    <div class="editor-header">
      <div class="editor-icon">⚙️</div>
      <div>
        <h3 class="editor-title">Painel de Pontuação</h3>
        <p class="editor-subtitle">Atualize os pontos de cada casa</p>
      </div>
    </div>

    <div class="editor-grid">
      <div
        v-for="team in localTeams"
        :key="team.id"
        class="editor-row"
        :style="{ '--team-color': team.primaryColor }"
      >
        <div class="editor-team-info">
          <img :src="team.logo" :alt="team.name" class="editor-logo" />
          <span class="editor-team-name">{{ team.name }}</span>
        </div>
        <div class="input-wrapper">
          <button class="qty-btn" @click="adjustScore(team, -100)">−</button>
          <input
            type="number"
            v-model.number="team.score"
            min="0"
            class="score-input"
            @change="validateScore(team)"
          />
          <button class="qty-btn" @click="adjustScore(team, 100)">+</button>
        </div>
      </div>
    </div>

    <button class="save-btn" @click="saveScores">
      <span>Salvar Pontuações</span>
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="18" height="18">
        <path d="M19 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11l5 5v11a2 2 0 0 1-2 2z"/>
        <polyline points="17 21 17 13 7 13 7 21"/>
        <polyline points="7 3 7 8 15 8"/>
      </svg>
    </button>

    <div class="saved-toast" :class="{ visible: showToast }">
      ✅ Pontuações salvas!
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  teams: { type: Array, required: true },
})

const emit = defineEmits(['update:teams'])

const localTeams = ref(props.teams.map((t) => ({ ...t })))
const showToast = ref(false)

watch(
  () => props.teams,
  (newTeams) => {
    localTeams.value = newTeams.map((t) => ({ ...t }))
  },
  { deep: true }
)

function adjustScore(team, delta) {
  team.score = Math.max(0, (team.score || 0) + delta)
}

function validateScore(team) {
  if (isNaN(team.score) || team.score < 0) team.score = 0
}

function saveScores() {
  emit('update:teams', localTeams.value.map((t) => ({ ...t })))
  showToast.value = true
  setTimeout(() => (showToast.value = false), 2500)
}
</script>

<style scoped>
.editor-panel {
  background: linear-gradient(135deg, #1a202c 0%, #2d3748 100%);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 24px;
  padding: 2rem;
  position: relative;
  overflow: hidden;
}

.editor-panel::before {
  content: '';
  position: absolute;
  top: -80px;
  right: -80px;
  width: 200px;
  height: 200px;
  background: radial-gradient(circle, rgba(99, 102, 241, 0.1) 0%, transparent 70%);
  pointer-events: none;
}

.editor-header {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1.8rem;
}

.editor-icon {
  font-size: 1.8rem;
  width: 52px;
  height: 52px;
  background: rgba(99, 102, 241, 0.15);
  border-radius: 14px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.editor-title {
  font-size: 1.1rem;
  font-weight: 700;
  color: #e2e8f0;
  margin: 0 0 0.25rem;
}

.editor-subtitle {
  font-size: 0.8rem;
  color: #718096;
  margin: 0;
}

.editor-grid {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.editor-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  background: rgba(0, 0, 0, 0.2);
  border-radius: 14px;
  padding: 0.85rem 1rem;
  border: 1px solid rgba(255, 255, 255, 0.04);
  transition: border-color 0.2s;
}

.editor-row:hover {
  border-color: color-mix(in srgb, var(--team-color) 30%, transparent);
}

.editor-team-info {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.editor-logo {
  width: 38px;
  height: 38px;
  object-fit: contain;
  filter: drop-shadow(0 2px 6px rgba(0, 0, 0, 0.4));
}

.editor-team-name {
  font-size: 0.95rem;
  font-weight: 700;
  color: #e2e8f0;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.input-wrapper {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.qty-btn {
  width: 32px;
  height: 32px;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  background: rgba(255, 255, 255, 0.05);
  color: #e2e8f0;
  font-size: 1.1rem;
  cursor: pointer;
  transition: all 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
  line-height: 1;
}

.qty-btn:hover {
  background: color-mix(in srgb, var(--team-color) 20%, rgba(255, 255, 255, 0.1));
  border-color: var(--team-color);
}

.score-input {
  width: 90px;
  background: rgba(0, 0, 0, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 10px;
  color: var(--team-color, #e2e8f0);
  font-size: 1rem;
  font-weight: 700;
  padding: 0.45rem 0.6rem;
  text-align: center;
  transition: border-color 0.2s;
  font-family: inherit;
}

.score-input:focus {
  outline: none;
  border-color: var(--team-color);
  box-shadow: 0 0 0 3px color-mix(in srgb, var(--team-color) 15%, transparent);
}

.score-input::-webkit-inner-spin-button,
.score-input::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

.save-btn {
  width: 100%;
  padding: 0.85rem 1.5rem;
  background: linear-gradient(135deg, #6366f1, #8b5cf6);
  border: none;
  border-radius: 14px;
  color: white;
  font-size: 0.95rem;
  font-weight: 700;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.6rem;
  letter-spacing: 0.3px;
  transition: all 0.2s;
  font-family: inherit;
}

.save-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(99, 102, 241, 0.4);
}

.save-btn:active {
  transform: translateY(0);
}

.saved-toast {
  position: absolute;
  bottom: 1.5rem;
  left: 50%;
  transform: translateX(-50%) translateY(20px);
  background: rgba(34, 197, 94, 0.15);
  border: 1px solid rgba(34, 197, 94, 0.4);
  color: #4ade80;
  padding: 0.6rem 1.2rem;
  border-radius: 99px;
  font-size: 0.85rem;
  font-weight: 600;
  opacity: 0;
  transition: all 0.3s ease;
  pointer-events: none;
  white-space: nowrap;
}

.saved-toast.visible {
  opacity: 1;
  transform: translateX(-50%) translateY(0);
}
</style>
