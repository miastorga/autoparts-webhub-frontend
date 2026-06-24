<script setup>
import { ref } from 'vue'
import Catalog from './views/Catalog.vue'
import ProductDetail from './views/ProductDetail.vue'
import Login from './views/Login.vue'
import Admin from './views/Admin.vue'

const pantallaActiva = ref('catalog')

function irA(pantalla) {
  pantallaActiva.value = pantalla
  window.scrollTo(0, 0)
}
</script>

<template>
  <div class="dev-nav">
    <button @click="irA('catalog')">1. Catálogo</button>
    <button @click="irA('detail')">2. Detalle + IA</button>
    <button @click="irA('login')">3. Login Admin</button>
    <button @click="irA('admin')">4. Panel Admin</button>
  </div>

  <Catalog v-if="pantallaActiva === 'catalog'" @ver-detalle="irA('detail')" />
  <ProductDetail v-else-if="pantallaActiva === 'detail'" @volver="irA('catalog')" />
  <Login v-else-if="pantallaActiva === 'login'" @login="irA('admin')" />
  <Admin v-else-if="pantallaActiva === 'admin'" @logout="irA('login')" />
</template>