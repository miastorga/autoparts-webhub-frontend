<script setup>
import { ref, onMounted, watch, computed } from 'vue'

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

const filtroEmail = ref('')
const appointmentsFiltradas = computed(() => {
  if (!filtroEmail.value) return appointments.value
  const term = filtroEmail.value.toLowerCase().trim()
  return appointments.value.filter(cita => 
    (cita.user_email && cita.user_email.toLowerCase().includes(term)) ||
    (cita.user_name && cita.user_name.toLowerCase().includes(term))
  )
})

// State for parts CRUD
const mostrarFormProducto = ref(false)
const productoEditando = ref({
  sku: '',
  name: '',
  category: '',
  price: 0,
  available_stock: 0,
  warehouse_location: '',
  image_url: ''
})
const esEdicion = ref(false)
const errorProducto = ref('')
const cargandoProducto = ref(false)

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

async function cancelarCita(id) {
  try {
    const res = await fetch(`/api/appointments/${id}/status`, {
      method: 'PATCH',
      headers: {
        'Content-Type': 'application/json',
        'x-bypass-auth': 'true'
      },
      body: JSON.stringify({ status: 'cancelled' })
    })
    if (res.ok) {
      await cargarDatos()
    } else {
      const data = await res.json()
      alert(data.error || 'No se pudo cancelar la cita')
    }
  } catch (err) {
    console.error('Error al cancelar cita:', err)
  }
}

// Atender Cita Modal
const mostrarModalAtender = ref(false)
const citaAtendiendo = ref(null)
const editandoItems = ref([])
const guardandoAtender = ref(false)
const errorAtender = ref('')

const todosLosRepuestos = ref([])
const repuestoSeleccionadoId = ref('')
const cantidadAAgregar = ref(1)

async function cargarTodosLosRepuestos() {
  try {
    const res = await fetch('/api/parts')
    if (res.ok) {
      todosLosRepuestos.value = await res.json()
    }
  } catch (err) {
    console.error('Error al cargar repuestos:', err)
  }
}

async function abrirAtenderCita(cita) {
  citaAtendiendo.value = cita
  editandoItems.value = cita.items.map(item => ({
    part_id: item.part_id,
    quantity: item.quantity,
    part_name: item.part_name,
    part_sku: item.part_sku,
    part_price: item.part_price
  }))
  errorAtender.value = ''
  repuestoSeleccionadoId.value = ''
  cantidadAAgregar.value = 1
  mostrarModalAtender.value = true
  await cargarTodosLosRepuestos()
}

function agregarRepuestoAEdicion() {
  if (!repuestoSeleccionadoId.value) return
  
  const part = todosLosRepuestos.value.find(p => p.id === parseInt(repuestoSeleccionadoId.value, 10))
  if (!part) return

  // Check if it's already in the cart
  const existente = editandoItems.value.find(i => i.part_id === part.id)
  if (existente) {
    existente.quantity += cantidadAAgregar.value
  } else {
    editandoItems.value.push({
      part_id: part.id,
      quantity: cantidadAAgregar.value,
      part_name: part.name,
      part_sku: part.sku,
      part_price: part.price
    })
  }

  // Reset fields
  repuestoSeleccionadoId.value = ''
  cantidadAAgregar.value = 1
}

function eliminarItemEdicion(partId) {
  if (editandoItems.value.length <= 1) {
    errorAtender.value = 'Una cita debe contener al menos 1 repuesto. Si el cliente no desea llevar nada, marca la cita como "No retiró".'
    return
  }
  editandoItems.value = editandoItems.value.filter(i => i.part_id !== partId)
}

function actualizarCantidadEdicion(partId, cant) {
  const item = editandoItems.value.find(i => i.part_id === partId)
  if (item) {
    const val = parseInt(cant, 10)
    if (val < 1) {
      item.quantity = 1
    } else {
      item.quantity = val
    }
  }
}

async function guardarCambiosCarrito() {
  errorAtender.value = ''
  guardandoAtender.value = true
  try {
    const res = await fetch(`/api/appointments/${citaAtendiendo.value.id}/items`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
        'x-bypass-auth': 'true'
      },
      body: JSON.stringify({
        items: editandoItems.value.map(i => ({
          part_id: i.part_id,
          quantity: i.quantity
        }))
      })
    })

    if (res.ok) {
      return true
    } else {
      const data = await res.json()
      errorAtender.value = data.error || 'Error al actualizar el carrito.'
      return false
    }
  } catch (err) {
    console.error('Error al guardar cambios de items:', err)
    errorAtender.value = 'Error de conexión.'
    return false
  } finally {
    guardandoAtender.value = false
  }
}

async function procesarCompletarAtender() {
  const guardado = await guardarCambiosCarrito()
  if (!guardado) return

  try {
    const res = await fetch(`/api/appointments/${citaAtendiendo.value.id}/status`, {
      method: 'PATCH',
      headers: {
        'Content-Type': 'application/json',
        'x-bypass-auth': 'true'
      },
      body: JSON.stringify({ status: 'completed' })
    })

    if (res.ok) {
      mostrarModalAtender.value = false
      await cargarDatos()
    } else {
      const data = await res.json()
      errorAtender.value = data.error || 'Error al completar la cita.'
    }
  } catch (err) {
    console.error('Error al completar cita:', err)
    errorAtender.value = 'Error de conexión.'
  }
}

async function procesarCancelarAtender() {
  try {
    const res = await fetch(`/api/appointments/${citaAtendiendo.value.id}/status`, {
      method: 'PATCH',
      headers: {
        'Content-Type': 'application/json',
        'x-bypass-auth': 'true'
      },
      body: JSON.stringify({ status: 'cancelled' })
    })

    if (res.ok) {
      mostrarModalAtender.value = false
      await cargarDatos()
    } else {
      const data = await res.json()
      errorAtender.value = data.error || 'Error al cancelar la cita.'
    }
  } catch (err) {
    console.error('Error al cancelar cita:', err)
    errorAtender.value = 'Error de conexión.'
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

function abrirAgregarProducto() {
  esEdicion.value = false
  errorProducto.value = ''
  productoEditando.value = {
    sku: '',
    name: '',
    category: '',
    price: 0,
    available_stock: 0,
    warehouse_location: '',
    image_url: ''
  }
  mostrarFormProducto.value = true
}

function abrirEditarProducto(item) {
  esEdicion.value = true
  errorProducto.value = ''
  productoEditando.value = { ...item }
  mostrarFormProducto.value = true
}

async function guardarProducto() {
  errorProducto.value = ''
  
  const p = productoEditando.value
  if (!p.sku || !p.name || !p.category || p.price === undefined || p.available_stock === undefined || !p.warehouse_location) {
    errorProducto.value = 'Por favor, completa todos los campos.'
    return
  }

  try {
    cargandoProducto.value = true
    const url = esEdicion.value ? `/api/parts/${p.id}` : '/api/parts'
    const method = esEdicion.value ? 'PUT' : 'POST'
    
    const res = await fetch(url, {
      method,
      headers: {
        'Content-Type': 'application/json',
        'x-bypass-auth': 'true' // bypass para desarrollo/sesión
      },
      body: JSON.stringify(p)
    })

    if (res.ok) {
      mostrarFormProducto.value = false
      await cargarDatos()
    } else {
      const data = await res.json()
      errorProducto.value = data.error || 'Error al guardar el producto.'
    }
  } catch (err) {
    console.error('Error al guardar producto:', err)
    errorProducto.value = 'Error de red al guardar.'
  } finally {
    cargandoProducto.value = false
  }
}

async function eliminarProducto(id) {
  if (!confirm('¿Estás seguro de que deseas eliminar este repuesto? Esta acción no se puede deshacer.')) {
    return
  }

  try {
    const res = await fetch(`/api/parts/${id}`, {
      method: 'DELETE',
      headers: {
        'x-bypass-auth': 'true'
      }
    })

    if (res.ok) {
      await cargarDatos()
    } else {
      const data = await res.json()
      alert(data.error || 'Error al eliminar el producto.')
    }
  } catch (err) {
    console.error('Error al eliminar producto:', err)
    alert('Error de red al intentar eliminar.')
  }
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
          <div class="content-header" style="display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 15px; margin-bottom: 20px;">
            <h3 style="color: var(--primary); margin: 0;">Citas de Retiro Logísticas</h3>
            <input 
              type="text" 
              v-model="filtroEmail" 
              placeholder="🔍 Buscar por correo o nombre..." 
              style="padding: 8px 14px; border: 1px solid var(--border); border-radius: 8px; font-size: 0.9rem; width: 280px; background: var(--bg); color: var(--text); outline: none; transition: border-color 0.2s;"
            >
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
                <th>Estado</th>
                <th>Acciones</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="cita in appointmentsFiltradas" :key="cita.id" :class="{ 'row-completed': cita.status === 'completed' || cita.status === 'cancelled' }">
                <td>#{{ cita.id }}</td>
                <td>
                  <strong>{{ cita.user_name }}</strong>
                  <br><span style="font-size: 0.78rem; color: var(--text-light);">{{ cita.user_email }}</span>
                </td>
                <td class="part-name-cell" style="font-weight: 500;">{{ cita.part_name }}</td>
                <td style="color: var(--text-light);">{{ cita.part_sku }}</td>
                <td>{{ cita.quantity }}</td>
                <td>{{ formatoFecha(cita.appointment_date) }}</td>
                <td>{{ formatoHora(cita.appointment_time) }}</td>
                <td>
                  <span class="badge" :style="cita.status === 'completed' ? { background: '#dcfce7', color: '#166534' } : (cita.status === 'cancelled' ? { background: '#fee2e2', color: '#991b1b' } : { background: '#fef9c3', color: '#854d0e' })">
                    {{ cita.status === 'completed' ? 'Completado' : (cita.status === 'cancelled' ? 'Cancelado' : 'Pendiente') }}
                  </span>
                </td>
                <td>
                  <button 
                    v-if="cita.status === 'pending'" 
                    class="btn-complete" 
                    @click="abrirAtenderCita(cita)"
                  >
                    Atender
                  </button>
                  <span v-else style="color: var(--text-light); font-size: 0.85rem;">—</span>
                </td>
              </tr>
              <tr v-if="appointmentsFiltradas.length === 0">
                <td colspan="9" style="text-align: center; color: var(--text-light); padding: 30px;">
                  No se encontraron citas de retiro.
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <div v-else-if="seccionActiva === 'inventario'">
          <div class="content-header">
            <h3 style="color: var(--primary);">Gestión de Inventario</h3>
            <button class="btn" @click="abrirAgregarProducto">+ Agregar Producto</button>
          </div>

          <table>
            <thead>
              <tr>
                <th>Imagen</th>
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
                <td>
                  <img :src="item.image_url || '/images/repuesto_defecto.png'" style="width: 38px; height: 38px; border-radius: 50%; object-fit: cover; border: 1px solid var(--border); display: block;" />
                </td>
                <td style="color: var(--text-light);">{{ item.sku }}</td>
                <td style="font-weight: 500;">{{ item.name }}</td>
                <td>{{ item.category }}</td>
                <td>{{ item.warehouse_location }}</td>
                <td>{{ formatoPrecio(item.price) }}</td>
                <td>
                  <span class="badge" :style="estiloStock(item.available_stock)">{{ item.available_stock }} Unid.</span>
                </td>
                <td>
                  <a href="#" class="action-link" @click.prevent="abrirEditarProducto(item)">Editar</a>
                  <a href="#" class="action-link delete" @click.prevent="eliminarProducto(item.id)">Eliminar</a>
                </td>
              </tr>
              <tr v-if="inventario.length === 0">
                <td colspan="7" style="text-align: center; color: var(--text-light); padding: 30px;">
                  No hay repuestos en el inventario.
                </td>
              </tr>
            </tbody>
          </table>

          <!-- Modal para Agregar/Editar Producto -->
          <div v-if="mostrarFormProducto" class="modal-overlay" @click.self="mostrarFormProducto = false">
            <div class="modal-card">
              <div class="modal-header">
                <h3 style="margin: 0; color: var(--primary);">{{ esEdicion ? 'Editar Repuesto' : 'Agregar Nuevo Repuesto' }}</h3>
                <button class="close-btn" @click="mostrarFormProducto = false">&times;</button>
              </div>
              
              <form @submit.prevent="guardarProducto" class="modal-form">
                <div class="form-row" style="display: flex; gap: 15px; margin-bottom: 15px;">
                  <div class="form-group" style="flex: 1; display: flex; flex-direction: column; text-align: left;">
                    <label for="sku" style="margin-bottom: 5px; font-size: 0.9rem; font-weight: 500;">SKU</label>
                    <input 
                      type="text" 
                      id="sku" 
                      v-model="productoEditando.sku" 
                      placeholder="Ej: BRK-TY-001" 
                      required 
                      :disabled="esEdicion"
                      style="padding: 10px; border: 1px solid var(--border); border-radius: 8px; font-size: 0.95rem; background: var(--bg);"
                    >
                  </div>
                  <div class="form-group" style="flex: 1; display: flex; flex-direction: column; text-align: left;">
                    <label for="category" style="margin-bottom: 5px; font-size: 0.9rem; font-weight: 500;">Categoría</label>
                    <input 
                      type="text" 
                      id="category" 
                      v-model="productoEditando.category" 
                      placeholder="Ej: Frenos" 
                      required
                      style="padding: 10px; border: 1px solid var(--border); border-radius: 8px; font-size: 0.95rem; background: var(--bg);"
                    >
                  </div>
                </div>

                <div class="form-group" style="display: flex; flex-direction: column; text-align: left; margin-bottom: 15px;">
                  <label for="name" style="margin-bottom: 5px; font-size: 0.9rem; font-weight: 500;">Nombre del Producto</label>
                  <input 
                    type="text" 
                    id="name" 
                    v-model="productoEditando.name" 
                    placeholder="Ej: Pastillas de Freno Premium" 
                    required
                    style="padding: 10px; border: 1px solid var(--border); border-radius: 8px; font-size: 0.95rem; background: var(--bg);"
                  >
                </div>

                <div class="form-row" style="display: flex; gap: 15px; margin-bottom: 15px;">
                  <div class="form-group" style="flex: 1; display: flex; flex-direction: column; text-align: left;">
                    <label for="price" style="margin-bottom: 5px; font-size: 0.9rem; font-weight: 500;">Precio ($ CLP)</label>
                    <input 
                      type="number" 
                      id="price" 
                      v-model.number="productoEditando.price" 
                      min="0" 
                      required
                      style="padding: 10px; border: 1px solid var(--border); border-radius: 8px; font-size: 0.95rem; background: var(--bg);"
                    >
                  </div>
                  <div class="form-group" style="flex: 1; display: flex; flex-direction: column; text-align: left;">
                    <label for="stock" style="margin-bottom: 5px; font-size: 0.9rem; font-weight: 500;">Stock Disponible</label>
                    <input 
                      type="number" 
                      id="stock" 
                      v-model.number="productoEditando.available_stock" 
                      min="0" 
                      required
                      style="padding: 10px; border: 1px solid var(--border); border-radius: 8px; font-size: 0.95rem; background: var(--bg);"
                    >
                  </div>
                </div>

                <div class="form-group" style="display: flex; flex-direction: column; text-align: left; margin-bottom: 15px;">
                  <label for="location" style="margin-bottom: 5px; font-size: 0.9rem; font-weight: 500;">Ubicación en Bodega</label>
                  <input 
                    type="text" 
                    id="location" 
                    v-model="productoEditando.warehouse_location" 
                    placeholder="Ej: A-12" 
                    required
                    style="padding: 10px; border: 1px solid var(--border); border-radius: 8px; font-size: 0.95rem; background: var(--bg);"
                  >
                </div>

                <div class="form-group" style="display: flex; flex-direction: column; text-align: left; margin-bottom: 20px;">
                  <label for="image_url" style="margin-bottom: 5px; font-size: 0.9rem; font-weight: 500;">URL de Imagen del Producto</label>
                  <input 
                    type="text" 
                    id="image_url" 
                    v-model="productoEditando.image_url" 
                    placeholder="Ej: /images/freno_premium.png" 
                    style="padding: 10px; border: 1px solid var(--border); border-radius: 8px; font-size: 0.95rem; background: var(--bg);"
                  >
                </div>

                <p v-if="errorProducto" class="error-msg" style="color: #ef4444; font-size: 0.85rem; font-weight: 500; margin-bottom: 15px; text-align: left;">{{ errorProducto }}</p>

                <div class="form-actions" style="display: flex; gap: 10px; justify-content: flex-end;">
                  <button type="submit" class="btn" :disabled="cargandoProducto">
                    {{ cargandoProducto ? 'Guardando...' : 'Guardar Repuesto' }}
                  </button>
                  <button type="button" class="btn btn-outline" @click="mostrarFormProducto = false">
                    Cancelar
                  </button>
                </div>
              </form>
            </div>
          </div>
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
          <!-- Modal para Atender Cita (Múltiples Repuestos) -->
          <div v-if="mostrarModalAtender" class="modal-overlay" @click.self="mostrarModalAtender = false">
            <div class="modal-card" style="max-width: 600px;">
              <div class="modal-header">
                <h3 style="margin: 0; color: var(--primary);">Atender Cita de Retiro #{{ citaAtendiendo.id }}</h3>
                <button class="close-btn" @click="mostrarModalAtender = false">&times;</button>
              </div>

              <div style="text-align: left; margin-bottom: 20px; font-size: 0.92rem; color: var(--text-light); line-height: 1.5; border-bottom: 1px solid var(--border); padding-bottom: 12px;">
                <p><strong>Cliente:</strong> {{ citaAtendiendo.user_name }} ({{ citaAtendiendo.user_email }})</p>
                <p><strong>Fecha/Hora Programada:</strong> {{ formatoFecha(citaAtendiendo.appointment_date) }} a las {{ formatoHora(citaAtendiendo.appointment_time) }}</p>
                <p>
                  <strong>Creado por IA:</strong> 
                  <span class="badge" :style="citaAtendiendo.created_by_ia ? { background: '#dbeafe', color: '#1e40af', marginLeft: '5px' } : { background: '#f3f4f6', color: '#374151', marginLeft: '5px' }">
                    {{ citaAtendiendo.created_by_ia ? 'Sí (Gemini)' : 'No (Manual)' }}
                  </span>
                </p>
              </div>

              <h4 style="margin: 0 0 10px 0; text-align: left; color: var(--text); font-size: 0.95rem;">Repuestos Solicitados (Carrito)</h4>
              
              <div class="edit-items-list" style="display: flex; flex-direction: column; gap: 10px; margin-bottom: 20px; max-height: 250px; overflow-y: auto; padding-right: 5px;">
                <div v-for="item in editandoItems" :key="item.part_id" style="display: flex; align-items: center; justify-content: space-between; padding: 10px; background: var(--bg); border: 1px solid var(--border); border-radius: 8px;">
                  <div style="text-align: left; flex: 1; padding-right: 15px;">
                    <div style="font-weight: 600; font-size: 0.9rem; color: var(--text);">{{ item.part_name }}</div>
                    <div style="font-size: 0.78rem; color: var(--text-light);">SKU: {{ item.part_sku }} | Precio: {{ formatoPrecio(item.part_price) }}</div>
                  </div>
                  
                  <div style="display: flex; align-items: center; gap: 12px;">
                    <div class="quantity-picker" style="display: flex; align-items: center; border: 1px solid var(--border); border-radius: 6px; overflow: hidden; background: var(--surface);">
                      <button type="button" @click="actualizarCantidadEdicion(item.part_id, item.quantity - 1)" style="padding: 4px 8px; border: none; background: none; cursor: pointer; font-weight: bold; color: var(--text-light);">-</button>
                      <input type="number" :value="item.quantity" readonly style="width: 35px; text-align: center; border: none; background: none; font-size: 0.9rem; font-weight: 600; color: var(--text);">
                      <button type="button" @click="actualizarCantidadEdicion(item.part_id, item.quantity + 1)" style="padding: 4px 8px; border: none; background: none; cursor: pointer; font-weight: bold; color: var(--text-light);">+</button>
                    </div>
                    <button type="button" @click="eliminarItemEdicion(item.part_id)" style="background: none; border: none; cursor: pointer; font-size: 1.1rem; padding: 4px;">🗑️</button>
                  </div>
                </div>
              </div>

              <!-- Agregar Nuevo Item al Carrito de Cita -->
              <div class="add-item-to-appointment" style="margin-top: 15px; border-top: 1px dashed var(--border); padding-top: 15px; text-align: left;">
                <h5 style="margin: 0 0 10px 0; color: var(--text-light); font-size: 0.85rem; font-weight: 600;">Agregar otro repuesto al pedido:</h5>
                <div style="display: flex; gap: 10px; align-items: center; flex-wrap: wrap;">
                  <select 
                    v-model="repuestoSeleccionadoId" 
                    style="flex: 1; min-width: 200px; padding: 8px 12px; border: 1px solid var(--border); border-radius: 8px; font-size: 0.9rem; background: var(--bg); color: var(--text); outline: none;"
                  >
                    <option value="">-- Seleccionar Repuesto --</option>
                    <option 
                      v-for="part in todosLosRepuestos" 
                      :key="part.id" 
                      :value="part.id"
                      :disabled="part.available_stock === 0"
                    >
                      {{ part.name }} (SKU: {{ part.sku }}) - Stock: {{ part.available_stock }}
                    </option>
                  </select>
                  
                  <div class="quantity-picker" style="display: flex; align-items: center; border: 1px solid var(--border); border-radius: 6px; overflow: hidden; background: var(--surface); height: 38px;">
                    <button type="button" @click="cantidadAAgregar > 1 ? cantidadAAgregar-- : null" style="padding: 4px 8px; border: none; background: none; cursor: pointer; font-weight: bold; color: var(--text-light);">-</button>
                    <input type="number" v-model.number="cantidadAAgregar" style="width: 35px; text-align: center; border: none; background: none; font-size: 0.9rem; font-weight: 600; color: var(--text); outline: none;" min="1">
                    <button type="button" @click="cantidadAAgregar++" style="padding: 4px 8px; border: none; background: none; cursor: pointer; font-weight: bold; color: var(--text-light);">+</button>
                  </div>

                  <button 
                    type="button" 
                    class="btn" 
                    style="padding: 8px 16px; font-size: 0.88rem; min-height: 38px;"
                    :disabled="!repuestoSeleccionadoId"
                    @click="agregarRepuestoAEdicion"
                  >
                    + Agregar
                  </button>
                </div>
              </div>

              <p v-if="errorAtender" style="color: #ef4444; font-size: 0.85rem; font-weight: 500; margin-top: 15px; text-align: left;">{{ errorAtender }}</p>

              <div class="form-actions" style="display: flex; gap: 10px; justify-content: flex-end; border-top: 1px solid var(--border); padding-top: 15px; flex-wrap: wrap;">
                <button type="button" class="btn" style="background: #22c55e;" :disabled="guardandoAtender" @click="procesarCompletarAtender">
                  Confirmar Retiro (Completar)
                </button>
                <button type="button" class="btn" style="background: #ef4444;" :disabled="guardandoAtender" @click="procesarCancelarAtender">
                  Cliente no retiró (Cancelar)
                </button>
                <button type="button" class="btn btn-outline" @click="mostrarModalAtender = false">
                  Cerrar
                </button>
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
th { text-align: left; padding: 10px 8px; border-bottom: 2px solid var(--border); color: var(--text-light); font-weight: 600; font-size: 0.82rem; text-transform: uppercase; letter-spacing: 0.5px; }
td { padding: 10px 8px; border-bottom: 1px solid var(--border); font-size: 0.88rem; }
.part-name-cell { max-width: 140px; word-wrap: break-word; font-size: 0.85rem; }
.row-completed { opacity: 0.65; background-color: #f8fafc; }
.row-completed td { color: #94a3b8 !important; }
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

.btn-cancel {
  background: #ef4444; 
  color: white; 
  border: none; 
  border-radius: 6px; 
  padding: 6px 12px;
  font-weight: 600;
  font-size: 0.82rem;
  cursor: pointer; 
  transition: background 0.2s;
}
.btn-cancel:hover {
  background: #dc2626;
}

/* Modal Styling */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(15, 23, 42, 0.6);
  backdrop-filter: blur(8px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  animation: fadeIn 0.25s ease-out;
}

.modal-card {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 20px;
  width: 90%;
  max-width: 550px;
  padding: 30px;
  box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
  transform: translateY(0);
  animation: slideUp 0.3s cubic-bezier(0.16, 1, 0.3, 1);
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid var(--border);
  padding-bottom: 15px;
  margin-bottom: 20px;
}

.close-btn {
  background: none;
  border: none;
  font-size: 1.8rem;
  color: var(--text-light);
  cursor: pointer;
  line-height: 1;
  transition: color 0.2s;
}

.close-btn:hover {
  color: #ef4444;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideUp {
  from { transform: translateY(20px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}
</style>