<script setup>
import { ref } from 'vue'

const emit = defineEmits(['login', 'volver'])

const email = ref('')
const password = ref('')
const error = ref('')
const cargando = ref(false)

function iniciarSesion() {
  if (!email.value || !password.value) {
    error.value = 'Por favor, completa todos los campos.'
    return
  }
  
  cargando.value = true
  error.value = ''
  
  // Simular inicio de sesión en desarrollo
  setTimeout(() => {
    cargando.value = false
    const role = email.value.toLowerCase().includes('admin') ? 'admin' : 'mechanic'
    emit('login', { email: email.value, role })
  }, 800)
}
</script>

<template>
  <div class="screen login-container">
    <button class="btn btn-outline back-btn" @click="$emit('volver')">
      ← Volver al catálogo
    </button>
    
    <div class="login-card">
      <div class="card-header">
        <span class="logo-icon">📦</span>
        <h3>Panel Administrativo</h3>
        <p>Ingresa tus credenciales para acceder</p>
      </div>
      
      <form @submit.prevent="iniciarSesion">
        <div class="input-group">
          <label for="email">Correo Electrónico</label>
          <input 
            type="email" 
            id="email" 
            v-model="email" 
            placeholder="admin@autoparts.com"
            required
          >
        </div>
        
        <div class="input-group">
          <label for="password">Contraseña</label>
          <input 
            type="password" 
            id="password" 
            v-model="password" 
            placeholder="••••••••"
            required
          >
        </div>
        
        <p v-if="error" class="error-msg">{{ error }}</p>
        
        <button class="btn login-btn" type="submit" :disabled="cargando">
          {{ cargando ? 'Iniciando sesión...' : 'Iniciar Sesión' }}
        </button>
      </form>
    </div>
  </div>
</template>

<style scoped>
.login-container { display: flex; flex-direction: column; align-items: center; justify-content: center; min-height: 70vh; position: relative; }
.back-btn { position: absolute; top: 0; left: 0; }

.login-card { background: var(--surface); width: 100%; max-width: 400px; padding: 40px; border-radius: 20px; box-shadow: var(--shadow); border: 1px solid var(--border); }
.card-header { text-align: center; margin-bottom: 30px; }
.logo-icon { font-size: 3rem; display: block; margin-bottom: 12px; }
.card-header h3 { font-size: 1.5rem; color: var(--primary); margin-bottom: 6px; }
.card-header p { font-size: 0.88rem; color: var(--text-light); }

.input-group { margin-bottom: 20px; text-align: left; }
.input-group label { display: block; font-size: 0.85rem; color: var(--text-light); margin-bottom: 6px; font-weight: 500; }
.input-group input { width: 100%; padding: 12px 16px; border: 1px solid var(--border); border-radius: 8px; outline: none; background: var(--bg); color: var(--text); font-size: 0.95rem; transition: border-color 0.2s; }
.input-group input:focus { border-color: var(--accent); }

.error-msg { color: #ef4444; font-size: 0.85rem; font-weight: 500; margin-bottom: 15px; text-align: left; }
.login-btn { width: 100%; padding: 14px; font-size: 1rem; font-weight: 600; margin-top: 10px; }
</style>