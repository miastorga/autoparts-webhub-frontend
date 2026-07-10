<script setup>
import { ref, onMounted, watch } from 'vue'

const emit = defineEmits(['ver-detalle'])

const productos = ref([])
const marcas = ref([])
const modelos = ref([])
const anos = ref([])

const marcaSeleccionada = ref('')
const modeloSeleccionado = ref('')
const anoSeleccionado = ref('')
const buscarTexto = ref('')
const cargando = ref(false)

// Cargar marcas al montar
onMounted(async () => {
  try {
    cargando.value = true
    const res = await fetch('/api/vehicles/brands')
    marcas.value = await res.json()
    await cargarProductos()
  } catch (err) {
    console.error('Error al cargar marcas:', err)
  } finally {
    cargando.value = false
  }
})

// Cargar modelos cuando cambia la marca
watch(marcaSeleccionada, async (nuevaMarca) => {
  modeloSeleccionado.value = ''
  anoSeleccionado.value = ''
  modelos.value = []
  anos.value = []
  
  if (!nuevaMarca) {
    await cargarProductos()
    return
  }
  
  try {
    const res = await fetch(`/api/vehicles/models?brand=${nuevaMarca}`)
    modelos.value = await res.json()
  } catch (err) {
    console.error('Error al cargar modelos:', err)
  }
  await cargarProductos()
})

// Cargar años cuando cambia el modelo
watch(modeloSeleccionado, async (nuevoModelo) => {
  anoSeleccionado.value = ''
  anos.value = []
  
  if (!nuevoModelo) {
    await cargarProductos()
    return
  }
  
  try {
    const res = await fetch(`/api/vehicles/years?brand=${marcaSeleccionada.value}&model=${nuevoModelo}`)
    anos.value = await res.json()
  } catch (err) {
    console.error('Error al cargar años:', err)
  }
  await cargarProductos()
})

// Cargar productos cuando cambia el año o texto
watch(anoSeleccionado, async () => {
  await cargarProductos()
})

async function cargarProductos() {
  try {
    cargando.value = true
    let url = '/api/parts'
    
    // Si se seleccionó marca, modelo y año, filtramos por compatibilidad de vehículo
    if (marcaSeleccionada.value && modeloSeleccionado.value && anoSeleccionado.value) {
      const resVeh = await fetch(`/api/vehicles?brand=${marcaSeleccionada.value}&model=${modeloSeleccionado.value}&year=${anoSeleccionado.value}`)
      const vehs = await resVeh.json()
      if (vehs && vehs.length > 0) {
        url += `?vehicle_id=${vehs[0].id}`
      }
    }
    
    const res = await fetch(url)
    let parts = await res.json()
    
    // Filtrar localmente por texto de búsqueda si se ingresó algo
    if (buscarTexto.value.trim()) {
      const query = buscarTexto.value.toLowerCase().trim()
      parts = parts.filter(p => 
        p.name.toLowerCase().includes(query) || 
        p.sku.toLowerCase().includes(query) ||
        p.category.toLowerCase().includes(query)
      )
    }
    
    productos.value = parts
  } catch (err) {
    console.error('Error al cargar productos:', err)
  } finally {
    cargando.value = false
  }
}

function formatoPrecio(valor) {
  return '$' + valor.toLocaleString('es-CL')
}

function limpiarFiltros() {
  marcaSeleccionada.value = ''
  modeloSeleccionado.value = ''
  anoSeleccionado.value = ''
  buscarTexto.value = ''
  cargarProductos()
}
</script>

<template>
  <div class="screen">
    <h2>Encuentra tu repuesto</h2>

    <div class="filters">
      <select v-model="marcaSeleccionada">
        <option value="">Todas las Marcas</option>
        <option v-for="m in marcas" :key="m" :value="m">{{ m }}</option>
      </select>
      
      <select v-model="modeloSeleccionado" :disabled="!marcaSeleccionada">
        <option value="">Todos los Modelos</option>
        <option v-for="mod in modelos" :key="mod" :value="mod">{{ mod }}</option>
      </select>

      <select v-model="anoSeleccionado" :disabled="!modeloSeleccionado">
        <option value="">Todos los Años</option>
        <option v-for="a in anos" :key="a" :value="a">{{ a }}</option>
      </select>

      <input 
        type="text" 
        placeholder="¿Qué buscas? Ej. Pastillas, Filtro..." 
        v-model="buscarTexto"
        @keyup.enter="cargarProductos"
      >
      <button class="btn" style="padding: 0 20px;" @click="cargarProductos">Buscar</button>
      <button class="btn btn-outline" style="padding: 0 15px;" @click="limpiarFiltros">Limpiar</button>
    </div>

    <div v-if="cargando" class="loading">Cargando catálogo...</div>

    <div v-else class="grid">
      <div class="card" v-for="producto in productos" :key="producto.id">
        <div class="product-card-image">
          <img :src="producto.image_url || '/images/repuesto_defecto.png'" :alt="producto.name">
        </div>
        <h3>{{ producto.name }}</h3>
        <p class="vehiculo">SKU: {{ producto.sku }} | Ubicación: {{ producto.warehouse_location }}</p>
        <p class="price">{{ formatoPrecio(producto.price) }}</p>
        <button class="btn" style="width: 100%;" @click="$emit('ver-detalle', producto.id)">
          Ver Detalles
        </button>
      </div>
      <div v-if="productos.length === 0" class="no-results">
        No se encontraron repuestos para tu selección.
      </div>
    </div>
  </div>
</template>

<style scoped>
.filters { display: flex; gap: 12px; margin-bottom: 30px; background: var(--surface); padding: 16px; border-radius: 12px; box-shadow: var(--shadow); }
.filters input, .filters select { flex: 1; padding: 12px; border: 1px solid var(--border); border-radius: 8px; outline: none; font-size: 0.95rem; color: var(--text); transition: border-color 0.2s; background: var(--surface); }
.filters input:focus, .filters select:focus { border-color: var(--accent); }

.loading { text-align: center; font-size: 1.2rem; color: var(--text-light); margin-top: 50px; width: 100%; }
.no-results { grid-column: 1 / -1; text-align: center; color: var(--text-light); padding: 40px; font-size: 1.1rem; }

.grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(240px, 1fr)); gap: 24px; }
.card { background: var(--surface); padding: 20px; border-radius: 16px; text-align: center; box-shadow: var(--shadow); transition: transform 0.2s, box-shadow 0.2s; border: 1px solid transparent; display: flex; flex-direction: column; }
.card:hover { transform: translateY(-4px); box-shadow: 0 10px 15px -3px rgb(0 0 0 / 0.1); border-color: var(--border); }
.card:hover .product-card-image img { transform: scale(1.06); }

.product-card-image { height: 140px; background: var(--bg); border-radius: 10px; margin-bottom: 16px; display: flex; align-items: center; justify-content: center; overflow: hidden; border: 1px solid var(--border); position: relative; }
.product-card-image img { width: 100%; height: 100%; object-fit: cover; transition: transform 0.3s ease; }

.card h3 { font-size: 1.1rem; color: var(--primary); margin-bottom: 8px; height: 44px; overflow: hidden; display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical; text-align: left; }
.vehiculo { color: var(--text-light); font-size: 0.8rem; margin-bottom: 12px; text-align: left; }
.price { color: var(--accent); font-weight: 700; font-size: 1.4rem; margin-bottom: 16px; text-align: left; margin-top: auto; }
</style>