<script setup>
import { ref, onMounted, computed } from 'vue'
import Home from './views/Home.vue'
import Catalog from './views/Catalog.vue'
import ProductDetail from './views/ProductDetail.vue'
import Login from './views/Login.vue'
import Admin from './views/Admin.vue'

const pantallaActiva = ref('home')
const productoIdSeleccionado = ref(null)
const usuarioLogeado = ref(null)

onMounted(async () => {
  try {
    const res = await fetch('/api/auth/session')
    if (res.ok) {
      const data = await res.json()
      if (data && data.user) {
        usuarioLogeado.value = {
          id: data.user.id,
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

// Chatbot Asistente IA Global
const chatAbierto = ref(false)
const mensajes = ref([
  { tipo: 'ia', texto: '¡Hola! Soy el asistente de AutoParts WebHub. ¿En qué puedo ayudarte hoy?' }
])
const nuevoMensaje = ref('')
const enviandoChat = ref(false)

async function enviarMensaje() {
  if (!nuevoMensaje.value.trim() || enviandoChat.value) return
  
  const textoUsuario = nuevoMensaje.value.trim()
  mensajes.value.push({ tipo: 'user', texto: textoUsuario })
  nuevoMensaje.value = ''
  enviandoChat.value = true

  try {
    const historyPayload = mensajes.value.slice(0, -1)
      .filter(m => m.texto && m.texto.trim())
      .map(m => ({
        role: m.tipo === 'ia' ? 'model' : 'user',
        parts: [{ text: m.texto }]
      }))

    const activePartId = pantallaActiva.value === 'detail' ? productoIdSeleccionado.value : null;

    const res = await fetch('/api/chat', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        message: textoUsuario,
        history: historyPayload,
        currentPartId: activePartId
      })
    })

    if (res.ok) {
      const data = await res.json()
      mensajes.value.push({ tipo: 'ia', texto: data.reply })
    } else {
      mensajes.value.push({ 
        tipo: 'ia', 
        texto: 'Lo siento, en este momento el servicio del asistente está experimentando una alta demanda. Por favor, intenta preguntar de nuevo en unos segundos.' 
      })
    }
  } catch (err) {
    console.error('Error al enviar chat:', err)
    mensajes.value.push({ tipo: 'ia', texto: 'Error de red al conectar con el asistente.' })
  } finally {
    enviandoChat.value = false
    setTimeout(() => {
      const chatBody = document.querySelector('.global-chat-body')
      if (chatBody) chatBody.scrollTop = chatBody.scrollHeight
    }, 100)
  }
}

// Carrito de Reservas
const carrito = ref([])
const cartAbierto = ref(false)
const fechaReservaCart = ref('')
const horaReservaCart = ref('')
const reservandoCart = ref(false)
const mensajeReservaCart = ref('')

const toastMensaje = ref('')
const mostrarToast = ref(false)

function mostrarNotificacion(msg) {
  toastMensaje.value = msg
  mostrarToast.value = true
  setTimeout(() => {
    mostrarToast.value = false
  }, 2500)
}

const totalItems = computed(() => carrito.value.reduce((sum, item) => sum + item.cantidad, 0))

function agregarAlCarrito(producto) {
  const itemExistente = carrito.value.find(i => i.producto.id === producto.id)
  
  if (itemExistente) {
    if (itemExistente.cantidad < producto.available_stock) {
      itemExistente.cantidad++
      mostrarNotificacion(`¡Cantidad de ${producto.name} aumentada en el carrito! 🛒`)
    } else {
      alert(`No puedes agregar más de este producto. El stock disponible es de ${producto.available_stock} unidades.`)
    }
  } else {
    if (producto.available_stock > 0) {
      carrito.value.push({
        producto,
        cantidad: 1
      })
      mostrarNotificacion(`¡${producto.name} agregado al carrito! 🛒`)
    } else {
      alert('Este repuesto no tiene stock disponible.')
    }
  }
}

function eliminarDelCarrito(partId) {
  carrito.value = carrito.value.filter(i => i.producto.id !== partId)
}

function actualizarCantidadCarrito(partId, cant) {
  const item = carrito.value.find(i => i.producto.id === partId)
  if (item) {
    const nuevaCant = parseInt(cant, 10)
    if (nuevaCant > item.producto.available_stock) {
      item.cantidad = item.producto.available_stock
      alert(`Cantidad limitada al stock disponible (${item.producto.available_stock} unidades).`)
    } else if (nuevaCant < 1) {
      item.cantidad = 1
    } else {
      item.cantidad = nuevaCant
    }
  }
}

function vaciarCarrito() {
  carrito.value = []
}

async function realizarReservaCart() {
  if (!fechaReservaCart.value || !horaReservaCart.value || reservandoCart.value) return
  
  reservandoCart.value = true
  mensajeReservaCart.value = ''

  try {
    const payloadItems = carrito.value.map(i => ({
      part_id: i.producto.id,
      quantity: i.cantidad
    }))

    const res = await fetch('/api/appointments', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        user_id: usuarioLogeado.value?.id || null,
        appointment_date: fechaReservaCart.value,
        appointment_time: horaReservaCart.value,
        status: 'pending',
        created_by_ia: false,
        items: payloadItems
      })
    })

    const data = await res.json()
    if (res.ok) {
      mensajeReservaCart.value = `¡Cita agendada con éxito! Cita ID: ${data.id}. Te esperamos el ${fechaReservaCart.value} a las ${horaReservaCart.value}.`
      fechaReservaCart.value = ''
      horaReservaCart.value = ''
      setTimeout(() => {
        vaciarCarrito()
        cartAbierto.value = false
        mensajeReservaCart.value = ''
        irA('catalog')
      }, 3000)
    } else {
      mensajeReservaCart.value = `Error al agendar: ${data.error || data.message || 'Slot no disponible o fuera de horario (09:00 - 17:30)'}`
    }
  } catch (err) {
    console.error('Error al agendar cita de carrito:', err)
    mensajeReservaCart.value = 'Error al conectar con el servidor para agendar.'
  } finally {
    reservandoCart.value = false
  }
}
</script>

<template>
  <header class="app-header">
    <div class="header-container">
      <div class="logo" @click="irA('home')">
        <span class="logo-icon">🚗</span>
        <span class="logo-text">AutoParts <span class="accent-text">WebHub</span></span>
      </div>
      <nav class="nav-links">
        <button :class="{ active: pantallaActiva === 'home' }" @click="irA('home')">
          Inicio
        </button>
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

  <Home v-if="pantallaActiva === 'home'" @explorar="irA('catalog')" @login="irA('login')" />
  <Catalog v-else-if="pantallaActiva === 'catalog'" @ver-detalle="(id) => irA('detail', id)" />
  <ProductDetail v-else-if="pantallaActiva === 'detail'" :partId="productoIdSeleccionado || 1" :user="usuarioLogeado" @volver="irA('catalog')" @ir-a-login="irA('login')" @agregar-carrito="(p) => agregarAlCarrito(p)" />
  <Login v-else-if="pantallaActiva === 'login'" @login="(u) => { usuarioLogeado = u; irA(u.role === 'client' ? 'catalog' : 'admin') }" @volver="irA('catalog')" />
  <Admin v-else-if="pantallaActiva === 'admin' && usuarioLogeado && (usuarioLogeado.role === 'admin' || usuarioLogeado.role === 'mechanic')" :user="usuarioLogeado" @logout="usuarioLogeado = null; irA('catalog')" />

  <!-- Floating Cart Button -->
  <button v-if="pantallaActiva !== 'admin'" class="cart-float-btn" @click="cartAbierto = !cartAbierto">
    <span class="cart-icon">🛒</span>
    <span class="cart-badge" :key="totalItems" v-if="totalItems > 0">{{ totalItems }}</span>
  </button>

  <!-- Cart Modal -->
  <div v-if="cartAbierto" class="cart-modal-overlay" @click.self="cartAbierto = false">
    <div class="cart-modal-card">
      <div class="cart-modal-header">
        <h3 style="margin: 0; display: flex; align-items: center; gap: 8px;">🛒 Mi Carrito de Reservas</h3>
        <button class="close-btn" @click="cartAbierto = false">&times;</button>
      </div>

      <div class="cart-modal-body">
        <div v-if="carrito.length === 0" class="empty-cart" style="text-align: center; padding: 40px 20px;">
          <div style="font-size: 3rem; margin-bottom: 12px;">🛒</div>
          <p style="font-weight: 600; font-size: 1.1rem; margin: 0;">Tu carrito está vacío</p>
          <p style="font-size: 0.88rem; color: var(--text-light); margin-top: 5px;">Agrega repuestos desde el catálogo.</p>
        </div>
        <div v-else class="cart-items-list" style="display: flex; flex-direction: column; gap: 15px;">
          <div v-for="item in carrito" :key="item.producto.id" class="cart-item-row" style="display: flex; justify-content: space-between; align-items: center; padding: 12px; background: var(--bg); border-radius: 10px; border: 1px solid var(--border);">
            <div class="item-info" style="display: flex; flex-direction: column; gap: 4px; flex: 1; padding-right: 15px;">
              <span class="item-name" style="font-weight: 600; font-size: 0.92rem; color: var(--text);">{{ item.producto.name }}</span>
              <span class="item-sku" style="font-size: 0.78rem; color: var(--text-light);">SKU: {{ item.producto.sku }}</span>
            </div>
            <div class="item-actions" style="display: flex; align-items: center; gap: 12px;">
              <div class="quantity-picker" style="display: flex; align-items: center; border: 1px solid var(--border); border-radius: 6px; overflow: hidden; background: var(--surface);">
                <button @click="actualizarCantidadCarrito(item.producto.id, item.cantidad - 1)" style="padding: 4px 10px; border: none; background: none; cursor: pointer; font-weight: bold; color: var(--text-light);">-</button>
                <input type="number" :value="item.cantidad" readonly style="width: 35px; text-align: center; border: none; background: none; font-size: 0.9rem; font-weight: 600; color: var(--text);">
                <button @click="actualizarCantidadCarrito(item.producto.id, item.cantidad + 1)" style="padding: 4px 10px; border: none; background: none; cursor: pointer; font-weight: bold; color: var(--text-light);">+</button>
              </div>
              <button class="btn-remove" @click="eliminarDelCarrito(item.producto.id)" style="background: none; border: none; cursor: pointer; font-size: 1.1rem; padding: 4px;">🗑️</button>
            </div>
          </div>

          <!-- Reservation Form -->
          <div class="reserva-form" style="margin-top: 15px; padding: 15px; background: var(--bg); border-radius: 12px; border: 1px solid var(--border);">
            <h4 style="margin-top: 0; margin-bottom: 12px;">Agendar Cita de Retiro</h4>
            <div style="display: flex; gap: 15px; flex-wrap: wrap;">
              <div style="flex: 1; min-width: 120px;">
                <label style="display: block; font-size: 0.82rem; color: var(--text-light); margin-bottom: 5px;">Fecha:</label>
                <input type="date" v-model="fechaReservaCart" style="width: 100%; padding: 8px; border-radius: 6px; border: 1px solid var(--border); background: var(--surface); color: var(--text);" required>
              </div>
              <div style="flex: 1; min-width: 120px;">
                <label style="display: block; font-size: 0.82rem; color: var(--text-light); margin-bottom: 5px;">Hora (09:00 - 17:30):</label>
                <input type="time" v-model="horaReservaCart" style="width: 100%; padding: 8px; border-radius: 6px; border: 1px solid var(--border); background: var(--surface); color: var(--text);" required>
              </div>
            </div>

            <div style="margin-top: 15px;">
              <button class="btn" style="width: 100%; padding: 10px;" :disabled="reservandoCart" @click="realizarReservaCart">
                {{ reservandoCart ? 'Agendando...' : 'Confirmar Reserva de Retiro' }}
              </button>
            </div>
            <p v-if="mensajeReservaCart" class="mensaje-reserva" style="margin-top: 10px; font-weight: 500; font-size: 0.9rem; text-align: center; color: var(--primary);">{{ mensajeReservaCart }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Floating Chatbot Widget -->
  <div v-if="pantallaActiva !== 'admin'" class="global-chatbot">
    <!-- Trigger Button -->
    <button v-if="!chatAbierto" class="chat-trigger" @click="chatAbierto = true">
      <span class="chat-trigger-icon">💬</span>
      <span class="chat-trigger-text">Asistente AutoParts</span>
    </button>

    <!-- Chat Panel -->
    <div v-if="chatAbierto" class="chat-panel">
      <div class="global-chat-head">
        <span>Asistente AutoParts</span>
        <button class="chat-close-btn" @click="chatAbierto = false">&times;</button>
      </div>

      <!-- If Logged In -->
      <div v-if="usuarioLogeado" style="display: flex; flex-direction: column; flex: 1; overflow: hidden;">
        <div class="global-chat-body">
          <div v-for="(m, i) in mensajes" :key="i" class="global-chat-msg" :class="m.tipo">
            <div class="msg-bubble">{{ m.texto }}</div>
          </div>
        </div>
        <div class="global-chat-input">
          <input 
            type="text" 
            v-model="nuevoMensaje" 
            placeholder="Pregunta compatibilidad, stock..." 
            @keyup.enter="enviarMensaje"
            :disabled="enviandoChat"
          >
          <button @click="enviarMensaje" :disabled="enviandoChat">
            <span v-if="enviandoChat">...</span>
            <span v-else>✈</span>
          </button>
        </div>
      </div>

      <!-- If Not Logged In (Locked Screen) -->
      <div v-else class="global-chat-locked">
        <div style="font-size: 2.2rem; margin-bottom: 12px;">🔒</div>
        <p style="font-size: 0.88rem; color: var(--text-light); margin-bottom: 16px; font-weight: 500; line-height: 1.4; padding: 0 15px;">
          Inicia sesión para interactuar con nuestro Asistente de IA y recibir ayuda personalizada.
        </p>
        <button class="btn btn-sm" @click="irA('login'); chatAbierto = false" style="padding: 8px 16px; font-size: 0.85rem;">
          Iniciar Sesión
        </button>
      </div>
    </div>
  </div>
  <!-- Toast Notification -->
  <div v-if="mostrarToast" class="cart-toast">
    {{ toastMensaje }}
  </div>
</template>