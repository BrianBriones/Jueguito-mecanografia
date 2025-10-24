# vue-project

This template should help get you started developing with Vue 3 in Vite.

## Recommended IDE Setup

[VS Code](https://code.visualstudio.com/) + [Vue (Official)](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur).

## Recommended Browser Setup

- Chromium-based browsers (Chrome, Edge, Brave, etc.):
  - [Vue.js devtools](https://chromewebstore.google.com/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd) 
  - [Turn on Custom Object Formatter in Chrome DevTools](http://bit.ly/object-formatters)
- Firefox:
  - [Vue.js devtools](https://addons.mozilla.org/en-US/firefox/addon/vue-js-devtools/)
  - [Turn on Custom Object Formatter in Firefox DevTools](https://fxdx.dev/firefox-devtools-custom-object-formatters/)

## Customize configuration

See [Vite Configuration Reference](https://vite.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Compile and Minify for Production

```sh
npm run build
```


<script setup>
import { ref, computed, onUnmounted } from 'vue';

const estatDelJoc = ref({
  paraules: [
    { id: 1, text: 'component', estat: 'pendent' },
    { id: 2, text: 'reactivitat', estat: 'pendent' },
    { id: 3, text: 'javascript', estat: 'pendent' },
    { id: 4, text: 'framework', estat: 'pendent' },
    { id: 5, text: 'template', estat: 'pendent' }
  ],
  indexParaulaActiva: 0,
  textEntrat: '',
  estadistiques: [],
});

const paraulaActiva = computed(() => {
  return estatDelJoc.value.paraules[estatDelJoc.value.indexParaulaActiva];
});

let tempsIniciJoc = 0;
const tempsCronometre = ref(0);
let intervalCronometre = null;
let jocAcabat = ref(false);

function iniciarCronometre() {
  tempsIniciJoc = Date.now();
  tempsCronometre.value = 0;

  if (intervalCronometre) clearInterval(intervalCronometre);

  intervalCronometre = setInterval(() => {
    tempsCronometre.value = Date.now() - tempsIniciJoc;
  }, 100);
}

function aturarCronometre() {
  if (intervalCronometre) {
    clearInterval(intervalCronometre);
    intervalCronometre = null;
  }
}

function validarProgres() {
  // Empieza el cronómetro al escribir la primera letra
  if (estatDelJoc.value.textEntrat.length === 1 && tempsIniciJoc === 0) {
    iniciarCronometre();
  }

  // Si la palabra escrita es correcta
  if (estatDelJoc.value.textEntrat === paraulaActiva.value.text) {
    // Guardamos tiempo parcial para esa palabra
    const tempsParaula = Date.now() - tempsIniciJoc;

    estatDelJoc.value.estadistiques.push({
      paraula: paraulaActiva.value.text,
      temps: tempsParaula,
      errors: 0,
    });

    paraulaActiva.value.estat = 'completada';

    estatDelJoc.value.indexParaulaActiva++;
    estatDelJoc.value.textEntrat = '';

    // Si ya no quedan palabras
    if (estatDelJoc.value.indexParaulaActiva >= estatDelJoc.value.paraules.length) {
      jocAcabat.value = true;
      aturarCronometre();
      console.log('Joc acabat!', estatDelJoc.value.estadistiques);
    }
  }
}

onUnmounted(() => {
  aturarCronometre();
});
</script>

<template>
  <div class="game-engine">
    <div class="paraules-container">
      <div
        v-for="(paraula, index) in estatDelJoc.paraules"
        :key="paraula.id"
        class="paraula"
        :class="{ 'paraula-activa': index === estatDelJoc.indexParaulaActiva }"
      >
        {{ paraula.text }}
      </div>
    </div>

    <!-- Cronómetro siempre visible -->
    <div class="cronometre">
      Temps: {{ (tempsCronometre / 1000).toFixed(2) }} s
    </div>

    <input
      type="text"
      class="text-input"
      v-model="estatDelJoc.textEntrat"
      @input="validarProgres"
      :disabled="jocAcabat"
      placeholder="Comença a escriure..."
    />
  </div>
</template>

<style scoped>
.game-engine {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 20px;
}
.paraules-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.paraula {
  font-size: 24px;
  margin: 5px 0;
}
.paraula-activa {
  font-weight: bold;
  color: blue;
}
.text-input {
  font-size: 24px;
  padding: 10px;
  margin-top: 20px;
  border: 1px solid #ccc;
  border-radius: 4px;
}
.cronometre {
  font-size: 20px;
  margin-top: 15px;
  font-weight: bold;
  color: darkred;
}
</style>
