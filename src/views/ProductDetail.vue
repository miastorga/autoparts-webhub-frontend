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

defineEmits(['volver', 'ir-a-login', 'agregar-carrito'])

const producto = ref(null)
const cargando = ref(true)

onMounted(async () => {
  await cargarDetalle()
})

watch(() => props.partId, async () => {
  await cargarDetalle()
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
        <img :src="producto.image_url || '/images/repuesto_defecto.png'" :alt="producto.name">
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
        
        <div>
          <button 
            class="btn" 
            style="padding: 14px 30px; font-size: 1.05rem;" 
            :disabled="producto.available_stock === 0"
            @click="$emit('agregar-carrito', producto)"
          >
            {{ producto.available_stock === 0 ? 'Sin Stock' : 'Agregar al Carrito de Reserva' }}
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
.detail-img { width: 350px; height: 350px; background: var(--bg); border-radius: 12px; display: flex; align-items: center; justify-content: center; overflow: hidden; border: 1px solid var(--border); }
.detail-img img { width: 100%; height: 100%; object-fit: cover; }
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