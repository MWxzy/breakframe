<script setup>
import { ref } from "vue";

const devices = [
  { name: "Mobile", width: 375, height: 667 },
  { name: "Tablet", width: 768, height: 1024 },
  { name: "Laptop", width: 1280, height: 800 },
  { name: "Desktop", width: 1440, height: 900 },
];

const url = ref("http://localhost:5173");
const input = ref(url.value);

function load() {
  let formatted = input.value.trim();

  if (!formatted.startsWith("http")) {
    formatted = "http://" + formatted;
  }

  url.value = formatted;
}
</script>

<template>
  <div class="app">
    <div class="header">
      <h1>Breakframe</h1>

      <input @keyup.enter="load"
        v-model="input"
        placeholder="Enter URL (e.g. http://localhost:5173)"
      />

      <button @click="load">GO</button>
    </div>

    <div class="grid">
      <div v-for="device in devices" :key="device.name" class="device">
        <p>{{ device.name }}</p>

        <div
          class="frame"
          :style="{
            width: device.width + 'px',
            height: device.height + 'px',
          }"
        >
          <iframe
            :src="url"
            :title="device.name"
            :width="device.width"
            :height="device.height"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.app {
  background: #0f0f0f;
  color: white;
  min-height: 100vh;
  font-family: sans-serif;
}

.header {
  text-align: center;
  padding: 20px;
}

input {
  padding: 10px;
  width: 300px;
  margin-right: 10px;
  border: none;
  outline: none;
}

button {
  padding: 10px 20px;
  cursor: pointer;
}

.grid {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  justify-content: center;
  padding: 20px;
}

.device {
  text-align: center;
}

.frame {
  border: 2px solid #333;
  overflow: hidden;
  background: black;
}

iframe {
  border: none;
}

.frame iframe {
  transform: scale(0.8);
  transform-origin: top left;
}
</style>