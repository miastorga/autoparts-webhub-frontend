<script setup>
import { ref } from 'vue'

const emit = defineEmits(['login', 'volver'])

const modo = ref('login') // 'login' o 'registro'

const email = ref('')
const password = ref('')
const name = ref('')
const confirmPassword = ref('')

const error = ref('')
const exitoMsg = ref('')
const cargando = ref(false)

function cambiarModo(nuevoModo) {
  modo.value = nuevoModo
  error.value = ''
  exitoMsg.value = ''
  email.value = ''
  password.value = ''
  name.value = ''
  confirmPassword.value = ''
}

async function procesarFormulario() {
  error.value = ''
  exitoMsg.value = ''
  
  if (modo.value === 'login') {
    if (!email.value || !password.value) {
      error.value = 'Por favor, completa todos los campos.'
      return
    }
    
    cargando.value = true
    setTimeout(() => {
      cargando.value = false
      const role = email.value.toLowerCase().includes('admin') 
        ? 'admin' 
        : (email.value.toLowerCase().includes('mecanico') || email.value.toLowerCase().includes('mechanic') ? 'mechanic' : 'client')
      emit('login', { email: email.value, role })
    }, 800)
  } else {
    if (!name.value || !email.value || !password.value || !confirmPassword.value) {
      error.value = 'Por favor, completa todos los campos.'
      return
    }
    if (password.value !== confirmPassword.value) {
      error.value = 'Las contraseñas no coinciden.'
      return
    }
    
    cargando.value = true
    try {
      const res = await fetch('/api/users/register', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          name: name.value,
          email: email.value,
          password_hash: password.value
        })
      })
      
      const data = await res.json()
      if (res.ok) {
        exitoMsg.value = '¡Cuenta creada con éxito! Ahora puedes iniciar sesión.'
        setTimeout(() => {
          cambiarModo('login')
        }, 1500)
      } else {
        error.value = data.error || 'Ocurrió un error al registrar la cuenta.'
      }
    } catch (err) {
      console.error('Error al registrar usuario:', err)
      error.value = 'Error de red al intentar registrar.'
    } finally {
      cargando.value = false
    }
  }
}

async function iniciarConGoogle() {
  try {
    const res = await fetch('/api/auth/csrf');
    if (!res.ok) throw new Error('Error al obtener el token CSRF');
    const { csrfToken } = await res.json();
    
    const form = document.createElement('form');
    form.method = 'POST';
    form.action = '/api/auth/signin/google';
    
    const csrfInput = document.createElement('input');
    csrfInput.type = 'hidden';
    csrfInput.name = 'csrfToken';
    csrfInput.value = csrfToken;
    form.appendChild(csrfInput);
    
    const callbackInput = document.createElement('input');
    callbackInput.type = 'hidden';
    callbackInput.name = 'callbackUrl';
    callbackInput.value = window.location.origin;
    form.appendChild(callbackInput);
    
    document.body.appendChild(form);
    form.submit();
  } catch (err) {
    console.error('Error al iniciar sesión con Google:', err);
    error.value = 'No se pudo iniciar la sesión con Google.';
  }
}
</script>

<template>
  <div class="screen login-container">
    <button class="btn btn-outline back-btn" @click="$emit('volver')">
      ← Volver al catálogo
    </button>
    
    <div class="login-card">
      <div class="card-header">
        <span class="logo-icon">🚗</span>
        <h3>{{ modo === 'login' ? 'Iniciar Sesión' : 'Registrarse' }}</h3>
        <p>{{ modo === 'login' ? 'Ingresa tus credenciales para acceder' : 'Completa los datos para crear tu cuenta' }}</p>
      </div>
      
      <form @submit.prevent="procesarFormulario">
        <div v-if="modo === 'registro'" class="input-group">
          <label for="name">Nombre Completo</label>
          <input 
            type="text" 
            id="name" 
            v-model="name" 
            placeholder="Juan Pérez"
            required
          >
        </div>

        <div class="input-group">
          <label for="email">Correo Electrónico</label>
          <input 
            type="email" 
            id="email" 
            v-model="email" 
            placeholder="correo@ejemplo.com"
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

        <div v-if="modo === 'registro'" class="input-group">
          <label for="confirmPassword">Confirmar Contraseña</label>
          <input 
            type="password" 
            id="confirmPassword" 
            v-model="confirmPassword" 
            placeholder="••••••••"
            required
          >
        </div>
        
        <p v-if="error" class="error-msg">{{ error }}</p>
        <p v-if="exitoMsg" class="success-msg">{{ exitoMsg }}</p>
        
        <button class="btn login-btn" type="submit" :disabled="cargando">
          {{ cargando ? 'Procesando...' : (modo === 'login' ? 'Iniciar Sesión' : 'Registrarse') }}
        </button>

        <div v-if="modo === 'login'">
          <div class="divider"><span>o</span></div>

          <button class="btn btn-outline google-btn" type="button" @click="iniciarConGoogle">
            <svg class="google-icon" viewBox="0 0 24 24" width="18" height="18" xmlns="http://www.w3.org/2000/svg">
              <path d="M22.56 12.25c0-.78-.07-1.53-.2-2.25H12v4.26h5.92c-.26 1.37-1.04 2.53-2.21 3.31v2.77h3.57c2.08-1.92 3.28-4.74 3.28-8.09z" fill="#4285F4"/>
              <path d="M12 23c2.97 0 5.46-.98 7.28-2.66l-3.57-2.77c-.98.66-2.23 1.06-3.71 1.06-2.86 0-5.29-1.93-6.16-4.53H2.18v2.84C3.99 20.53 7.7 23 12 23z" fill="#34A853"/>
              <path d="M5.84 14.09c-.22-.66-.35-1.36-.35-2.09s.13-1.43.35-2.09V7.06H2.18C1.43 8.55 1 10.22 1 12s.43 3.45 1.18 4.94l2.85-2.22.81-.63z" fill="#FBBC05"/>
              <path d="M12 5.38c1.62 0 3.06.56 4.21 1.64l3.15-3.15C17.45 2.09 14.97 1 12 1 7.7 1 3.99 3.47 2.18 7.06l3.66 2.84c.87-2.6 3.3-4.52 6.16-4.52z" fill="#EA4335"/>
            </svg>
            Iniciar sesión con Google
          </button>
        </div>

        <div class="toggle-mode">
          <a v-if="modo === 'login'" href="#" @click.prevent="cambiarModo('registro')">
            ¿No tienes una cuenta? Regístrate aquí
          </a>
          <a v-else href="#" @click.prevent="cambiarModo('login')">
            ¿Ya tienes una cuenta? Inicia sesión aquí
          </a>
        </div>
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
.success-msg { color: #22c55e; font-size: 0.85rem; font-weight: 500; margin-bottom: 15px; text-align: left; }
.login-btn { width: 100%; padding: 14px; font-size: 1rem; font-weight: 600; margin-top: 10px; }

.toggle-mode { text-align: center; margin-top: 20px; }
.toggle-mode a { font-size: 0.88rem; color: var(--accent); text-decoration: none; font-weight: 500; }
.toggle-mode a:hover { text-decoration: underline; }

.divider {
  display: flex;
  align-items: center;
  text-align: center;
  margin: 20px 0;
  color: var(--text-light);
  font-size: 0.85rem;
}
.divider::before, .divider::after {
  content: '';
  flex: 1;
  border-bottom: 1px solid var(--border);
}
.divider:not(:empty)::before {
  margin-right: .25em;
}
.divider:not(:empty)::after {
  margin-left: .25em;
}

.google-btn {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  background: white;
  color: #374151;
  border: 1px solid var(--border);
  padding: 12px;
  font-weight: 500;
  font-size: 0.95rem;
}
.google-btn:hover {
  background: #f9fafb;
  transform: translateY(-1px);
}
.google-icon {
  display: inline-block;
}
</style>