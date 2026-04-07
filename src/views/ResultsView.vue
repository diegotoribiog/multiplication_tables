<script setup>
import { useRoute, useRouter } from "vue-router";

// router to return to the start
const router = useRouter();

// we use route to read the data coming from the game
const route = useRoute();

// main stats
const attempts = route.query.attempts;
const correct = route.query.correct;
const incorrect = route.query.incorrect;

// question history (comes as a string from the URL, we parse it)
const historyData = route.query.history ? JSON.parse(route.query.history) : [];
</script>

<template>
  <div class="p-6 max-w-md mx-auto text-center">
    <!-- title -->
    <h1 class="text-3xl font-bold mb-6">Resultados</h1>

    <!-- main stats -->
    <div class="bg-gray-100 p-4 rounded mb-6">
      <p>Intentos: {{ attempts }}</p>
      <p class="text-green-600">Aciertos: {{ correct }}</p>
      <p class="text-red-600">Fallos: {{ incorrect }}</p>
    </div>

    <!-- record -->
    <h2 class="text-xl mb-3">Historial</h2>

    <div v-for="(item, i) in historyData" :key="i" class="text-sm mb-1">
      {{ item.question }} - {{ item.correct ? "✔️" : "❌" }} -
      {{ (item.time / 1000).toFixed(2) }}s
    </div>

    <!-- back button -->
    <RouterLink 
      to="/" 
      class="mt-6 inline-block bg-blue-500 text-white p-3 rounded hover:bg-blue-600"
    >
      Volver
    </RouterLink>
  </div>
</template>
