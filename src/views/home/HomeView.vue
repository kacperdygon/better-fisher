<script setup lang="ts">
import { computed, onMounted, reactive, ref } from 'vue';
import type IFlashcard from '@/lib/IFlashcard.ts';

const maxGuesses = 3;

const flashcards = ref<IFlashcard[] | null>(null);
const currentFlashcard = ref<IFlashcard | null>(null);
const lastIndex = ref(-1);

onMounted(() => {
  loadData();
});

async function loadData() {
  const res = await fetch('/flashcards.json');
  flashcards.value = await res.json();

  flashcards.value?.forEach((flashcard) => {
    flashcard.guessedCount = 0;
  });
}

const isStarted = ref(false);
const counter = reactive({
  positive: 0,
  negative: 0,
});
const result = reactive({
  message: '',
  color: 'red',
});
const resultColor = computed(() => result.color);

function handleStart() {
  isStarted.value = true;
  assignNewFlashcard();
}

function assignNewFlashcard() {
  if (availableFlashcardsNumber.value > 0) {
    currentFlashcard.value = getRandomFlashcard(lastIndex.value);
  } else {
    alert('Wygrałeś!');
    window.location.reload();
  }
}

const availableFlashcardsNumber = computed(() => {
  let counter = 0;
  flashcards.value?.forEach((flashcard) => {
    if (flashcard.guessedCount < maxGuesses) {
      counter++;
    }
  });
  return counter;
});

function getRandomFlashcard(excludedIndex: number): IFlashcard | null {
  if (!flashcards.value) {
    alert(`Flashcards not loaded`);
    return null;
  }
  const arrayLength = flashcards.value.length;

  let randomIndex;
  do {
    randomIndex = Math.floor(Math.random() * arrayLength);
  } while (
    flashcards.value[randomIndex].guessedCount >= maxGuesses ||
    randomIndex === excludedIndex ||
    flashcards.value.length <= 1
  );

  lastIndex.value = randomIndex;
  return flashcards.value[randomIndex];
}

function handleSubmit(event: Event) {
  const input = event.target as HTMLInputElement;
  const userInput = input.value;
  if (userInput === currentFlashcard.value?.english) {
    counter.positive++;
    updateMessage(true);
    currentFlashcard.value.guessedCount++;
  } else {
    counter.negative++;
    updateMessage(false, userInput);
  }

  input.value = '';

  assignNewFlashcard();
}

function updateMessage(correct: boolean, userInput: string = '') {
  if (correct) {
    result.message = `Brawo! Poprawna odpowiedź: ${currentFlashcard.value?.english} - ${currentFlashcard.value?.polish}`;
    result.color = 'green';
  } else {
    result.message = `Źle! Poprawna odpowieź: ${currentFlashcard.value?.english}. Twa odpowiedź: ${userInput}`;
    result.color = 'red';
  }
}
</script>

<template>
  <div class="flex flex-col text-center items-center justify-center w-full h-screen">
    <main class="flex flex-col text-center items-center">
      <template v-if="!isStarted">
        <button @click="handleStart" v-show="!isStarted">Start</button>
      </template>
      <template v-if="isStarted">
        <p>{{ currentFlashcard?.polish }}</p>
        <input @keyup.enter="handleSubmit" class="border border-solid w-50" />
        <p v-show="result.message.length !== 0" class="result-message">{{ result.message }}</p>
        <p>Good ones: {{ counter.positive }}</p>
        <p>Bad ones: {{ counter.negative }}</p>
      </template>
    </main>
  </div>
</template>

<style scoped>
.result-message {
  color: v-bind(resultColor);
}
</style>
