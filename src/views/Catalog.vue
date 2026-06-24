<script setup>
defineEmits(['ver-detalle'])

const productos = [
  { id: 1, nombre: 'Filtro de Aceite', vehiculo: 'Chevrolet Spark', precio: 8500, img: 'Filtro de Aceite' },
  { id: 2, nombre: 'Pastillas de Freno', vehiculo: 'Toyota Yaris', precio: 25000, img: 'Pastillas Freno' },
  { id: 3, nombre: 'Batería 12V 55Ah', vehiculo: 'Universal', precio: 65000, img: 'Batería 12V' },
]

function formatoPrecio(valor) {
  return '$' + valor.toLocaleString('es-CL')
}
</script>

<template>
  <div class="screen">
    <h2>Encuentra tu repuesto</h2>

    <div class="filters">
      <select>
        <option>Todas las Marcas</option>
        <option>Toyota</option>
        <option>Chevrolet</option>
      </select>
      <select>
        <option>Todos los Modelos</option>
        <option>Yaris</option>
        <option>Spark</option>
      </select>
      <input type="text" placeholder="¿Qué buscas? Ej. Pastillas, Filtro...">
      <button class="btn" style="padding: 0 30px;">Buscar</button>
    </div>

    <div class="grid">
      <div class="card" v-for="producto in productos" :key="producto.id">
        <div class="img-placeholder">{{ producto.img }}</div>
        <h3>{{ producto.nombre }}</h3>
        <p class="vehiculo">{{ producto.vehiculo }}</p>
        <p class="price">{{ formatoPrecio(producto.precio) }}</p>
        <button class="btn" style="width: 100%;" @click="$emit('ver-detalle', producto.id)">
          Ver Detalles
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.filters { display: flex; gap: 12px; margin-bottom: 30px; background: var(--surface); padding: 16px; border-radius: 12px; box-shadow: var(--shadow); }
.filters input, .filters select { flex: 1; padding: 12px; border: 1px solid var(--border); border-radius: 8px; outline: none; font-size: 0.95rem; color: var(--text); transition: border-color 0.2s; }
.filters input:focus, .filters select:focus { border-color: var(--accent); }

.grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(240px, 1fr)); gap: 24px; }
.card { background: var(--surface); padding: 20px; border-radius: 16px; text-align: center; box-shadow: var(--shadow); transition: transform 0.2s, box-shadow 0.2s; border: 1px solid transparent; }
.card:hover { transform: translateY(-4px); box-shadow: 0 10px 15px -3px rgb(0 0 0 / 0.1); border-color: var(--border); }
.img-placeholder { height: 140px; background: var(--bg); border-radius: 8px; margin-bottom: 16px; display: flex; align-items: center; justify-content: center; color: var(--text-light); font-size: 0.9rem; border: 1px dashed var(--border); }
.card h3 { font-size: 1.1rem; color: var(--primary); margin-bottom: 8px; }
.vehiculo { color: var(--text-light); font-size: 0.9rem; margin-bottom: 12px; }
.price { color: var(--accent); font-weight: 700; font-size: 1.4rem; margin-bottom: 16px; }
</style>