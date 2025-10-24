<script setup>
import { onMounted, ref } from 'vue';
import GameEngine from './components/gameEngine.vue';
import { io } from 'socket.io-client';

// Estat per controlar quina vista es mostra
const vistaActual = ref('salaEspera'); // 'salaEspera', 'lobby', 'joc'
// Estat per a la connexió
const nomJugador = ref('');
const jugadors = ref([]);
let socket = null;


onMounted(() => {
    socket = io('http://juego-meca.daw.inspedralbes.cat:28333');

    socket.on('updatePlayerList', (llistaDeJugadors) => {
    jugadors.value = llistaDeJugadors;
    });
});

function connectarAlServidor() {

  socket.emit('setPlayerName', { name: nomJugador.value });
  vistaActual.value = 'lobby';
}   
</script>

<template>
  <main class="app-container">
    <!-- VISTA 1: SALA D'ESPERA -->
    <div v-if="vistaActual === 'salaEspera'" class="vista-container">
      <h1 class="titol-joc">Type Racer Royale</h1>
      <input type="text" v-model="nomJugador" placeholder="Introdueix el teu nom" />
      <button @click="connectarAlServidor">Entra al Lobby</button>
    </div>

    <!-- VISTA 2: LOBBY -->
    <div v-else-if="vistaActual === 'lobby'" class="vista-container">
      <h2>Jugadors Connectats</h2>
      <ul class="llista-jugadors">
        <li v-for="jugador in jugadors" :key="jugador.id">{{ jugador.name }}</li>
      </ul>
      <div class="temps-selector">
        <label> Durada de la partida:</label>
        <select v-model="selectedTime">
          <option value="15">15 segons</option>
          <option value="30" selected>30 segons</option>
          <option value="60">60 segons</option>
        </select>
      </div>
      <button @click="vistaActual = 'joc'">Comença a Jugar!</button>
    </div>

    <!-- VISTA 3: JOC -->
    <div v-else-if="vistaActual === 'joc'" class="vista-container">
      <GameEngine :tempsMaxim="Number(selectedTime)" />
    </div>
  </main>
</template>

<style scoped>
.app-container {
  min-height: 100vh;
  background: radial-gradient(circle at top, #222, #000);
  display: flex;
  justify-content: center;
  align-items: center;
  color: white;
  font-family: 'JetBrains Mono', monospace;
}

.vista-container {
  text-align: center;
  background: rgba(20, 20, 20, 0.8);
  padding: 40px;
  border-radius: 10px;
  box-shadow: 0 0 15px rgba(255, 255, 0, 0.3);
  width: 400px;
}

.titol-joc {
  font-size: 2.2em;
  color: #ffcc00;
  margin-bottom: 20px;
}

input {
  padding: 10px;
  width: 80%;
  border: none;
  border-radius: 5px;
  margin-bottom: 20px;
  text-align: center;
}

button {
  background: #ffcc00;
  border: none;
  color: #111;
  padding: 10px 20px;
  border-radius: 6px;
  font-weight: bold;
  cursor: pointer;
  transition: 0.2s;
}

button:hover {
  background: #ffaa00;
}

.llista-jugadors {
  list-style: none;
  padding: 0;
  margin: 10px 0 20px;
}

.llista-jugadors li {
  background: #222;
  margin: 5px 0;
  padding: 6px;
  border-radius: 5px;
}

.temps-selector {
  margin-bottom: 20px;
}
</style>
