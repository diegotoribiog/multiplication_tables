<script setup>

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

    <!-- level links -->
    <div class="flex flex-col gap-3">
      <RouterLink
        :to="{ name: 'game', query: { level: 1 } }"
        class="bg-blue-500 text-white p-3 rounded hover:bg-blue-600 block"
      >
        Nivel 1: Tablas del 1, 2 y 10
      </RouterLink>

      <RouterLink
        :to="{ name: 'game', query: { level: 2 } }"
        class="bg-blue-500 text-white p-3 rounded hover:bg-blue-600 block"
      >
        Nivel 2: Tablas del 3, 4 y 5
      </RouterLink>

      <RouterLink
        :to="{ name: 'game', query: { level: 3 } }"
        class="bg-blue-500 text-white p-3 rounded hover:bg-blue-600 block"
      >
        Nivel 3: Tablas del 6, 7, 8 y 9
      </RouterLink>

      <RouterLink
        :to="{ name: 'game', query: { level: 4 } }"
        class="bg-blue-500 text-white p-3 rounded hover:bg-blue-600 block"
      >
        Nivel 4: Tablas del 6, 7, 8 (por números de 6 a 9) y 11
      </RouterLink>

      <RouterLink
        :to="{ name: 'game', query: { level: 5 } }"
        class="bg-blue-500 text-white p-3 rounded hover:bg-blue-600 block"
      >
        Nivel 5: Tablas del 12 y 13
      </RouterLink>
    </div>
  </div>
</template>
