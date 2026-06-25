<script setup>
import { ref, onMounted } from 'vue'
import Catalog from './views/Catalog.vue'
import ProductDetail from './views/ProductDetail.vue'
import Login from './views/Login.vue'
import Admin from './views/Admin.vue'

const pantallaActiva = ref('catalog')
const productoIdSeleccionado = ref(null)
const usuarioLogeado = ref(null)

onMounted(async () => {
  try {
    const res = await fetch('/api/auth/session')
    if (res.ok) {
      const data = await res.json()
      if (data && data.user) {
        usuarioLogeado.value = {
          email: data.user.email,
          name: data.user.name,
          role: data.user.role || 'client'
        }
      }
    }
  } catch (err) {
    console.error('Error al recuperar sesión activa:', err)
  }
})

function irA(pantalla, prodId = null) {
  if (pantalla === 'admin' && !usuarioLogeado.value) {
    pantallaActiva.value = 'login'
  } else {
    pantallaActiva.value = pantalla
  }
  if (prodId !== null) {
    productoIdSeleccionado.value = prodId
  }
  window.scrollTo(0, 0)
}

async function cerrarSesion() {
  usuarioLogeado.value = null
  try {
    const res = await fetch('/api/auth/csrf');
    if (!res.ok) throw new Error('Error al obtener el token CSRF');
    const { csrfToken } = await res.json();
    
    const form = document.createElement('form');
    form.method = 'POST';
    form.action = '/api/auth/signout';
    
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
    console.error('Error al cerrar sesión:', err);
    window.location.href = `/api/auth/signout?callbackUrl=${encodeURIComponent(window.location.origin)}`;
  }
}
</script>

<template>
  <header class="app-header">
    <div class="header-container">
      <div class="logo" @click="irA('catalog')">
        <span class="logo-icon">🚗</span>
        <span class="logo-text">AutoParts <span class="accent-text">WebHub</span></span>
      </div>
      <nav class="nav-links">
        <button :class="{ active: pantallaActiva === 'catalog' }" @click="irA('catalog')">
          Catálogo
        </button>
        <button v-if="productoIdSeleccionado" :class="{ active: pantallaActiva === 'detail' }" @click="irA('detail')">
          Detalle de Repuesto
        </button>
        <button v-if="usuarioLogeado && (usuarioLogeado.role === 'admin' || usuarioLogeado.role === 'mechanic')" :class="{ active: pantallaActiva === 'admin' }" @click="irA('admin')">
          Panel de Operaciones
        </button>
      </nav>
      <div class="auth-section">
        <div v-if="usuarioLogeado" class="user-menu">
          <span class="user-email">👤 {{ usuarioLogeado.email }}</span>
          <button class="btn btn-outline logout-btn" @click="cerrarSesion">
            Cerrar Sesión
          </button>
        </div>
        <button v-else class="btn login-btn" @click="irA('login')">
          Iniciar Sesión
        </button>
      </div>
    </div>
  </header>

  <Catalog v-if="pantallaActiva === 'catalog'" @ver-detalle="(id) => irA('detail', id)" />
  <ProductDetail v-else-if="pantallaActiva === 'detail'" :partId="productoIdSeleccionado || 1" :user="usuarioLogeado" @volver="irA('catalog')" @ir-a-login="irA('login')" />
  <Login v-else-if="pantallaActiva === 'login'" @login="(u) => { usuarioLogeado = u; irA(u.role === 'client' ? 'catalog' : 'admin') }" @volver="irA('catalog')" />
  <Admin v-else-if="pantallaActiva === 'admin' && usuarioLogeado && (usuarioLogeado.role === 'admin' || usuarioLogeado.role === 'mechanic')" :user="usuarioLogeado" @logout="usuarioLogeado = null; irA('catalog')" />
</template>