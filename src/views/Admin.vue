<script setup>
import { ref } from 'vue'

defineEmits(['logout'])

const seccionActiva = ref('inventario')

const inventario = ref([
  { sku: '#001', producto: 'Pastillas de Freno', vehiculo: 'Toyota Yaris', stock: 5 },
  { sku: '#002', producto: 'Filtro de Aceite', vehiculo: 'Chevrolet Spark', stock: 2 },
])

function estiloStock(stock) {
  return stock > 3
    ? { background: '#dcfce7', color: '#166534' }
    : { background: '#fef9c3', color: '#854d0e' }
}
</script>

<template>
  <div class="screen">
    <div class="header">
      <h2 style="margin: 0;">Panel de Administración</h2>
      <button class="btn btn-outline logout" @click="$emit('logout')">Cerrar Sesión</button>
    </div>

    <div class="admin-layout">
      <div class="sidebar">
        <ul>
          <li :class="{ active: seccionActiva === 'inventario' }" @click="seccionActiva = 'inventario'">
            📦 Inventario
          </li>
          <li :class="{ active: seccionActiva === 'citas' }" @click="seccionActiva = 'citas'">
            📅 Citas Logísticas
          </li>
          <li :class="{ active: seccionActiva === 'ajustes' }" @click="seccionActiva = 'ajustes'">
            ⚙️ Ajustes del Sistema
          </li>
        </ul>
      </div>

      <div class="admin-content">
        <div class="content-header">
          <h3 style="color: var(--primary);">Gestión de Inventario</h3>
          <button class="btn">+ Agregar Producto</button>
        </div>

        <table>
          <thead>
            <tr>
              <th>SKU</th>
              <th>Producto</th>
              <th>Vehículo</th>
              <th>Stock</th>
              <th>Acciones</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="item in inventario" :key="item.sku">
              <td style="color: var(--text-light);">{{ item.sku }}</td>
              <td style="font-weight: 500;">{{ item.producto }}</td>
              <td>{{ item.vehiculo }}</td>
              <td>
                <span class="badge" :style="estiloStock(item.stock)">{{ item.stock }} Unid.</span>
              </td>
              <td>
                <a href="#" class="action-link">Editar</a>
                <a href="#" class="action-link delete">Eliminar</a>
              </td>
            </tr>
          </tbody>
        </table>
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
.admin-content { flex: 1; background: var(--surface); padding: 30px; border-radius: 16px; box-shadow: var(--shadow); }

.content-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }

table { width: 100%; border-collapse: collapse; margin-top: 20px; }
th { text-align: left; padding: 14px; border-bottom: 2px solid var(--border); color: var(--text-light); font-weight: 600; font-size: 0.9rem; text-transform: uppercase; letter-spacing: 0.5px; }
td { padding: 16px 14px; border-bottom: 1px solid var(--border); font-size: 0.95rem; }
tbody tr:hover { background: var(--bg); }
.badge { padding: 4px 8px; border-radius: 12px; font-size: 0.85rem; }
.action-link { color: var(--accent); text-decoration: none; font-weight: 500; margin-right: 10px; }
.action-link:hover { text-decoration: underline; }
.action-link.delete { color: #ef4444; }
</style>