<script setup>
import { ref } from 'vue'

defineEmits(['volver'])

const mensajes = ref([
  { tipo: 'ia', texto: '¡Hola! ¿Necesitas ayuda verificando si estas pastillas le sirven a tu auto?' },
  { tipo: 'user', texto: 'Sí, tengo un Yaris año 2018 sedán.' },
  { tipo: 'ia', texto: 'Revisando... ✅ Sí, son 100% compatibles con el Yaris 2018 Sedán. ¿Deseas que te ayude a agendar un retiro en nuestra tienda física?' },
])

const nuevoMensaje = ref('')

function enviarMensaje() {
  if (!nuevoMensaje.value.trim()) return
  mensajes.value.push({ tipo: 'user', texto: nuevoMensaje.value })
  nuevoMensaje.value = ''
}
</script>

<template>
  <div class="screen">
    <button class="btn btn-outline" style="margin-bottom: 20px;" @click="$emit('volver')">
      ← Volver al catálogo
    </button>

    <div class="detail-view">
      <div class="detail-img">Imagen del Producto (Alta Resol.)</div>

      <div class="detail-info">
        <span class="tag">En Stock (5 unidades)</span>
        <h2 style="margin-bottom: 8px;">Pastillas de Freno Cerámicas</h2>
        <p class="descripcion">
          Pastillas de freno de alto rendimiento diseñadas para reducir el desgaste del disco y minimizar el polvo.
        </p>

        <div class="info-box">
          <p style="margin-bottom: 8px;"><strong>Marca:</strong> Alternativa Premium</p>
          <p><strong>Compatibilidad:</strong> Toyota Yaris (2018-2022)</p>
        </div>

        <div class="price" style="font-size: 2rem;">$25.000</div>
        <button class="btn" style="padding: 14px 30px; font-size: 1.05rem;">
          Reservar para Retiro en Tienda
        </button>
      </div>

      <div class="chatbot-inline">
        <div class="chat-head">
          <span>Asistente AutoParts</span>
        </div>
        <div class="chat-body">
          <div
            v-for="(msg, i) in mensajes"
            :key="i"
            class="msg"
            :class="msg.tipo"
          >
            {{ msg.texto }}
          </div>
        </div>
        <div class="chat-input">
          <input
            type="text"
            placeholder="Escribe tu respuesta..."
            v-model="nuevoMensaje"
            @keyup.enter="enviarMensaje"
          >
          <button @click="enviarMensaje">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="22" y1="2" x2="11" y2="13"></line><polygon points="22 2 15 22 11 13 2 9 22 2"></polygon></svg>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.detail-view { display: flex; gap: 30px; background: var(--surface); padding: 30px; border-radius: 16px; box-shadow: var(--shadow); align-items: flex-start; }
.detail-img { width: 350px; height: 350px; background: var(--bg); border-radius: 12px; display: flex; align-items: center; justify-content: center; color: var(--text-light); border: 1px dashed var(--border); }
.detail-info { flex: 1; }
.descripcion { color: var(--text-light); margin-bottom: 20px; line-height: 1.6; }
.tag { display: inline-block; padding: 4px 10px; background: #dcfce7; color: #166534; border-radius: 20px; font-size: 0.8rem; font-weight: 600; margin-bottom: 16px; }
.info-box { background: var(--bg); padding: 16px; border-radius: 8px; margin-bottom: 24px; border: 1px solid var(--border); }
.price { color: var(--accent); font-weight: 700; margin-bottom: 16px; }

.chatbot-inline { width: 320px; border-radius: 12px; box-shadow: var(--shadow); overflow: hidden; border: 1px solid var(--border); background: var(--surface); }
.chat-head { background: var(--primary); color: white; padding: 16px; font-weight: 600; font-size: 0.95rem; display: flex; justify-content: space-between; align-items: center; }
.chat-head span { display: flex; align-items: center; gap: 8px; }
.chat-head span::before { content: ''; width: 8px; height: 8px; background: #4ade80; border-radius: 50%; display: inline-block; }
.chat-body { height: 260px; padding: 16px; overflow-y: auto; background: var(--bg); display: flex; flex-direction: column; gap: 12px; }
.msg { padding: 10px 14px; border-radius: 12px; font-size: 0.9rem; max-width: 85%; line-height: 1.4; }
.msg.ia { background: var(--surface); border: 1px solid var(--border); border-bottom-left-radius: 4px; align-self: flex-start; }
.msg.user { background: var(--accent); color: white; border-bottom-right-radius: 4px; align-self: flex-end; }
.chat-input { display: flex; padding: 12px; background: var(--surface); border-top: 1px solid var(--border); }
.chat-input input { flex: 1; padding: 10px 14px; border: 1px solid var(--border); border-radius: 20px; outline: none; font-size: 0.9rem; }
.chat-input input:focus { border-color: var(--accent); }
.chat-input button { background: var(--accent); color: white; border: none; width: 36px; height: 36px; border-radius: 50%; margin-left: 8px; cursor: pointer; display: flex; align-items: center; justify-content: center; }
</style>