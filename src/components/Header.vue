<script setup>
import { useRouter, useRoute } from "vue-router";

const router = useRouter();
const route = useRoute();

// to find out which section is active
function isActive(path) {
  return route.path === path;
}
</script>

<template>
  <header
    class="bg-blue-500 text-white p-4 flex justify-center gap-6 shadow-md"
  >
  
  <RouterLink to="/">Inicio</RouterLink>
  <RouterLink to="/tables">Tablas</RouterLink>
  
  </header>
</template>
