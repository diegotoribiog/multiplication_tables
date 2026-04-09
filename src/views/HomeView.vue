<script setup>
import { useRouter } from "vue-router";

// we use the router to move between pages
const router = useRouter();

// when the user chooses a level, we send them to the game
function selectLevel(level) {
  router.push({ name: 'game', query: { level } });
}
</script>

<template>
  <div class="p-6 max-w-md mx-auto text-center">
    <!-- title -->
    <h1 class="text-3xl font-bold mb-6">Selecciona nivel</h1>

    <!-- game description -->
    <p class="mb-6 text-gray-700">
      Elige un nivel y practica las tablas de multiplicar. Tienes 60 segundos
      para responder tantas preguntas como puedas. ¡Intenta acertar el máximo
      posible!
    </p>

    <!-- level buttons -->
    <div class="flex flex-col gap-3">
      <button
        class="bg-blue-500 text-white p-3 rounded hover:bg-blue-600"
        @click="selectLevel(1)"
      >
        Nivel 1: Tablas del 1, 2 y 10
      </button>

      <button
        class="bg-blue-500 text-white p-3 rounded hover:bg-blue-600"
        @click="selectLevel(2)"
      >
        Nivel 2: Tablas del 3, 4 y 5
      </button>

      <button
        class="bg-blue-500 text-white p-3 rounded hover:bg-blue-600"
        @click="selectLevel(3)"
      >
        Nivel 3: Tablas del 6, 7, 8 y 9
      </button>

      <button
        class="bg-blue-500 text-white p-3 rounded hover:bg-blue-600"
        @click="selectLevel(4)"
      >
        Nivel 4: Tablas del 6, 7, 8 (por números de 6 a 9) y 11
      </button>

      <button
        class="bg-blue-500 text-white p-3 rounded hover:bg-blue-600"
        @click="selectLevel(5)"
      >
        Nivel 5: Tablas del 12 y 13
      </button>
    </div>
  </div>
</template>
