<script setup>
import { ref } from 'vue';

const text = ref('');
const result = ref(null);
const detectedLanguage = ref(null);

const checkText = async () => {
  try {
    const response = await fetch('http://localhost:8010/v2/check', {
      method: 'POST',
      headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
      body: `text=${encodeURIComponent(text.value)}&language=es`
    });

    const data = await response.json();
    result.value = data;

    // Extraer el idioma detectado correctamente
    if (data.language && data.language.detectedLanguage) {
      detectedLanguage.value = data.language.detectedLanguage;
    }
  } catch (error) {
    console.error('Error al conectar con LanguageTool:', error);
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
      <p><strong>Nombre:</strong> {{ detectedLanguage.name }}</p>
      <p><strong>Código:</strong> {{ detectedLanguage.code }}</p>
      <p><strong>Confianza:</strong> {{ (detectedLanguage.confidence * 100).toFixed(2) }}%</p>
    </div>

    <div v-if="result && result.matches.length > 0">
      <h3>Correcciones:</h3>
      <ul>
        <li v-for="(error, index) in result.matches" :key="index">
          <p><strong>Error:</strong> <span class="error-text">{{ error.context.text.substring(error.offset, error.offset + error.length) }}</span></p>
          <p><strong>Sugerencias:</strong> {{ error.replacements.map(rep => rep.value).join(', ') || 'Sin sugerencias' }}</p>
          <p><strong>Mensaje:</strong> {{ error.message }}</p>
          <p><strong>Tipo de error:</strong> {{ error.type.typeName }}</p>
          <p><strong>Categoría:</strong> {{ error.rule.category.name }}</p>
          <p><strong>Regla aplicada:</strong> {{ error.rule.description }}</p>
          <p><strong>Oración completa:</strong> <i>{{ error.sentence }}</i></p>
        </li>
      </ul>
      <pre>{{ result }}</pre>
    </div>
  </div>
</template>

<style>
.container {
  max-width: 700px;
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
  padding: 10px;
  border-bottom: 1px solid #ddd;
  background: #f8f8f8;
  margin-bottom: 10px;
  border-radius: 5px;
}

.error-text {
  color: red;
  font-weight: bold;
}
</style>
