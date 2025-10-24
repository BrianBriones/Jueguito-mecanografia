<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue';

const props = defineProps({
  tempsMaxim: { type: Number, default: 30 },
  jugadors: { type: Array, default: () => [] }
});

const estatDelJoc = ref({
  // Llista de paraules a escriure. Cada paraula és un objecte.
  paraules: [
    { id: 1, text: 'component', estat: 'pendent' },
    { id: 2, text: 'reactivitat', estat: 'pendent' },
    { id: 3, text: 'javascript', estat: 'pendent' },
    { id: 4, text: 'framework', estat: 'pendent' },
    { id: 5, text: 'template', estat: 'pendent' }
  ],
  indexParaulaActiva: 0,
  textEntrat: '',
  estadistiques: []
});

const paraulaActiva = computed(
  () => estatDelJoc.value.paraules[estatDelJoc.value.indexParaulaActiva]
);

const jocFinalitzat = ref(false);

const tempsRestant = ref(props.tempsMaxim);
let intervalTemps = null;

// listener global
onMounted(() => {
  window.addEventListener('keydown', PressioTecla);
  iniciarTemporitzador();
});

onUnmounted(() => {
  window.removeEventListener('keydown', PressioTecla);
  clearInterval(intervalTemps);
});

// Temporitzador de compte enrere
function iniciarTemporitzador() {
  intervalTemps = setInterval(() => {
    tempsRestant.value--;
    if (tempsRestant.value <= 0) {
      tempsRestant.value = 0;
      finalitzarJoc();
    }
  }, 1000);
}

function finalitzarJoc() {
  clearInterval(intervalTemps);
  jocFinalitzat.value = true;
}

// teclat visual
const Teclat = ref([
  ['Q', 'W', 'E', 'R', 'T', 'Y', 'U', 'I', 'O', 'P'],
  ['A', 'S', 'D', 'F', 'G', 'H', 'J', 'K', 'L'],
  ['Z', 'X', 'C', 'V', 'B', 'N', 'M']
]);

const teclaPremuda = ref('');

function PressioTecla(event) {
  const tecla = event.key.toUpperCase();
  teclaPremuda.value = tecla;
  setTimeout(() => (teclaPremuda.value = ''), 100);
}

// FUNCIÓ VALIDAR PROGRES

// Variable per guardar el temps d'inici de cada paraula
let tempsIniciParaula = 0;

function iniciarCronometreParaula() {
  tempsIniciParaula = Date.now();
}

// Funció principal que s'executa a cada pulsació
function validarProgres() {
  const target = paraulaActiva.value.text;
  const input = estatDelJoc.value.textEntrat;

  if (input.length === 1 && tempsIniciParaula === 0) {
    iniciarCronometreParaula();
  }

  // si l’entrada no coincideix amb el prefix correcte
  if (!target.startsWith(input)) paraulaActiva.value.errors++;

  if (input === target) {
    const tempsTrigat = Date.now() - tempsIniciParaula;

    // Desem les estadístiques
    estatDelJoc.value.estadistiques.push({
      paraula: target,
      temps: tempsTrigat,
      errors: paraulaActiva.value.errors
    });

    paraulaActiva.value.estatDelJoc = 'completada';
    estatDelJoc.value.indexParaulaActiva++;
    estatDelJoc.value.textEntrat = '';
    tempsIniciParaula = 0;

    if (estatDelJoc.value.indexParaulaActiva >= estatDelJoc.value.paraules.length) {
      finalitzarJoc();
    }
  }
}

function getClasseLletra(indexLletra) {
  const correcte = paraulaActiva.value.text[indexLletra];
  const entrat = estatDelJoc.value.textEntrat[indexLletra];

  if (entrat === undefined) return '';
  if (entrat === correcte) return 'lletra-correcta';
  return 'lletra-incorrecta';
}

const tempsTotal = computed(
  () =>
    estatDelJoc.value.estadistiques.reduce((acc, item) => acc + item.temps, 0) /
    1000
);

const errorsTotals = computed(
  () =>
    estatDelJoc.value.estadistiques.reduce((acc, item) => acc + item.errors, 0)
);

// funcio per reiniciar el joc
function reiniciarJoc() {
  jocFinalitzat.value = false;
  estatDelJoc.value.indexParaulaActiva = 0;
  estatDelJoc.value.textEntrat = '';
  estatDelJoc.value.estadistiques = [];
  tempsRestant.value = props.tempsMaxim;
  estatDelJoc.value.paraules.forEach(p => {
    p.estat = 'pendent';
    p.errors = 0;
  });
  iniciarTemporitzador();
}
</script>

<template>
  <div class="game-engine">
    <!-- Temporitzador -->
    <div class="temporitzador">
      <div class="temps-text">{{ tempsRestant }}s</div>
      <div class="barra-temporitzador">
        <div
          class="progres"
          :style="{ width: (tempsRestant / props.tempsMaxim) * 100 + '%' }"
        ></div>
      </div>
    </div>

    <!-- Llista de jugadors connectats -->
    <div v-if="props.jugadors.length" class="jugadors-connectats">
      <h4>Jugadors connectats</h4>
      <ul>
        <li v-for="j in props.jugadors" :key="j.id">{{ j.name }}</li>
      </ul>
    </div>

    <div class="paraules-container" v-if="!jocFinalitzat">
      <div
        v-for="(paraula, index) in estatDelJoc.paraules"
        :key="paraula.id"
        class="paraula"
        :class="{ 'paraula-activa': index === estatDelJoc.indexParaulaActiva }"
      >
        <template v-if="index === estatDelJoc.indexParaulaActiva">
          <span
            v-for="(lletra, lletraIndex) in paraula.text.split('')"
            :key="lletraIndex"
            class="lletra"
            :class="getClasseLletra(lletraIndex)"
          >
            {{ lletra }}
          </span>
        </template>
        <template v-else>
          {{ paraula.text }}
        </template>
      </div>
    </div>

    <input
      type="text"
      class="text-input"
      v-model="estatDelJoc.textEntrat"
      @input="validarProgres"
      placeholder="Comença a escriure..."
    />

    <div v-if="jocFinalitzat" class="resultats">
      <h2>Resultats del Joc</h2>
      <p>Temps total: {{ tempsTotal }} segons</p>
      <p>Errors totals: {{ errorsTotals }}</p>
      <button @click="reiniciarJoc">Reiniciar joc</button>
    </div>

    <!-- teclat visual -->
    <div v-if="!jocFinalitzat" class="teclat-visual">
      <div
        v-for="(fila, filaIndex) in Teclat"
        :key="filaIndex"
        class="fila-teclat"
      >
        <div
          v-for="lletra in fila"
          :key="lletra"
          class="tecla"
          :class="{ 'tecla-premuda': teclaPremuda === lletra }"
        >
          {{ lletra }}
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/*  ESTRUCTURA GENERAL  */
.game-engine {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  min-height: 100vh;
  background-color: #1a1a1a;
  color: #e5e5e5;
  font-family: 'JetBrains Mono', monospace;
  padding-top: 60px;
}

/*  TEMPORITZADOR  */
.temporitzador {
  margin-bottom: 20px;
  text-align: center;
}

.temps-text {
  font-size: 20px;
  color: #ffcc00;
  margin-bottom: 8px;
}

.barra-temporitzador {
  width: 300px;
  height: 8px;
  background: #333;
  border-radius: 4px;
  overflow: hidden;
}

.progres {
  height: 100%;
  background: linear-gradient(90deg, #ffcc00, #ffaa00);
  transition: width 1s linear;
}

/*  JUGADORS CONNECTATS  */
.jugadors-connectats {
  background: rgba(40, 40, 40, 0.6);
  padding: 10px 20px;
  border-radius: 8px;
  margin-bottom: 20px;
}

.jugadors-connectats ul {
  list-style: none;
  padding: 0;
  margin: 5px 0 0;
  font-size: 14px;
  color: #ccc;
}

/*  RESULTATS  */
.resultats button {
  margin-top: 20px;
  background: #ffcc00;
  border: none;
  padding: 8px 16px;
  border-radius: 6px;
  cursor: pointer;
}

/*  PARAULES  */
.paraules-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  max-width: 800px;
  gap: 10px;
  font-size: 24px;
  line-height: 1.8;
  color: #666;
}

.paraula {
  user-select: none;
  transition: color 0.3s ease;
}

.paraula-activa {
  color: #ffcc00;
  font-weight: 600;
  transform: scale(1.05);
}

.lletra {
  transition: color 0.15s ease, transform 0.15s ease;
}

.lletra-correcta {
  color: #e5e5e5;
}

.lletra-incorrecta {
  color: #ff5555;
}

/*  TECLAT VISUAL  */
.teclat-visual {
  margin-top: 40px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 5px;
}

.fila-teclat {
  display: flex;
  gap: 5px;
}

.tecla {
  width: 40px;
  height: 40px;
  border-radius: 6px;
  background: #222;
  color: #bbb;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
  box-shadow: inset 0 -2px 0 #111;
  transition: all 0.1s ease;
}

.tecla-premuda {
  background: #ffcc00;
  color: #111;
  transform: scale(0.95);
}

/*  INPUT  */
.text-input {
  margin-top: 40px;
  background: transparent;
  border: none;
  border-bottom: 2px solid #333;
  font-size: 24px;
  color: #ffcc00;
  text-align: center;
  outline: none;
  padding: 10px;
  width: 400px;
}

.text-input::placeholder {
  color: #555;
}

/*  RESULTATS  */
h2 {
  color: #ffcc00;
  margin-bottom: 15px;
}

p {
  margin: 6px 0;
  color: #aaa;
}

/*  ANIMACIONS  */
.paraula-activa,
.tecla-premuda {
  transition: all 0.1s ease-in-out;
}

/*  RESPONSIVE  */
@media (max-width: 768px) {
  .barra-temporitzador {
    width: 200px;
  }

  .paraules-container {
    max-width: 90%;
    font-size: 18px;
    gap: 6px;
  }

  .text-input {
    width: 80%;
    font-size: 20px;
  }

  .tecla {
    width: 32px;
    height: 32px;
    font-size: 14px;
  }

  .jugadors-connectats {
    width: 90%;
    font-size: 12px;
  }
}

@media (max-width: 480px) {
  .game-engine {
    padding-top: 40px;
  }

  .temps-text {
    font-size: 18px;
  }

  .paraules-container {
    font-size: 16px;
  }

  .text-input {
    width: 90%;
    font-size: 18px;
  }

  .tecla {
    width: 28px;
    height: 28px;
    font-size: 12px;
  }

  .resultats button {
    width: 80%;
    font-size: 16px;
  }
}
</style>
