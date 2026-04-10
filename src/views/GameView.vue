<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";
import { useRoute, useRouter } from "vue-router";

// import simple-keyboard library
import Keyboard from "simple-keyboard";
import "simple-keyboard/build/css/index.css";

// read the route and navigate
const route = useRoute();
const router = useRouter();

// get the level from URL query
const level = Number(route.query.level);

// current question and correct answer
const question = ref(null);
const correctAnswer = ref(null);

// game timer (seconds)
const timeLeft = ref(60);
let timer = null;

// start time of current question
let questionStartTime = 0;

// last question to avoid repetition
let lastQuestion = "";

// game statistics
const attempts = ref(0);
const correct = ref(0);
const incorrect = ref(0);

// history of questions with correctness and time spent
const history = ref([]);

// game over state
const gameOver = ref(false);

// user input answer
const userAnswer = ref("");

// simple-keyboard instance
let keyboard = null;

// level configurations
const levels = {
  1: { tables: [1, 2, 10], range: [1, 10] },
  2: { tables: [3, 4, 5], range: [1, 10] },
  3: { tables: [6, 7, 8, 9], range: [1, 10] },
  4: {
    tables: [6, 7, 8, 11],
    range: [6, 9],
    special: {
      table: 11,
      range: [1, 10],
      probability: 0.3,
    },
  },
  5: { tables: [12, 13], range: [1, 10] },
};

// validate level and start game
onMounted(() => {
  if (!levels[level]) {
    router.push({ name: "home" });
    return;
  }

  // generate first question
  generateQuestion();

  // start game timer
  startTimer();

  // initialize simple-keyboard
  initKeyboard();
});

// destroy keyboard when leaving page
onBeforeUnmount(() => {
  if (keyboard) keyboard.destroy();
});

// get a random element from an array
function random(arr) {
  return arr[Math.floor(Math.random() * arr.length)];
}

// get a random number between min and max inclusive
function randomBetween(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

function generateQuestion() {
  const lvl = levels[level];

  let table;
  let multiplier;

  // handle special case if exists
  if (lvl.special && Math.random() < lvl.special.probability) {
    table = lvl.special.table;
    multiplier = randomBetween(lvl.special.range[0], lvl.special.range[1]);
  } else {
    table = random(lvl.tables.filter((t) => t !== lvl.special?.table));
    multiplier = randomBetween(lvl.range[0], lvl.range[1]);
  }

  let newQuestion = `${table} x ${multiplier}`;

  // avoid repeating question
  while (newQuestion === lastQuestion) {
    table = random(lvl.tables);
    multiplier = randomBetween(lvl.range[0], lvl.range[1]);
    newQuestion = `${table} x ${multiplier}`;
  }

  question.value = newQuestion;
  correctAnswer.value = table * multiplier;
  lastQuestion = newQuestion;

  // save start time for timing
  questionStartTime = performance.now();
}

// start the countdown timer
function startTimer() {
  timer = setInterval(() => {
    timeLeft.value--;

    if (timeLeft.value <= 0) {
      clearInterval(timer);
      gameOver.value = true;
    }
  }, 1000);
}

// handle answer submission
function submitAnswer() {
  if (gameOver.value) return; // cannot answer after game over

  const timeSpent = performance.now() - questionStartTime;
  const isCorrect = Number(userAnswer.value) === correctAnswer.value;

  attempts.value++;
  if (isCorrect) correct.value++;
  else incorrect.value++;

  history.value.push({
    question: question.value,
    correct: isCorrect,
    time: timeSpent,
  });

  userAnswer.value = "";
  updateKeyboard(""); // reset keyboard input
  generateQuestion();
}

// virtual keyboard input handler (simple-keyboard)
function handleInput(num) {
  if (!gameOver.value) {
    userAnswer.value += num;
    updateKeyboard(userAnswer.value);
  }
}

// delete last character
function handleDelete() {
  if (!gameOver.value) {
    userAnswer.value = userAnswer.value.slice(0, -1);
    updateKeyboard(userAnswer.value);
  }
}

// update keyboard input display
function updateKeyboard(value) {
  if (keyboard) keyboard.setInput(value);
}

// handle key press from simple-keyboard
function onKeyPress(button) {
  if (button === "{bksp}") {
    handleDelete();
  } else if (button === "{enter}") {
    submitAnswer();
  } else {
    handleInput(button);
  }
}

// initialize simple-keyboard
function initKeyboard() {
  keyboard = new Keyboard.default({
    onKeyPress: onKeyPress,
    layout: {
      default: ["1 2 3", "4 5 6", "7 8 9", "{bksp} 0 {enter}"],
    },

    display: {
      "{bksp}": "⌫",
      "{enter}": "OK",
    },
  });
}

// restart current level
function restartLevel() {
  timeLeft.value = 60;
  attempts.value = 0;
  correct.value = 0;
  incorrect.value = 0;
  history.value = [];
  gameOver.value = false;

  userAnswer.value = "";
  updateKeyboard("");

  generateQuestion();
  startTimer();
}
</script>

<template>
  <div class="p-6 max-w-md mx-auto text-center">
    <!-- timer -->
    <h1 class="text-xl font-semibold mb-4">
      {{ $t("game.time") }}: {{ timeLeft }}
    </h1>

    <!-- question -->
    <h2 class="text-3xl font-bold my-6">
      {{ question }}
    </h2>

    <!-- user answer -->
    <div class="text-3xl mb-6 bg-gray-100 p-4 rounded">
      {{ userAnswer || "_" }}
    </div>

    <!-- simple-keyboard container -->
    <div class="simple-keyboard"></div>

    <!-- statistics (always visible) -->
    <div class="mt-6 bg-gray-100 p-4 rounded text-center">
      <p>{{ $t("game.attempts") }}: {{ attempts }}</p>
      <p class="text-green-600">{{ $t("game.correct") }}: {{ correct }}</p>
      <p class="text-red-600">{{ $t("game.incorrect") }}: {{ incorrect }}</p>
    </div>

    <!-- results & history (only when game is over) -->
    <div v-if="gameOver" class="mt-6 bg-gray-100 p-4 rounded text-center">
      <h3 class="mt-2 font-semibold mb-2">{{ $t("game.history") }}</h3>
      <div
        v-for="(item, i) in history"
        :key="i"
        class="text-sm mb-1 flex justify-center items-center gap-2"
      >
        <span>{{ item.question }}</span>
        <span>{{ item.correct ? "✔️" : "❌" }}</span>
        <span>{{ (item.time / 1000).toFixed(2) }} s</span>
      </div>
    </div>

    <!-- buttons -->
    <div class="flex justify-center mt-4 gap-x-4">
      <button
        class="bg-blue-500 text-white p-2 rounded hover:bg-blue-600"
        @click="restartLevel"
      >
        {{ $t("game.restart") }}
      </button>

      <RouterLink
        :to="{ name: 'home' }"
        class="bg-blue-500 text-white p-2 rounded hover:bg-blue-600"
      >
        {{ $t("game.home") }}
      </RouterLink>
    </div>
  </div>
</template>
