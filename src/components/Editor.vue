<template>
    <!-- Aquí aplicamos la clase según el valor de isDark -->
    <div class="lt-container" :class="isDark ? 'dark-mode' : 'light-mode'">
      <!-- ENCABEZADO SUPERIOR (similar a la primera imagen) -->
      <div class="lt-topbar">
        <div class="lt-dropdown">
            <span>Texto temporal ▼</span>
        </div>
        <!-- Botón para cambiar el tema -->
        <button class="theme-switcher" @click="toggleTheme">
            {{ isDark ? 'Claro' : 'Oscuro' }}
        </button>
        </div>

  
      <!-- CONTENIDO PRINCIPAL -->
      <div class="lt-main">
        <!-- COLUMNA IZQUIERDA: Texto y título -->
        <div class="lt-left">
          <h1 class="lt-title">Bienvenido al editor</h1>
          <!-- EJEMPLO: Textarea -->
          <textarea
            class="lt-textarea"
            v-model="text"
            @input="checkText"
            placeholder="Escribe o pega tu texto aquí..."
          ></textarea>
        </div>
  
        <!-- COLUMNA DERECHA: Panel de evaluación -->
        <div class="lt-right">
          <!-- Barra superior con progreso y toggles -->
          <div class="lt-eval-top">
            <div class="lt-progress-row">
              <div class="lt-progress-label">Evaluación del texto</div>
              <div class="lt-progress-bar">
                <div
                  class="lt-progress-fill"
                  :style="{ width: progressPercent + '%' }"
                ></div>
              </div>
              <div class="lt-progress-value">{{ progressPercent }}</div>
            </div>
  
            <div class="lt-toggle-row">
              <span>Objetivo</span>
              <button class="lt-obj-btn">Fijar un objetivo ▼</button>
            </div>
  
            <div class="lt-buttons-row">
              <button class="lt-correct-btn">
                Corregir {{ result.matches.length }}
              </button>
            <!-- Botón que muestra el total de palabras -->
            <button class="lt-paraphrase-btn">
            {{ wordCount }} palabras
            </button>

            </div>
          </div>
  
          <!-- LISTADO DE ERRORES -->
          <div class="lt-errors-list">
            <div
              v-for="(match, index) in result.matches"
              :key="index"
              class="lt-error-item"
            >
              <div class="lt-error-header" @click="toggleCollapse(index)">
                <span
                  class="lt-dot"
                  :class="dotColor(match.rule.issueType)"
                ></span>
  
                <div class="lt-error-type">
                  {{
                    match.context.text.slice(
                      match.context.offset,
                      match.context.offset + match.context.length
                    )
                  }}
                  -
                  {{ getErrorLabel(match.rule.issueType) }}
                </div>
                <div class="lt-error-summary">
                  {{ match.rule.description || 'Error detectado' }}
                </div>
                <div class="lt-arrow-indicator">
                  <span v-if="collapsedIndex === index">▲</span>
                  <span v-else>▼</span>
                </div>
              </div>
  
              <div v-if="collapsedIndex === index" class="lt-error-details">
                <p class="lt-error-msg">{{ match.message }}</p>
                <div class="lt-suggestions">
                  <strong>Sugerencias:</strong>
                  <span v-if="match.replacements.length">
                    {{ match.replacements.map(r => r.value).join(', ') }}
                  </span>
                  <span v-else>Sin sugerencias</span>
                </div>
                <div v-if="match.replacements.length" class="suggestions-container">
                  <button
                    v-for="(rep, i) in match.replacements"
                    :key="i"
                    class="suggestion-button"
                    @click="applySuggestion(match, rep.value)"
                  >
                    {{ rep.value }}
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import { ref,computed  } from 'vue';
  
  export default {
    name: 'LanguageToolClone',
    setup() {
      const text = ref('');
      const result = ref({ matches: [] });
      const collapsedIndex = ref(null);
  
      // Modo oscuro por defecto:
      const isDark = ref(true);
  
      // Botón para conmutar el tema
      const toggleTheme = () => {
        isDark.value = !isDark.value;
      };
  
      // Porcentaje de corrección (ejemplo)
      const progressPercent = ref(81);
  
      // Llamada a LanguageTool
      const checkText = async () => {
        try {
          const response = await fetch('http://localhost:8010/v2/check', {
            method: 'POST',
            headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
            body: `text=${encodeURIComponent(text.value)}&language=es`,
          });
          const data = await response.json();
          result.value = data;
        } catch (error) {
          console.error('Error al conectar con LanguageTool:', error);
        }
      };
  
      // Reemplazar palabra errónea con la sugerencia
      const applySuggestion = (match, suggestion) => {
        const offset = match.context.offset;
        const length = match.context.length;
        const before = text.value.slice(0, offset);
        const after = text.value.slice(offset + length);
  
        text.value = before + suggestion + after;
        checkText();
      };
  
      // Colapsar/expandir
      const toggleCollapse = (idx) => {
        collapsedIndex.value = collapsedIndex.value === idx ? null : idx;
      };
  
      // Etiquetas según issueType
      const getErrorLabel = (type) => {
        if (!type) return 'Error';
        switch (type.toLowerCase()) {
          case 'misspelling':
            return 'Ortografía';
          case 'grammar':
            return 'Error gramatical';
          case 'style':
            return 'Estilo';
          default:
            return 'Error';
        }
      };
  
      // Puntito de color
      const dotColor = (type) => {
        if (!type) return 'dot-default';
        switch (type.toLowerCase()) {
          case 'misspelling':
            return 'dot-pink';
          case 'grammar':
            return 'dot-yellow';
          case 'style':
            return 'dot-purple';
          default:
            return 'dot-default';
        }
      };
      const wordCount = computed(() => {
        // Dividir por espacios (u otros separadores) y filtrar vacíos
        return text.value
            .trim()
            .split(/\s+/)
            .filter(Boolean).length;
        });
      return {
        isDark,
        toggleTheme,
        wordCount,
        text,
        result,
        collapsedIndex,
        progressPercent,
  
        checkText,
        applySuggestion,
        toggleCollapse,
        getErrorLabel,
        dotColor,
      };
    },
  };
  </script>
  
  <style scoped>
  /* Estilos base (por defecto en modo oscuro) */
  .lt-container {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    font-family: Arial, sans-serif;
  }
  .lt-topbar {
  display: flex;
  align-items: center;
  justify-content: space-between; /* Separa a izquierda y derecha */
  height: 40px;
  padding: 0 20px;
  border-bottom: 1px solid #333;
  background-color: #1a1a1a;
}

  /* MODO OSCURO */
  .dark-mode {
    background-color: #1e1e1e;
    color: #fff;
  }
  .dark-mode .lt-topbar {
    background-color: #1a1a1a;
    border-bottom: 1px solid #333;
  }
  .dark-mode .lt-dropdown {
    color: #aaa;
  }
  
  /* MODO CLARO */
  .light-mode {
    background-color: #f9f9f9;
    color: #000;
  }
  .light-mode .lt-topbar {
    background-color: #eaeaea;
    border-bottom: 1px solid #ccc;
  }
  .light-mode .lt-dropdown {
    color: #333;
  }
  
  /* Botón para cambiar tema */
  .theme-switcher {
    margin-left: auto;
    padding: 4px 8px;
    background-color: #333;
    border: 1px solid #555;
    color: #aaa;
    border-radius: 4px;
    cursor: pointer;
  }
  .light-mode .theme-switcher {
    background-color: #ddd;
    color: #444;
    border: 1px solid #999;
  }
  
  /* CONTENEDOR PRINCIPAL: 2 columnas */
  .lt-main {
    flex: 1;
    display: flex;
  }
  
  /* COLUMNA IZQUIERDA (texto) */
  .lt-left {
    min-width: 500px;
    flex: 1;
    padding: 20px;
    box-sizing: border-box;
  }
  
  /* TÍTULO */
  .lt-title {
    font-size: 1.8rem;
    font-weight: 600;
    margin-bottom: 20px;
  }
  
  /* TEXTAREA */
  .lt-textarea {
    width: 100%;
    height: calc(100vh - 130px);
    border: 1px solid #444;
    border-radius: 5px;
    font-size: 16px;
    padding: 10px;
    resize: none;
    outline: none;
  }
  .dark-mode .lt-textarea {
    background-color: #2b2b2b;
    color: #fff;
  }
  .light-mode .lt-textarea {
    background-color: #fff;
    color: #000;
    border: 1px solid #ccc;
  }
  
  /* COLUMNA DERECHA (Panel correcciones) */
  .lt-right {
    min-width: 500px;
    flex: 1;
    display: flex;
    flex-direction: column;
    box-sizing: border-box;
    border-left: 1px solid #333;
    padding: 20px;
  }
  
  /* Modo claro override en el panel derecho */
  .light-mode .lt-right {
    border-left: 1px solid #ccc;
  }
  
  /* Barra superior (eval-top) */
  .lt-eval-top {
    padding-bottom: 20px;
    margin-bottom: 20px;
    border-bottom: 1px solid #333;
  }
  .light-mode .lt-eval-top {
    border-bottom: 1px solid #ccc;
  }
  
  /* Fila de progreso */
  .lt-progress-row {
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 10px;
  }
  .lt-progress-label {
    font-weight: bold;
  }
  .lt-progress-bar {
    flex: 1;
    height: 8px;
    background-color: #444;
    border-radius: 4px;
    position: relative;
  }
  .dark-mode .lt-progress-bar {
    background-color: #444;
  }
  .light-mode .lt-progress-bar {
    background-color: #ccc;
  }
  
  .lt-progress-fill {
    background-color: #0f62fe;
    height: 100%;
    border-radius: 4px;
    transition: width 0.3s ease;
  }
  .lt-progress-value {
    min-width: 30px;
    text-align: right;
  }
  
  /* Botón objetivo */
  .lt-obj-btn {
    background-color: #333;
    border: 1px solid #555;
    color: #aaa;
    padding: 4px 10px;
    border-radius: 4px;
    cursor: pointer;
  }
  .light-mode .lt-obj-btn {
    background-color: #ddd;
    border: 1px solid #999;
    color: #333;
  }
  
  /* Botones corregir / parafrasear */
  .lt-buttons-row {
    margin-top: 15px;
    display: flex;
    gap: 10px;
  }
  .lt-correct-btn,
  .lt-paraphrase-btn {
    flex: 1;
    font-weight: bold;
    border: none;
    padding: 8px 0;
    border-radius: 4px;
    cursor: pointer;
  }
  .lt-correct-btn {
    background-color: #0f62fe;
    color: #fff;
  }
  .lt-correct-btn:hover {
    background-color: #0053c9;
  }
  .lt-paraphrase-btn {
    background-color: #333;
    color: #fff;
    border: 1px solid #555;
  }
  .lt-paraphrase-btn:hover {
    background-color: #444;
  }
  
  /* Modo claro override */
  .light-mode .lt-paraphrase-btn {
    background-color: #ddd;
    color: #333;
    border: 1px solid #999;
  }
  .light-mode .lt-paraphrase-btn:hover {
    background-color: #ccc;
  }
  
  /* LISTADO DE ERRORES */
  .lt-errors-list {
    margin-top: 10px;
    overflow-y: auto;
    flex: 1;
  }
  .lt-error-item {
    border: 1px solid #444;
    border-radius: 5px;
    margin-bottom: 10px;
    background-color: #2b2b2b;
  }
  .light-mode .lt-error-item {
    background-color: #f0f0f0;
    border: 1px solid #ccc;
  }
  
  /* Encabezado del error */
  .lt-error-header {
    display: flex;
    align-items: center;
    cursor: pointer;
    padding: 8px 10px;
    border-bottom: 1px solid #444;
    position: relative;
  }
  .lt-error-header:hover {
    background-color: #3a3a3a;
  }
  .light-mode .lt-error-header {
    border-bottom: 1px solid #ccc;
  }
  .light-mode .lt-error-header:hover {
    background-color: #e2e2e2;
  }
  
  /* Punto de color (tipo de error) */
  .lt-dot {
    display: inline-block;
    width: 8px;
    height: 8px;
    border-radius: 50%;
    margin-right: 6px;
  }
  .dot-pink { background-color: #ff66cc; }
  .dot-yellow { background-color: #ffbc00; }
  .dot-purple { background-color: #9d4edd; }
  .dot-default { background-color: #ccc; }
  
  /* Título (\"Ortografía\", etc.) */
  .lt-error-type {
    font-weight: bold;
    margin-right: 8px;
  }
  
  /* Descripción breve */
  .lt-error-summary {
    color: #ccc;
    flex: 1;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
  .light-mode .lt-error-summary {
    color: #666;
  }
  
  /* Flecha */
  .lt-arrow-indicator {
    margin-left: 5px;
  }
  
  /* Detalles del error */
  .lt-error-details {
    padding: 10px;
    display: flex;
    flex-direction: column;
    gap: 8px;
    color: #ddd;
  }
  .light-mode .lt-error-details {
    color: #333;
  }
  
  .lt-error-msg {
    margin: 0;
  }
  
  /* Sugerencias */
  .lt-suggestions {
    font-size: 0.9rem;
    padding: 6px;
    border-radius: 4px;
    background-color: #1e1e1e;
    color: #ccc;
  }
  .light-mode .lt-suggestions {
    background-color: #f4f4f4;
    color: #555;
  }
  .suggestions-container {
    display: flex;
    gap: 8px;
    margin-top: 6px;
  }
  .suggestion-button {
    background-color: #0f62fe;
    color: #fff;
    border: none;
    border-radius: 4px;
    padding: 5px 8px;
    font-size: 0.9rem;
    cursor: pointer;
  }
  .suggestion-button:hover {
    background-color: #0053c9;
  }
  .light-mode .suggestion-button {
    background-color: #79a4f3;
    color: #333;
  }
  .light-mode .suggestion-button:hover {
    background-color: #0053c9;
  }
  </style>
  