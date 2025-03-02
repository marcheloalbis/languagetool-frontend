<script setup>
import { ref } from 'vue';

const text = ref('');
const result = ref(null);

const checkText = async () => {
  try {
    const response = await fetch('http://localhost:8010/v2/check', {
      method: 'POST',
      headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
      body: `text=${encodeURIComponent(text.value)}&language=es`
    });
    result.value = await response.json();
  } catch (error) {
    console.error('Error al conectar con LanguageTool:', error);
  }
};
</script>

<template>
  <div>
    <h2>Corrector de Texto</h2>
    <textarea v-model="text" rows="5" cols="50" placeholder="Escribe algo..."></textarea>
    <br />
    <button @click="checkText">Revisar Texto</button>
    <h3>Resultados:</h3>
    <pre>{{ result }}</pre>
  </div>
</template>

<style>
textarea {
  width: 100%;
  padding: 10px;
  font-size: 16px;
}
button {
  padding: 10px;
  margin-top: 10px;
  cursor: pointer;
}
</style>
