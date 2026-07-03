<script setup>
defineEmits(['explorar', 'login'])
</script>

<template>
  <div class="home-container">
    <!-- Animated background glowing shapes -->
    <div class="glow-orb orb-1"></div>
    <div class="glow-orb orb-2"></div>
    <div class="glow-orb orb-3"></div>

    <section class="hero-section">
      <div class="hero-badge animate-fade-in">
        <span class="pulse-dot"></span>
        Logística Automotriz Inteligente
      </div>
      
      <h1 class="hero-title animate-slide-up">
        Encuentra y Reserva tus <br>
        <span class="gradient-text">Repuestos Premium</span>
      </h1>
      
      <p class="hero-subtitle animate-slide-up-delay-1">
        La plataforma moderna de gestión de repuestos y retiro express. Reserva tu repuesto en segundos, recibe asistencia inteligente con IA y retira directo en bodega.
      </p>

      <div class="hero-actions animate-slide-up-delay-2">
        <button class="btn btn-primary btn-glow" style="padding: 14px 28px; font-size: 1.05rem;" @click="$emit('explorar')">
          Explorar Catálogo 🚀
        </button>
        <button class="btn btn-outline" style="padding: 14px 28px; font-size: 1.05rem;" @click="$emit('login')">
          Acceder al Panel 👤
        </button>
      </div>
    </section>

    <!-- Stats & Key Pillars Section -->
    <section class="pillars-section animate-fade-in-delayed">
      <div class="pillar-card">
        <div class="pillar-icon-wrapper">
          <div class="css-icon gear-icon">
            <span></span>
          </div>
        </div>
        <h3>Catálogo de Alta Gama</h3>
        <p>Acceso inmediato a pastillas de freno, amortiguadores y repuestos de marcas líderes con stock en tiempo real.</p>
      </div>

      <div class="pillar-card">
        <div class="pillar-icon-wrapper">
          <div class="css-icon calendar-icon">
            <span></span>
          </div>
        </div>
        <h3>Citas de Retiro Express</h3>
        <p>Selecciona tus productos, agenda una hora conveniente de retiro en mostrador y evita molestas esperas.</p>
      </div>

      <div class="pillar-card">
        <div class="pillar-icon-wrapper">
          <div class="css-icon robot-icon">
            <span></span>
          </div>
        </div>
        <h3>Asistente IA Integrado</h3>
        <p>Pregunta a nuestro chatbot impulsado por IA sobre la compatibilidad de piezas o haz que agende por ti.</p>
      </div>
    </section>

    <!-- Info Banner Grid -->
    <section class="info-banner animate-fade-in-delayed">
      <div class="banner-content">
        <span class="banner-tag">Bodega Física</span>
        <h2>Ubicaciones y Bodegas Transparentes</h2>
        <p>
          Cada repuesto cuenta con su ubicación asignada en bodega física (ej: pasillo A-12, estantería B). Esto permite a nuestros mecánicos preparar tu pedido antes de que llegues a la sucursal.
        </p>
      </div>
      <div class="banner-interactive-card">
        <div class="map-card-inner">
          <div class="radar-circle"></div>
          <div class="radar-center"></div>
          <div class="location-dot dot-1"></div>
          <div class="location-dot dot-2"></div>
          <span style="font-weight: 600; font-size: 0.95rem; color: white; z-index: 2;">Sucursal Principal AutoParts</span>
          <span style="font-size: 0.8rem; color: #a1a1aa; z-index: 2; margin-top: 4px;">Atención Express: 09:00 a 17:30 hrs</span>
        </div>
      </div>
    </section>
  </div>
</template>

<style scoped>
.home-container {
  position: relative;
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 60px 20px;
  overflow: hidden;
  min-height: 80vh;
}

/* Background Glowing Orbs */
.glow-orb {
  position: absolute;
  border-radius: 50%;
  filter: blur(100px);
  opacity: 0.12;
  z-index: -1;
  pointer-events: none;
}
.orb-1 {
  width: 350px;
  height: 350px;
  background: var(--accent);
  top: -50px;
  right: 10%;
  animation: orbMove1 20s infinite alternate ease-in-out;
}
.orb-2 {
  width: 400px;
  height: 400px;
  background: #1e3a8a;
  bottom: 50px;
  left: -50px;
  animation: orbMove2 25s infinite alternate ease-in-out;
}
.orb-3 {
  width: 250px;
  height: 250px;
  background: #3b82f6;
  top: 40%;
  left: 30%;
  animation: orbMove3 15s infinite alternate ease-in-out;
}

@keyframes orbMove1 {
  0% { transform: translate(0, 0) scale(1); }
  100% { transform: translate(60px, 40px) scale(1.1); }
}
@keyframes orbMove2 {
  0% { transform: translate(0, 0) scale(1); }
  100% { transform: translate(-40px, -60px) scale(1.2); }
}
@keyframes orbMove3 {
  0% { transform: translate(0, 0) scale(1); }
  100% { transform: translate(30px, -30px) scale(0.9); }
}

/* Hero Section */
.hero-section {
  text-align: center;
  margin-bottom: 80px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.hero-badge {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  background: rgba(59, 130, 246, 0.1);
  border: 1px solid rgba(59, 130, 246, 0.25);
  color: var(--accent);
  padding: 6px 14px;
  border-radius: 20px;
  font-size: 0.85rem;
  font-weight: 600;
  margin-bottom: 24px;
}

.pulse-dot {
  width: 8px;
  height: 8px;
  background-color: var(--accent);
  border-radius: 50%;
  display: inline-block;
  box-shadow: 0 0 0 0 rgba(59, 130, 246, 0.7);
  animation: pulseDot 1.6s infinite;
}

@keyframes pulseDot {
  0% {
    transform: scale(0.95);
    box-shadow: 0 0 0 0 rgba(59, 130, 246, 0.7);
  }
  70% {
    transform: scale(1);
    box-shadow: 0 0 0 8px rgba(59, 130, 246, 0);
  }
  100% {
    transform: scale(0.95);
    box-shadow: 0 0 0 0 rgba(59, 130, 246, 0);
  }
}

.hero-title {
  font-size: 3.5rem;
  font-weight: 800;
  line-height: 1.15;
  letter-spacing: -0.03em;
  color: var(--text);
  margin-bottom: 20px;
}

.gradient-text {
  background: linear-gradient(135deg, #60a5fa, #3b82f6, #1d4ed8);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.hero-subtitle {
  font-size: 1.2rem;
  color: var(--text-light);
  max-width: 680px;
  line-height: 1.6;
  margin-bottom: 35px;
}

.hero-actions {
  display: flex;
  gap: 16px;
  justify-content: center;
}

.btn-glow {
  box-shadow: 0 4px 20px rgba(59, 130, 246, 0.35);
  transition: box-shadow 0.2s, transform 0.2s;
}
.btn-glow:hover {
  box-shadow: 0 8px 25px rgba(59, 130, 246, 0.55);
}

/* Key Pillars / Cards */
.pillars-section {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 24px;
  margin-bottom: 80px;
}

.pillar-card {
  background: var(--surface);
  border: 1px solid var(--border);
  padding: 35px 30px;
  border-radius: 20px;
  text-align: left;
  box-shadow: var(--shadow);
  transition: transform 0.3s, border-color 0.3s, box-shadow 0.3s;
}

.pillar-card:hover {
  transform: translateY(-8px);
  border-color: rgba(59, 130, 246, 0.3);
  box-shadow: 0 12px 30px rgba(0, 0, 0, 0.08);
}

.pillar-icon-wrapper {
  width: 54px;
  height: 54px;
  border-radius: 12px;
  background: var(--bg);
  border: 1px solid var(--border);
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 24px;
}

.pillar-card h3 {
  font-size: 1.25rem;
  font-weight: 700;
  color: var(--text);
  margin-bottom: 12px;
}

.pillar-card p {
  color: var(--text-light);
  line-height: 1.6;
  font-size: 0.92rem;
  margin: 0;
}

/* CSS Icons (Geometric, No Images) */
.css-icon {
  position: relative;
  width: 26px;
  height: 26px;
}

.gear-icon span {
  display: block;
  width: 100%;
  height: 100%;
  border: 4px dashed var(--accent);
  border-radius: 50%;
  animation: spinGear 8s linear infinite;
}
.gear-icon::before {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 8px;
  height: 8px;
  background: var(--accent);
  border-radius: 50%;
}

@keyframes spinGear {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.calendar-icon span {
  display: block;
  width: 22px;
  height: 22px;
  border: 2px solid var(--accent);
  border-radius: 4px;
  position: relative;
}
.calendar-icon span::after {
  content: '';
  position: absolute;
  top: 4px;
  left: 0;
  width: 100%;
  height: 2px;
  background: var(--accent);
}
.calendar-icon span::before {
  content: '';
  position: absolute;
  top: -6px;
  left: 4px;
  width: 2px;
  height: 6px;
  background: var(--accent);
  border-radius: 1px;
  box-shadow: 10px 0 var(--accent);
}

.robot-icon span {
  display: block;
  width: 24px;
  height: 16px;
  border: 2px solid var(--accent);
  border-radius: 4px;
  margin-top: 6px;
  position: relative;
}
.robot-icon span::before {
  content: '';
  position: absolute;
  top: -6px;
  left: 50%;
  transform: translateX(-50%);
  width: 2px;
  height: 6px;
  background: var(--accent);
}
.robot-icon span::after {
  content: '';
  position: absolute;
  top: 3px;
  left: 4px;
  width: 4px;
  height: 4px;
  background: var(--accent);
  border-radius: 50%;
  box-shadow: 8px 0 var(--accent);
}

/* Info Banner Grid */
.info-banner {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(320px, 1fr));
  gap: 40px;
  align-items: center;
  background: linear-gradient(135deg, rgba(15, 23, 42, 0.6) 0%, rgba(30, 41, 59, 0.4) 100%);
  border: 1px solid var(--border);
  padding: 50px;
  border-radius: 24px;
  backdrop-filter: blur(10px);
}

.banner-content {
  text-align: left;
}

.banner-tag {
  font-size: 0.78rem;
  font-weight: 700;
  color: var(--accent);
  text-transform: uppercase;
  letter-spacing: 0.05em;
  display: block;
  margin-bottom: 12px;
}

.banner-content h2 {
  font-size: 2.2rem;
  font-weight: 800;
  color: white;
  margin-bottom: 18px;
}

.banner-content p {
  color: #94a3b8;
  line-height: 1.7;
  font-size: 0.98rem;
  margin: 0;
}

/* Interactive Radar Animation Card */
.banner-interactive-card {
  display: flex;
  justify-content: center;
}

.map-card-inner {
  width: 100%;
  max-width: 320px;
  height: 200px;
  background: rgba(15, 23, 42, 0.8);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 16px;
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}

.radar-circle {
  position: absolute;
  width: 260px;
  height: 260px;
  border: 1px solid rgba(59, 130, 246, 0.15);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
}
.radar-circle::before {
  content: '';
  width: 160px;
  height: 160px;
  border: 1px solid rgba(59, 130, 246, 0.25);
  border-radius: 50%;
}
.radar-circle::after {
  content: '';
  width: 80px;
  height: 80px;
  border: 1px solid rgba(59, 130, 246, 0.4);
  border-radius: 50%;
}

.radar-center {
  position: absolute;
  width: 140px;
  height: 140px;
  background: conic-gradient(from 0deg, rgba(59, 130, 246, 0.15) 0deg, transparent 90deg);
  border-radius: 50%;
  animation: spinRadar 5s linear infinite;
}

@keyframes spinRadar {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.location-dot {
  position: absolute;
  width: 8px;
  height: 8px;
  background: #3b82f6;
  border-radius: 50%;
  box-shadow: 0 0 10px #3b82f6;
}
.dot-1 {
  top: 40px;
  left: 80px;
  animation: pulseLocation 2s infinite ease-in-out;
}
.dot-2 {
  bottom: 50px;
  right: 70px;
  animation: pulseLocation 2s infinite ease-in-out 1s;
}

@keyframes pulseLocation {
  0% { transform: scale(1); opacity: 0.6; }
  50% { transform: scale(1.5); opacity: 1; box-shadow: 0 0 14px #3b82f6; }
  100% { transform: scale(1); opacity: 0.6; }
}

/* Animations */
.animate-fade-in {
  opacity: 0;
  animation: fadeInEffect 0.8s ease-out forwards;
}
.animate-fade-in-delayed {
  opacity: 0;
  animation: fadeInEffect 0.8s ease-out 0.4s forwards;
}
.animate-slide-up {
  opacity: 0;
  transform: translateY(30px);
  animation: slideUpEffect 0.8s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}
.animate-slide-up-delay-1 {
  opacity: 0;
  transform: translateY(30px);
  animation: slideUpEffect 0.8s cubic-bezier(0.16, 1, 0.3, 1) 0.15s forwards;
}
.animate-slide-up-delay-2 {
  opacity: 0;
  transform: translateY(30px);
  animation: slideUpEffect 0.8s cubic-bezier(0.16, 1, 0.3, 1) 0.3s forwards;
}

@keyframes fadeInEffect {
  to { opacity: 1; }
}
@keyframes slideUpEffect {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* Responsiveness */
@media (max-width: 768px) {
  .hero-title {
    font-size: 2.5rem;
  }
  .info-banner {
    padding: 30px;
  }
}
</style>
