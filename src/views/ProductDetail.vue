<script setup>
import { ref, onMounted, watch } from 'vue'

const props = defineProps({
  partId: {
    type: Number,
    required: true
  },
  user: {
    type: Object,
    default: null
  }
})

defineEmits(['volver', 'ir-a-login'])

const producto = ref(null)
const cargando = ref(true)

const mensajes = ref([
  { tipo: 'ia', texto: '¡Hola! Soy el asistente de AutoParts WebHub. ¿Tienes dudas sobre la compatibilidad de este repuesto o quieres agendar una cita de retiro?' }
])

const nuevoMensaje = ref('')
const enviandoChat = ref(false)

// Estado para el modal/formulario de reserva directa
const mostrarFormReserva = ref(false)
const fechaReserva = ref('')
const horaReserva = ref('')
const reservando = ref(false)
const mensajeReserva = ref('')

onMounted(async () => {
  await cargarDetalle()
})

watch(() => props.partId, async () => {
  await cargarDetalle()
  // Resetear chat
  mensajes.value = [
    { tipo: 'ia', texto: '¡Hola! Soy el asistente de AutoParts WebHub. ¿Tienes dudas sobre la compatibilidad de este repuesto o quieres agendar una cita de retiro?' }
  ]
  mostrarFormReserva.value = false
  mensajeReserva.value = ''
})

async function cargarDetalle() {
  try {
    cargando.value = true
    const res = await fetch(`/api/parts/${props.partId}`)
    if (res.ok) {
      producto.value = await res.json()
    } else {
      console.error('Error al cargar repuesto:', res.statusText)
    }
  } catch (err) {
    console.error('Error al cargar detalle:', err)
  } finally {
    cargando.value = false
  }
}

async function enviarMensaje() {
  if (!nuevoMensaje.value.trim() || enviandoChat.value) return
  
  const textoUsuario = nuevoMensaje.value.trim()
  mensajes.value.push({ tipo: 'user', texto: textoUsuario })
  nuevoMensaje.value = ''
  enviandoChat.value = true

  try {
    // Convertir el historial de mensajes de la UI al formato que espera Gemini en el Backend
    // Excluimos el último mensaje que acabamos de agregar (se manda en la propiedad 'message')
    const historyPayload = mensajes.value.slice(0, -1).map(m => ({
      role: m.tipo === 'ia' ? 'model' : 'user',
      parts: [{ text: m.texto }]
    }))

    const res = await fetch('/api/chat', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-bypass-auth': 'true' // Para saltar la autenticación de desarrollo en el backend
      },
      body: JSON.stringify({
        message: textoUsuario,
        history: historyPayload
      })
    })

    if (res.ok) {
      const data = await res.json()
      mensajes.value.push({ tipo: 'ia', texto: data.reply })
    } else {
      const data = await res.json().catch(() => ({}))
      mensajes.value.push({ 
        tipo: 'ia', 
        texto: `Lo siento, ocurrió un error al procesar tu solicitud: ${data.message || res.statusText}` 
      })
    }
  } catch (err) {
    console.error('Error al enviar chat:', err)
    mensajes.value.push({ tipo: 'ia', texto: 'Error de red. Asegúrate de que el backend esté encendido.' })
  } finally {
    enviandoChat.value = false
    // Scroll al final del chat
    setTimeout(() => {
      const chatBody = document.querySelector('.chat-body')
      if (chatBody) chatBody.scrollTop = chatBody.scrollHeight
    }, 100)
  }
}

async function realizarReserva() {
  if (!fechaReserva.value || !horaReserva.value || reservando.value) return
  
  reservando.value = true
  mensajeReserva.value = ''

  try {
    const res = await fetch('/api/appointments', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-bypass-auth': 'true' // Bypass para desarrollo
      },
      body: JSON.stringify({
        user_id: 1, // Usuario de prueba
        part_id: props.partId,
        quantity: 1,
        appointment_date: fechaReserva.value,
        appointment_time: horaReserva.value,
        status: 'pending',
        created_by_ia: false
      })
    })

    const data = await res.json()
    if (res.ok) {
      mensajeReserva.value = `¡Cita agendada con éxito! Cita ID: ${data.id}. Te esperamos el ${fechaReserva.value} a las ${horaReserva.value}.`
      fechaReserva.value = ''
      horaReserva.value = ''
      // Opcional: recargar detalle si cambia stock
      await cargarDetalle()
    } else {
      mensajeReserva.value = `Error al agendar: ${data.error || data.message || 'Slot no disponible o fuera de horario (09:00 - 17:30)'}`
    }
  } catch (err) {
    console.error('Error al agendar cita:', err)
    mensajeReserva.value = 'Error al conectar con el servidor para agendar.'
  } finally {
    reservando.value = false
  }
}

function formatoPrecio(valor) {
  return valor ? '$' + valor.toLocaleString('es-CL') : ''
}
</script>

<template>
  <div class="screen">
    <button class="btn btn-outline" style="margin-bottom: 20px;" @click="$emit('volver')">
      ← Volver al catálogo
    </button>

    <div v-if="cargando" class="loading">Cargando detalles de repuesto...</div>

    <div v-else-if="producto" class="detail-view">
      <div class="detail-img">
        <div style="font-size: 3.5rem; margin-bottom: 12px;">⚙️</div>
        <strong>{{ producto.category }}</strong>
      </div>

      <div class="detail-info">
        <span class="tag" :class="{ 'no-stock': producto.available_stock === 0 }">
          En Stock ({{ producto.available_stock }} unidades)
        </span>
        <h2 style="margin-bottom: 8px;">{{ producto.name }}</h2>
        <p class="descripcion">
          Repuesto automotriz de categoría premium. Ubicación en bodega física: <strong>{{ producto.warehouse_location }}</strong>.
        </p>

        <div class="info-box">
          <p style="margin-bottom: 8px;"><strong>SKU:</strong> {{ producto.sku }}</p>
          <p><strong>Categoría:</strong> {{ producto.category }}</p>
        </div>

        <div class="price" style="font-size: 2rem;">{{ formatoPrecio(producto.price) }}</div>
        
        <div v-if="!mostrarFormReserva">
          <button class="btn" style="padding: 14px 30px; font-size: 1.05rem;" @click="mostrarFormReserva = true">
            Reservar para Retiro en Tienda
          </button>
        </div>
        
        <div v-else class="reserva-form">
          <h4>Agendar Cita de Retiro</h4>
          <div class="form-row">
            <div>
              <label>Fecha:</label>
              <input type="date" v-model="fechaReserva" required>
            </div>
            <div>
              <label>Hora (09:00 - 17:30):</label>
              <input type="time" v-model="horaReserva" required>
            </div>
          </div>
          
          <div class="form-actions" style="margin-top: 15px;">
            <button class="btn" :disabled="reservando" @click="realizarReserva">
              {{ reservando ? 'Reservando...' : 'Confirmar Reserva' }}
            </button>
            <button class="btn btn-outline" style="margin-left: 10px;" @click="mostrarFormReserva = false">
              Cancelar
            </button>
          </div>
          
          <p v-if="mensajeReserva" class="mensaje-reserva">{{ mensajeReserva }}</p>
        </div>
      </div>

      <div class="chatbot-inline">
        <div class="chat-head">
          <span>Asistente AutoParts</span>
        </div>
        <div v-if="props.user" class="chat-wrapper">
          <div class="chat-body">
            <div
              v-for="(msg, i) in mensajes"
              :key="i"
              class="msg"
              :class="msg.tipo"
            >
              {{ msg.texto }}
            </div>
            <div v-if="enviandoChat" class="msg ia loading-dots">Asistente está escribiendo...</div>
          </div>
          <div class="chat-input">
            <input
              type="text"
              placeholder="Pregunta compatibilidad, stock..."
              v-model="nuevoMensaje"
              :disabled="enviandoChat"
              @keyup.enter="enviarMensaje"
            >
            <button @click="enviarMensaje" :disabled="enviandoChat">
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="22" y1="2" x2="11" y2="13"></line><polygon points="22 2 15 22 11 13 2 9 22 2"></polygon></svg>
            </button>
          </div>
        </div>
        <div v-else class="chat-locked">
          <div style="font-size: 2.5rem; margin-bottom: 12px;">🔒</div>
          <p style="font-size: 0.9rem; color: var(--text-light); margin-bottom: 15px; font-weight: 500; line-height: 1.4; padding: 0 10px;">
            Inicia sesión para interactuar con nuestro Asistente de IA y recibir ayuda personalizada.
          </p>
          <button class="btn btn-sm" @click="$emit('ir-a-login')" style="padding: 8px 16px; font-size: 0.85rem;">
            Iniciar Sesión
          </button>
        </div>
      </div>
    </div>
    <div v-else class="no-results">El repuesto seleccionado no existe.</div>
  </div>
</template>

<style scoped>
.loading { text-align: center; font-size: 1.2rem; color: var(--text-light); margin-top: 50px; }
.detail-view { display: flex; gap: 30px; background: var(--surface); padding: 30px; border-radius: 16px; box-shadow: var(--shadow); align-items: flex-start; }
.detail-img { width: 350px; height: 350px; background: var(--bg); border-radius: 12px; display: flex; flex-direction: column; align-items: center; justify-content: center; color: var(--text-light); border: 1px dashed var(--border); }
.detail-info { flex: 1; }
.descripcion { color: var(--text-light); margin-bottom: 20px; line-height: 1.6; }
.tag { display: inline-block; padding: 4px 10px; background: #dcfce7; color: #166534; border-radius: 20px; font-size: 0.8rem; font-weight: 600; margin-bottom: 16px; }
.tag.no-stock { background: #fee2e2; color: #991b1b; }
.info-box { background: var(--bg); padding: 16px; border-radius: 8px; margin-bottom: 24px; border: 1px solid var(--border); }
.price { color: var(--accent); font-weight: 700; margin-bottom: 16px; }

.reserva-form { background: var(--bg); padding: 20px; border-radius: 12px; border: 1px solid var(--border); margin-top: 15px; }
.reserva-form h4 { color: var(--primary); margin-bottom: 15px; }
.form-row { display: flex; gap: 15px; }
.form-row label { display: block; font-size: 0.85rem; color: var(--text-light); margin-bottom: 5px; font-weight: 500; }
.form-row input { padding: 10px; border: 1px solid var(--border); border-radius: 6px; outline: none; width: 100%; background: var(--surface); color: var(--text); }
.mensaje-reserva { margin-top: 15px; font-weight: 600; font-size: 0.9rem; color: var(--accent); }

.chatbot-inline { width: 320px; border-radius: 12px; box-shadow: var(--shadow); overflow: hidden; border: 1px solid var(--border); background: var(--surface); }
.chat-head { background: var(--primary); color: white; padding: 16px; font-weight: 600; font-size: 0.95rem; display: flex; justify-content: space-between; align-items: center; }
.chat-head span { display: flex; align-items: center; gap: 8px; }
.chat-head span::before { content: ''; width: 8px; height: 8px; background: #4ade80; border-radius: 50%; display: inline-block; }
.chat-body { height: 260px; padding: 16px; overflow-y: auto; background: var(--bg); display: flex; flex-direction: column; gap: 12px; }
.msg { padding: 10px 14px; border-radius: 12px; font-size: 0.9rem; max-width: 85%; line-height: 1.4; word-wrap: break-word; }
.msg.ia { background: var(--surface); border: 1px solid var(--border); border-bottom-left-radius: 4px; align-self: flex-start; }
.msg.user { background: var(--accent); color: white; border-bottom-right-radius: 4px; align-self: flex-end; }
.loading-dots { font-style: italic; color: var(--text-light); }
.chat-input { display: flex; padding: 12px; background: var(--surface); border-top: 1px solid var(--border); }
.chat-input input { flex: 1; padding: 10px 14px; border: 1px solid var(--border); border-radius: 20px; outline: none; font-size: 0.9rem; background: var(--surface); color: var(--text); }
.chat-input input:focus { border-color: var(--accent); }
.chat-input button { background: var(--accent); color: white; border: none; width: 36px; height: 36px; border-radius: 50%; margin-left: 8px; cursor: pointer; display: flex; align-items: center; justify-content: center; }
.chat-input button:disabled { background: var(--border); cursor: not-allowed; }

.chat-locked {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  padding: 30px 20px;
  background: var(--bg);
  height: 316px;
}
</style>