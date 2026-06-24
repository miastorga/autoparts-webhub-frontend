<script setup>
import { ref, onMounted, watch } from 'vue'

const props = defineProps({
  user: {
    type: Object,
    required: true
  }
})

defineEmits(['logout'])

// Pestaña inicializada en 'citas' como primera opción
const seccionActiva = ref('citas')

const inventario = ref([])
const appointments = ref([])
const settings = ref([])
const cargando = ref(false)
const cargandoAjustes = ref(false)
const mensajeAjustes = ref('')

onMounted(async () => {
  await cargarDatos()
})

watch(seccionActiva, async () => {
  await cargarDatos()
})

async function cargarDatos() {
  try {
    cargando.value = true
    if (seccionActiva.value === 'inventario') {
      const res = await fetch('/api/parts')
      if (res.ok) {
        inventario.value = await res.json()
      }
    } else if (seccionActiva.value === 'citas') {
      const res = await fetch('/api/appointments', {
        headers: {
          'x-bypass-auth': 'true' // Bypass auth en desarrollo para pruebas locales
        }
      })
      if (res.ok) {
        appointments.value = await res.json()
      }
    } else if (seccionActiva.value === 'ajustes') {
      await cargarAjustes()
    }
  } catch (err) {
    console.error('Error al cargar datos en admin:', err)
  } finally {
    cargando.value = false
  }
}

async function cargarAjustes() {
  try {
    cargandoAjustes.value = true
    mensajeAjustes.value = ''
    const res = await fetch('/api/settings', {
      headers: {
        'x-bypass-auth': 'true'
      }
    })
    if (res.ok) {
      settings.value = await res.json()
    }
  } catch (err) {
    console.error('Error al cargar ajustes:', err)
  } finally {
    cargandoAjustes.value = false
  }
}

async function guardarAjustes() {
  try {
    cargandoAjustes.value = true
    mensajeAjustes.value = ''
    const res = await fetch('/api/settings', {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
        'x-bypass-auth': 'true'
      },
      body: JSON.stringify({ settings: settings.value })
    })
    if (res.ok) {
      mensajeAjustes.value = 'Configuraciones guardadas con éxito.'
      setTimeout(() => { mensajeAjustes.value = '' }, 3000)
    } else {
      mensajeAjustes.value = 'Error al guardar configuraciones.'
    }
  } catch (err) {
    console.error('Error al guardar ajustes:', err)
    mensajeAjustes.value = 'Error de red al guardar.'
  } finally {
    cargandoAjustes.value = false
  }
}

async function completarCita(id) {
  try {
    const res = await fetch(`/api/appointments/${id}/status`, {
      method: 'PATCH',
      headers: {
        'Content-Type': 'application/json',
        'x-bypass-auth': 'true'
      },
      body: JSON.stringify({ status: 'completed' })
    })
    if (res.ok) {
      await cargarDatos()
    } else {
      const data = await res.json()
      alert(data.error || 'No se pudo completar la cita')
    }
  } catch (err) {
    console.error('Error al completar cita:', err)
  }
}

function friendlySettingName(key) {
  const names = {
    'hourly_appointment_limit': 'Límite de citas por hora',
    'allow_start_time': 'Hora de inicio permitida',
    'allow_end_time': 'Hora de término permitida'
  }
  return names[key] || key
}

function formatoPrecio(valor) {
  return valor ? '$' + valor.toLocaleString('es-CL') : ''
}

function formatoFecha(fechaStr) {
  const d = new Date(fechaStr)
  return d.toLocaleDateString('es-CL', { timeZone: 'UTC' })
}

function formatoHora(horaStr) {
  return horaStr ? horaStr.substring(0, 5) : ''
}

function estiloStock(stock) {
  return stock > 3
    ? { background: '#dcfce7', color: '#166534' }
    : { background: '#fef9c3', color: '#854d0e' }
}
</script>

<template>
  <div class="screen">
    <div class="header">
      <h2 style="margin: 0;">Panel de Operaciones</h2>
      <div style="display: flex; align-items: center; gap: 15px;">
        <span style="font-size: 0.95rem; color: var(--text-light);">
          Conectado como: <strong>{{ props.user.email }}</strong> 
          <span style="text-transform: capitalize; font-size: 0.85rem; background: var(--bg); padding: 4px 8px; border-radius: 6px; margin-left: 5px; font-weight: 600; border: 1px solid var(--border);">
            {{ props.user.role === 'admin' ? 'Administrador' : 'Mecánico' }}
          </span>
        </span>
        <button class="btn btn-outline logout" @click="$emit('logout')">Cerrar Sesión</button>
      </div>
    </div>

    <div class="admin-layout">
      <div class="sidebar">
        <ul>
          <li :class="{ active: seccionActiva === 'citas' }" @click="seccionActiva = 'citas'">
            📅 Citas Logísticas
          </li>
          <li :class="{ active: seccionActiva === 'inventario' }" @click="seccionActiva = 'inventario'">
            📦 Inventario
          </li>
          <li v-if="props.user.role === 'admin'" :class="{ active: seccionActiva === 'ajustes' }" @click="seccionActiva = 'ajustes'">
            ⚙️ Ajustes del Sistema
          </li>
        </ul>
      </div>

      <div class="admin-content">
        <div v-if="cargando" class="loading-pane">Cargando información...</div>
        
        <div v-else-if="seccionActiva === 'citas'">
          <div class="content-header">
            <h3 style="color: var(--primary);">Citas de Retiro Logísticas</h3>
          </div>

          <table>
            <thead>
              <tr>
                <th>ID</th>
                <th>Cliente</th>
                <th>Repuesto</th>
                <th>SKU</th>
                <th>Cantidad</th>
                <th>Fecha</th>
                <th>Hora</th>
                <th>Creado por IA</th>
                <th>Estado</th>
                <th>Acciones</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="cita in appointments" :key="cita.id">
                <td>#{{ cita.id }}</td>
                <td>
                  <strong>{{ cita.user_name }}</strong>
                  <br><span style="font-size: 0.8rem; color: var(--text-light);">{{ cita.user_email }}</span>
                </td>
                <td style="font-weight: 500;">{{ cita.part_name }}</td>
                <td style="color: var(--text-light);">{{ cita.part_sku }}</td>
                <td>{{ cita.quantity }}</td>
                <td>{{ formatoFecha(cita.appointment_date) }}</td>
                <td>{{ formatoHora(cita.appointment_time) }}</td>
                <td>
                  <span class="badge" :style="cita.created_by_ia ? { background: '#dbeafe', color: '#1e40af' } : { background: '#f3f4f6', color: '#374151' }">
                    {{ cita.created_by_ia ? 'Sí (Gemini)' : 'No (Manual)' }}
                  </span>
                </td>
                <td>
                  <span class="badge" :style="cita.status === 'completed' ? { background: '#dcfce7', color: '#166534' } : (cita.status === 'cancelled' ? { background: '#fee2e2', color: '#991b1b' } : { background: '#fef9c3', color: '#854d0e' })">
                    {{ cita.status === 'completed' ? 'Completado' : (cita.status === 'cancelled' ? 'Cancelado' : 'Pendiente') }}
                  </span>
                </td>
                <td>
                  <button 
                    v-if="cita.status === 'pending'" 
                    class="btn-complete" 
                    @click="completarCita(cita.id)"
                  >
                    Completar
                  </button>
                  <span v-else style="color: var(--text-light); font-size: 0.85rem;">—</span>
                </td>
              </tr>
              <tr v-if="appointments.length === 0">
                <td colspan="10" style="text-align: center; color: var(--text-light); padding: 30px;">
                  No hay citas de retiro programadas.
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <div v-else-if="seccionActiva === 'inventario'">
          <div class="content-header">
            <h3 style="color: var(--primary);">Gestión de Inventario</h3>
            <button class="btn">+ Agregar Producto</button>
          </div>

          <table>
            <thead>
              <tr>
                <th>SKU</th>
                <th>Producto</th>
                <th>Categoría</th>
                <th>Ubicación</th>
                <th>Precio</th>
                <th>Stock</th>
                <th>Acciones</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in inventario" :key="item.id">
                <td style="color: var(--text-light);">{{ item.sku }}</td>
                <td style="font-weight: 500;">{{ item.name }}</td>
                <td>{{ item.category }}</td>
                <td>{{ item.warehouse_location }}</td>
                <td>{{ formatoPrecio(item.price) }}</td>
                <td>
                  <span class="badge" :style="estiloStock(item.available_stock)">{{ item.available_stock }} Unid.</span>
                </td>
                <td>
                  <a href="#" class="action-link">Editar</a>
                  <a href="#" class="action-link delete">Eliminar</a>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <div v-else-if="seccionActiva === 'ajustes' && props.user.role === 'admin'">
          <h3 style="color: var(--primary); margin-bottom: 20px;">Ajustes del Sistema</h3>
          
          <div v-if="cargandoAjustes" class="loading-pane">Cargando ajustes...</div>
          <div v-else>
            <table style="margin-bottom: 20px;">
              <thead>
                <tr>
                  <th>Configuración</th>
                  <th>Valor</th>
                  <th>Descripción</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="setting in settings" :key="setting.setting_key">
                  <td style="font-weight: 600;">{{ friendlySettingName(setting.setting_key) }}</td>
                  <td>
                    <input 
                      v-model="setting.setting_value" 
                      class="settings-input"
                      :type="setting.setting_key === 'hourly_appointment_limit' ? 'number' : 'text'"
                    >
                  </td>
                  <td style="color: var(--text-light); font-size: 0.88rem;">{{ setting.description }}</td>
                </tr>
              </tbody>
            </table>
            
            <div style="margin-top: 25px; display: flex; align-items: center; gap: 15px;">
              <button class="btn" @click="guardarAjustes" :disabled="cargandoAjustes">
                {{ cargandoAjustes ? 'Guardando...' : 'Guardar Cambios' }}
              </button>
              <span v-if="mensajeAjustes" style="color: var(--primary); font-weight: 500; font-size: 0.95rem;">
                {{ mensajeAjustes }}
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 30px; }
.logout { border-color: #ef4444; color: #ef4444; }

.admin-layout { display: flex; gap: 24px; }
.sidebar { width: 240px; background: var(--surface); padding: 20px; border-radius: 16px; box-shadow: var(--shadow); align-self: flex-start; }
.sidebar ul { list-style: none; }
.sidebar li { padding: 12px 16px; margin-bottom: 8px; border-radius: 8px; cursor: pointer; font-size: 0.95rem; color: var(--text); font-weight: 500; transition: all 0.2s; }
.sidebar li:hover { background: var(--bg); }
.sidebar li.active { background: var(--primary); color: white; }
.admin-content { flex: 1; background: var(--surface); padding: 30px; border-radius: 16px; box-shadow: var(--shadow); overflow-x: auto; }

.content-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }

table { width: 100%; border-collapse: collapse; margin-top: 20px; }
th { text-align: left; padding: 14px; border-bottom: 2px solid var(--border); color: var(--text-light); font-weight: 600; font-size: 0.9rem; text-transform: uppercase; letter-spacing: 0.5px; }
td { padding: 16px 14px; border-bottom: 1px solid var(--border); font-size: 0.95rem; }
tbody tr:hover { background: var(--bg); }
.badge { padding: 4px 8px; border-radius: 12px; font-size: 0.85rem; font-weight: 600; }
.action-link { color: var(--accent); text-decoration: none; font-weight: 500; margin-right: 10px; }
.action-link:hover { text-decoration: underline; }
.action-link.delete { color: #ef4444; }

.loading-pane { text-align: center; color: var(--text-light); padding: 50px; font-size: 1.1rem; }

.settings-input {
  width: 140px; 
  padding: 8px 12px; 
  border-radius: 6px; 
  border: 1px solid var(--border); 
  background: var(--bg); 
  color: var(--text);
  font-size: 0.9rem;
  outline: none;
  transition: border-color 0.2s;
}
.settings-input:focus {
  border-color: var(--primary);
}

.btn-complete {
  background: #22c55e; 
  color: white; 
  border: none; 
  border-radius: 6px; 
  padding: 6px 12px;
  font-weight: 600;
  font-size: 0.82rem;
  cursor: pointer; 
  transition: background 0.2s;
}
.btn-complete:hover {
  background: #16a34a;
}
</style>