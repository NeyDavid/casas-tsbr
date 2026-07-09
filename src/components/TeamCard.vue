<template>
  <div
    class="team-card"
    :class="{ 'is-leader': isLeader }"
    :style="{ '--team-color': team.primaryColor, '--team-secondary': team.secondaryColor }"
  >
    <div class="rank-badge" v-if="isLeader">
      <span>👑</span>
    </div>
    <div class="position-number">{{ position }}º</div>

    <div class="logo-wrapper">
      <div class="logo-glow" :style="{ background: team.primaryColor }"></div>
      <img :src="team.logo" :alt="team.name" class="team-logo" />
    </div>

    <div class="team-info">
      <h3 class="team-name">{{ team.name }}</h3>
      <div class="score-display">
        <span class="score-value">{{ team.score.toLocaleString('pt-BR') }}</span>
        <span class="score-label">pontos</span>
      </div>
    </div>

    <div class="progress-bar-container">
      <div
        class="progress-bar-fill"
        :style="{
          width: maxScore > 0 ? `${(team.score / maxScore) * 100}%` : '0%',
          background: `linear-gradient(90deg, ${team.primaryColor}, ${team.gradient[1]})`,
        }"
      ></div>
    </div>
  </div>
</template>

<script setup>
defineProps({
  team: { type: Object, required: true },
  position: { type: Number, required: true },
  isLeader: { type: Boolean, default: false },
  maxScore: { type: Number, default: 1 },
})
</script>

<style scoped>
.team-card {
  position: relative;
  background: linear-gradient(135deg, #1a202c 0%, #2d3748 100%);
  border: 1px solid rgba(255, 255, 255, 0.06);
  border-radius: 20px;
  padding: 1.5rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  transition: transform 0.3s ease, box-shadow 0.3s ease, border-color 0.3s ease;
  overflow: hidden;
}

.team-card::before {
  content: '';
  position: absolute;
  inset: 0;
  border-radius: 20px;
  border: 2px solid transparent;
  background: linear-gradient(135deg, var(--team-color, #fff), transparent 60%) border-box;
  -webkit-mask: linear-gradient(#fff 0 0) padding-box, linear-gradient(#fff 0 0);
  -webkit-mask-composite: destination-out;
  mask-composite: exclude;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.team-card:hover::before,
.team-card.is-leader::before {
  opacity: 1;
}

.team-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.4), 0 0 30px rgba(var(--team-color-rgb, 255, 255, 255), 0.1);
}

.team-card.is-leader {
  background: linear-gradient(135deg, #1a202c 0%, #2d3748 80%, color-mix(in srgb, var(--team-color) 15%, #2d3748) 100%);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3), 0 0 40px color-mix(in srgb, var(--team-color) 20%, transparent);
}

.rank-badge {
  position: absolute;
  top: 12px;
  right: 12px;
  font-size: 1.4rem;
  animation: float 2s ease-in-out infinite;
}

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-4px); }
}

.position-number {
  position: absolute;
  top: 14px;
  left: 16px;
  font-size: 0.85rem;
  font-weight: 800;
  color: rgba(255, 255, 255, 0.3);
  letter-spacing: 0.5px;
}

.logo-wrapper {
  position: relative;
  width: 100px;
  height: 100px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.logo-glow {
  position: absolute;
  inset: 0;
  border-radius: 50%;
  opacity: 0.15;
  filter: blur(16px);
}

.team-logo {
  width: 90px;
  height: 90px;
  object-fit: contain;
  position: relative;
  z-index: 1;
  filter: drop-shadow(0 4px 12px rgba(0, 0, 0, 0.5));
  transition: transform 0.3s ease;
}

.team-card:hover .team-logo {
  transform: scale(1.08);
}

.team-info {
  text-align: center;
}

.team-name {
  font-size: 1.2rem;
  font-weight: 800;
  color: #e2e8f0;
  margin: 0 0 0.4rem;
  letter-spacing: 0.5px;
  text-transform: uppercase;
}

.score-display {
  display: flex;
  align-items: baseline;
  gap: 0.3rem;
  justify-content: center;
}

.score-value {
  font-size: 2rem;
  font-weight: 900;
  color: var(--team-color, #fff);
  line-height: 1;
  transition: all 0.5s ease;
  text-shadow: 0 0 20px color-mix(in srgb, var(--team-color) 50%, transparent);
}

.score-label {
  font-size: 0.75rem;
  color: #718096;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.progress-bar-container {
  width: 100%;
  height: 6px;
  background: rgba(255, 255, 255, 0.07);
  border-radius: 99px;
  overflow: hidden;
}

.progress-bar-fill {
  height: 100%;
  border-radius: 99px;
  transition: width 0.8s cubic-bezier(0.4, 0, 0.2, 1);
}
</style>
