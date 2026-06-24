<script setup>
import { ref } from 'vue'
import Catalog from './views/Catalog.vue'
import ProductDetail from './views/ProductDetail.vue'
import Login from './views/Login.vue'
import Admin from './views/Admin.vue'

const pantallaActiva = ref('catalog')
const productoIdSeleccionado = ref(null)
const usuarioLogeado = ref(null)

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
</script>

<template>
  <div class="dev-nav">
    <button @click="irA('catalog')">1. Catálogo</button>
    <button @click="irA('detail', 1)">2. Detalle + IA</button>
    <button @click="irA('login')">3. Login Personal</button>
    <button v-if="usuarioLogeado" @click="irA('admin')">4. Panel de Operaciones</button>
  </div>

  <Catalog v-if="pantallaActiva === 'catalog'" @ver-detalle="(id) => irA('detail', id)" />
  <ProductDetail v-else-if="pantallaActiva === 'detail'" :partId="productoIdSeleccionado || 1" @volver="irA('catalog')" />
  <Login v-else-if="pantallaActiva === 'login'" @login="(u) => { usuarioLogeado = u; irA('admin') }" @volver="irA('catalog')" />
  <Admin v-else-if="pantallaActiva === 'admin' && usuarioLogeado" :user="usuarioLogeado" @logout="usuarioLogeado = null; irA('login')" />
</template>