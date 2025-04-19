<script setup>
import { ref, onMounted } from "vue";
import PocketBase from 'pocketbase'

const pb = new PocketBase('http://127.0.0.1:8090');
const hi = "my name is ";
let name = ref("");
let number = ref(0);
let capitalCities = ref([]);
const isTrue = ref(true);
const records = ref([]);
let todoInput = ref("");
const API_LINK =
  "https://www.e-solat.gov.my/index.php?r=esolatApi/takwimsolat&period=today&zone=SGR02";
let prayerTime = ref({});

const API_LINK_JOKE = "https://official-joke-api.appspot.com/random_joke";
let joke = ref({});

function increment() {
  number.value++;
}
let intervalId = null;

function startInterval() {
  intervalId = setInterval(increment, 1000);
}

function stopInterval() {
  clearInterval(intervalId);
  intervalId = null;
}

function addElement() {
  if (name.value.trim()) {
    capitalCities.value.push(name.value.trim());
    name.value = "";
  }
}

function getPrayerTime() {
  fetch(API_LINK)
    .then((result) => result.json())
    .then((data) => {
      prayerTime.value = data.prayerTime[0];
    });
}

function getJoke() {
  fetch(API_LINK_JOKE)
    .then((result) => result.json())
    .then((data) => {
      joke.value = data;
    });
}


onMounted(async () => {
  getPrayerTime();
  try{
    records.value = await pb.collection('todos').getFullList();
  } catch (error) {
    console.error("Error fetching records:", error);
  }
});

function createTodo(){
  const newTodo = {
    title: todoInput.value,
    isDone: false
  }
  pb.collection('todos').create(newTodo).then(res => {
    records.value.push(res);
    todoInput.value = "";
  })
}



</script>

<template>
  <div class="min-h-screen bg-purple-50 p-8 font-sans">
    <div class="max-w-4xl mx-auto space-y-8">

      <!-- Todo List Section -->
      <div class="bg-white p-6 rounded-2xl shadow-lg border-l-8 border-purple-500">
        <h2 class="text-2xl font-bold text-purple-700 mb-4 flex items-center gap-2">
          📝 Todo List
        </h2>
        <ul class="list-disc list-inside text-gray-800 space-y-1">
          <li v-for="record in records" :key="record.id">{{ record.title }}</li>
        </ul>
      </div>

      <!-- Insert Todo Section -->
      <div class="bg-white p-6 rounded-2xl shadow-lg border-l-8 border-purple-500">
        <h2 class="text-2xl font-bold text-purple-700 mb-4 flex items-center gap-2">
          ➕ Insert Todo
        </h2>
        <div class="flex gap-3">
          <input
            v-model="todoInput"
            placeholder="Enter todo"
            class="flex-1 border border-purple-300 px-4 py-2 rounded-lg focus:outline-none focus:ring-2 focus:ring-purple-400"
          />
          <button @click="createTodo" class="bg-purple-600 hover:bg-purple-700 text-white px-4 py-2 rounded-lg transition">
            Add Todo
          </button>
        </div>
      </div>

      <!-- Counter Section -->
      <div class="bg-white p-6 rounded-2xl shadow-lg border-l-8 border-purple-500">
        <h2 class="text-2xl font-bold text-purple-700 mb-4 flex items-center gap-2">
          🔢 Live Counter
        </h2>
        <p class="text-4xl font-semibold text-purple-800 mb-4">{{ number }}</p>
        <div class="flex gap-4">
          <button @click="startInterval" class="bg-purple-600 hover:bg-purple-700 text-white px-4 py-2 rounded-lg transition">Start</button>
          <button @click="stopInterval" class="bg-purple-400 hover:bg-purple-500 text-white px-4 py-2 rounded-lg transition">Stop</button>
        </div>
      </div>

      <!-- City List Section -->
      <div class="bg-white p-6 rounded-2xl shadow-lg border-l-8 border-purple-500">
        <h2 class="text-2xl font-bold text-purple-700 mb-4 flex items-center gap-2">
          🏙️ {{ hi }}
        </h2>
        <div class="flex items-center gap-3 mb-4">
          <input
            v-model="name"
            placeholder="Enter a city"
            class="w-full border border-purple-300 px-4 py-2 rounded-lg focus:outline-none focus:ring-2 focus:ring-purple-400"
          />
          <button @click="addElement" class="bg-purple-600 hover:bg-purple-700 text-white px-4 py-2 rounded-lg transition">Add</button>
        </div>
        <ol class="list-decimal list-inside text-gray-700 space-y-1">
          <li v-for="(city, index) in capitalCities" :key="index">{{ city }}</li>
        </ol>
        <div class="mt-4">
          <p v-if="isTrue" class="text-lg text-purple-600 font-medium">England is my city</p>
          <button @click="isTrue = !isTrue" class="mt-2 px-4 py-2 bg-purple-300 text-purple-900 rounded-lg hover:bg-purple-400 transition">
            Toggle
          </button>
        </div>
      </div>

      <!-- Prayer Time Section -->
      <div class="bg-white p-6 rounded-2xl shadow-lg border-l-8 border-purple-500">
        <h2 class="text-2xl font-bold text-purple-700 mb-4 flex items-center gap-2">
          🕌 Prayer Times
        </h2>
        <button @click="getPrayerTime" class="mb-4 px-4 py-2 bg-purple-600 text-white rounded-lg hover:bg-purple-700 transition">
          Refresh Prayer Time
        </button>
        <div class="overflow-x-auto rounded-lg border border-purple-200">
          <table class="min-w-full text-center">
            <thead class="bg-purple-100 text-purple-800">
              <tr>
                <th class="px-4 py-2">Fajr</th>
                <th class="px-4 py-2">Dhuha</th>
                <th class="px-4 py-2">Dhuhr</th>
                <th class="px-4 py-2">Asr</th>
                <th class="px-4 py-2">Maghrib</th>
                <th class="px-4 py-2">Isha</th>
              </tr>
            </thead>
            <tbody>
              <tr class="bg-white text-gray-700">
                <td class="px-4 py-2">{{ prayerTime.fajr }}</td>
                <td class="px-4 py-2">{{ prayerTime.dhuha }}</td>
                <td class="px-4 py-2">{{ prayerTime.dhuhr }}</td>
                <td class="px-4 py-2">{{ prayerTime.asr }}</td>
                <td class="px-4 py-2">{{ prayerTime.maghrib }}</td>
                <td class="px-4 py-2">{{ prayerTime.isha }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <!-- Joke Section -->
      <div class="bg-white p-6 rounded-2xl shadow-lg border-l-8 border-purple-500">
        <h2 class="text-2xl font-bold text-purple-700 mb-4 flex items-center gap-2">
          😂 Random Joke
        </h2>
        <button @click="getJoke" class="px-4 py-2 bg-purple-600 text-white rounded-lg hover:bg-purple-700 transition">
          Get a Joke
        </button>
        <div class="mt-4 text-left">
          <p class="text-lg font-medium text-gray-800">{{ joke.setup }}</p>
          <p class="italic text-purple-600 mt-1">{{ joke.punchline }}</p>
        </div>
      </div>

    </div>
  </div>
</template>

<style scoped></style>
