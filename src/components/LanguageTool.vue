<script setup>
import { ref } from 'vue';

const text = ref('');
const result = ref(null);
const detectedLanguage = ref(null);
const suggestions = ref([]);

const checkText = async () => {
  try {
    const response = await fetch('http://localhost:8010/v2/check', {
      method: 'POST',
      headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
      body: `text=${encodeURIComponent(text.value)}&language=auto`
    });

    const data = await response.json();
    result.value = data;

    // Extraer el idioma detectado correctamente
    if (data.detectedLanguage) {
      detectedLanguage.value = data.detectedLanguage;
    }
  } catch (error) {
    console.error('Error al conectar con LanguageTool:', error);
  }
};

const getSuggestions = async (word) => {
  try {
    const response = await fetch(`http://localhost:8010/v2/suggest?word=${encodeURIComponent(word)}&language=es`);
    const data = await response.json();
    suggestions.value = data.suggestions;
  } catch (error) {
    console.error('Error al obtener sugerencias:', error);
  }
};
</script>

<template>
  <div class="container">
    <h1>Corrector de Texto con LanguageTool</h1>
    
    <textarea v-model="text" rows="5" placeholder="Escribe algo..."></textarea>
    
    <div class="buttons">
      <button @click="checkText">Revisar Texto</button>
    </div>

    <div v-if="detectedLanguage">
      <h3>Idioma Detectado:</h3>
      <p>{{ detectedLanguage.name }} ({{ detectedLanguage.code }})</p>
    </div>

    <div v-if="result && result.matches.length > 0">
      <h3>Correcciones:</h3>
      <ul>
        <li v-for="(error, index) in result.matches" :key="index">
          <strong>Error:</strong> "{{ error.context.text.substring(error.offset, error.offset + error.length) }}" 
          <br />
          <strong>Sugerencias:</strong> {{ error.replacements.map(rep => rep.value).join(', ') || 'Sin sugerencias' }}
          <br />
          <small>{{ error.message }}</small>
        </li>
      </ul>
      <pre>{{ result }}</pre>
    </div>

    <div v-if="suggestions.length > 0">
      <h3>Sugerencias para Palabras Incorrectas:</h3>
      <ul>
        <li v-for="(suggestion, index) in suggestions" :key="index">{{ suggestion }}</li>
      </ul>
    </div>
  </div>
</template>

<style>
.container {
  max-width: 600px;
  margin: auto;
  text-align: center;
  font-family: Arial, sans-serif;
}

textarea {
  width: 100%;
  padding: 10px;
  font-size: 16px;
  border-radius: 5px;
  border: 1px solid #ccc;
}

.buttons {
  margin-top: 10px;
}

button {
  padding: 10px;
  margin: 5px;
  background: #007bff;
  color: white;
  border: none;
  cursor: pointer;
  border-radius: 5px;
}

button:hover {
  background: #0056b3;
}

h3 {
  margin-top: 20px;
  color: #333;
}

ul {
  text-align: left;
  padding: 0;
  list-style-type: none;
}

li {
  padding: 5px;
  border-bottom: 1px solid #ddd;
}
</style>
