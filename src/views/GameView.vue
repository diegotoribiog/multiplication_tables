<script setup>
import { ref, onMounted } from "vue";
import { useRoute, useRouter } from "vue-router";
import VirtualKeyboard from "../components/VirtualKeyboard.vue";

// to read the level and navigate
const route = useRoute();
const router = useRouter();

// we get the level from the URL
const level = Number(route.query.level);

// current question and correct answer
const question = ref(null);
const correctAnswer = ref(null);

// gametime (60s)
const timeLeft = ref(60);
let timer = null;

// moment at which each question appears
let questionStartTime = 0;

// last question to avoid repetition
let lastQuestion = "";

// statistics
const attempts = ref(0);
const correct = ref(0);
const incorrect = ref(0);

// we keep a history of questions
const history = ref([]);

// level settings
const levels = {
  1: { tables: [1, 2, 10], range: [1, 10] },
  2: { tables: [3, 4, 5], range: [1, 10] },
  3: { tables: [6, 7, 8, 9], range: [1, 10] },
  4: { tables: [6, 7, 8], range: [6, 9], extra: [11] },
  5: { tables: [12, 13], range: [1, 10] },
};

// random number from an array
function random(arr) {
  return arr[Math.floor(Math.random() * arr.length)];
}

// random number between min and max
function randomBetween(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

// generate a new question
function generateQuestion() {
  const lvl = levels[level];

  let table;
  let multiplier;
  let newQuestion = "";

  do {
    // level 4 (has special logic)
    if (level === 4) {
      const useExtra = Math.random() < 0.3;

      if (useExtra) {
        table = 11;
        multiplier = randomBetween(1, 10);
      } else {
        table = random(lvl.tables);
        multiplier = randomBetween(lvl.range[0], lvl.range[1]);
      }
    } else {
      table = random(lvl.tables);
      multiplier = randomBetween(lvl.range[0], lvl.range[1]);
    }

    newQuestion = `${table} x ${multiplier}`;
  } while (newQuestion === lastQuestion); // avoid repeating

  question.value = newQuestion;
  correctAnswer.value = table * multiplier;

  // save last question
  lastQuestion = newQuestion;

  // save start time
  questionStartTime = performance.now();
}

// start the counter
function startTimer() {
  timer = setInterval(() => {
    timeLeft.value--;

    // the game ends when it reaches 0.
    if (timeLeft.value <= 0) {
      clearInterval(timer);

      // we sent the results
      router.push({
        path: "/results",
        query: {
          attempts: attempts.value,
          correct: correct.value,
          incorrect: incorrect.value,
          history: JSON.stringify(history.value),
        },
      });
    }
  }, 1000);
}

// user response
const userAnswer = ref("");

// when the user responds
function submitAnswer() {
  // we calculated how long it took
  const timeSpent = performance.now() - questionStartTime;

  const isCorrect = Number(userAnswer.value) === correctAnswer.value;

  attempts.value++;

  if (isCorrect) correct.value++;
  else incorrect.value++;

  // we saved information about the question
  history.value.push({
    question: question.value,
    correct: isCorrect,
    time: timeSpent,
  });

  // we clean and generate new
  userAnswer.value = "";
  generateQuestion();
}

// the game starts when the page loads
onMounted(() => {
  generateQuestion();
  startTimer();
});

// add number from virtual keyboard
function handleInput(num) {
  userAnswer.value += num;
}

// delete last number
function handleDelete() {
  userAnswer.value = userAnswer.value.slice(0, -1);
}
</script>

<template>
  <div class="p-6 max-w-md mx-auto text-center">
    <!-- time -->
    <h1 class="text-xl font-semibold mb-4">Tiempo: {{ timeLeft }}</h1>

    <!-- question -->
    <h2 class="text-3xl font-bold my-6">
      {{ question }}
    </h2>

    <!-- user answer -->
    <div class="text-3xl mb-6 bg-gray-100 p-4 rounded">
      {{ userAnswer || "_" }}
    </div>

    <!-- keyboard -->
    <VirtualKeyboard
      @input="handleInput"
      @delete="handleDelete"
      @submit="submitAnswer"
    />

    <!-- real-time stats -->
    <div class="mt-6 bg-gray-100 p-4 rounded text-left">
      <p>Intentos: {{ attempts }}</p>
      <p class="text-green-600">Aciertos: {{ correct }}</p>
      <p class="text-red-600">Fallos: {{ incorrect }}</p>
    </div>
  </div>
</template>
