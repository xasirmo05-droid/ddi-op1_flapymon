<template>
  <div id="game">

    <!-- PANTALLA INICIAL -->
    <div v-if="screen === 'start'" class="start-screen">
      <h1>🎮 Flappymon</h1>
      <!-- jugadores  -->
    <div v-if="jugadores.length">
      <h3>Jugadores guardados:</h3>
      <ul>
        <li v-for="(j, i) in jugadores" :key="i">
          <b>{{j.puntos}}</b> - {{ j.nombre}} - <img :src="j.pokemon" width="40" />
        </li>
      </ul>
      <button @click="clearPlayers">🧹 Borrar tabla de puntos</button>
    </div>


      <!-- NOMBRE USUARIO -->
      <input
        type="text"
        v-model="userName"
        placeholder="Nombre del jugador"
      />

      <!-- SELECT POKÉMON -->
      <select v-model="selectedPokemon" @change="fetchPokemonData">
        <option disabled value="">Selecciona un Pokémon</option>
        <option
          v-for="poke in pokemonList"
          :key="poke.name"
          :value="poke.url"
        >
          {{ poke.name }}
        </option>
      </select>

      <!-- LOADING -->
      <p v-if="loading">🔄 Cargando Pokémon...</p>

      <!-- ERROR -->
      <p v-if="error" class="error">{{ error }}</p>

      <!-- INFO POKÉMON -->
      <div v-if="pokemon" class="pokemon-card">
        <h3>{{ pokemon.name }}</h3>
        <img :src="pokemon.sprite" alt="pokemon" />
      </div>

      <!-- START -->
      <button
        @click="startGame"
        :disabled="!canStart"
      >
        Empezar juego
      </button>
    </div>

    <!-- PANTALLA DEL JUEGO -->
     <div v-if="screen === 'game' && currentGame.nombre && currentGame.pokemon">
      <h2>Jugador: {{ currentGame.nombre }}</h2>
      <p>Pokémon: {{ currentGame.pokemon.name }}</p>
      <img :src="currentGame.pokemon.sprite" width="80" />
      <p>Puntos: {{ currentGame.puntos }}</p>
      <button @click="addPoint">➕ Punto</button>
      <button @click="gameOver">💀 Terminar partida</button>

     <!-- jugador -->
      <div id = "game-area">
        <img
          v-if="currentGame.value.pokemon"
          :src="currentGame.value.pokemon.sprite"
          width="80"
        >
      </div>
    </div>
</div>

</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

onMounted(() => {
  fetchPokemon()
})

const jugadores = ref(JSON.parse(localStorage.getItem('jugadores') || '[]'))
console.log(jugadores.value)

const screen = ref('start')

// Usuario
const userName = ref('')
const maxJugadores = ref(20)
// declarar el ref al inicio
const currentGame = ref({
  nombre: '',
  pokemon: null,
  puntos: 0,
  jugando: false
})

// Pokémon
const pokemonList = ref([])
const selectedPokemon = ref('')
const pokemon = ref(null)
const loading = ref(false)
const error = ref(null)

function addPoint() {
  if(currentGame.value) {
    currentGame.value.puntos++
  }
}

// Computed: validación limpia
const canStart = computed(() => {
  return userName.value !== '' && pokemon.value !== null
})

async function fetchPokemon() {
  fetch(`https://pokeapi.co/api/v2/pokemon?limit=15`)
    .then(response => response.json())
    .then(data => {
      pokemonList.value = data.results
    })
    .catch(error => {
      console.error(error)
      error.value = 'Error al cargar Pokémon'
    })
}

function startGame() {
  if (!userName.value || !pokemon.value) return

  currentGame.value = {
    nombre: userName.value,
    pokemon: pokemon.value,
    puntos: 0,
    jugando: true
  }
  screen.value = 'game'
}


// limpiar lista de jugadores
function clearPlayers() {
  localStorage.removeItem('jugadores')
  jugadores.value = []
}


function addPlayer() {
  if (!currentGame.value.nombre || !currentGame.value.pokemon) return

  // Datos del jugador
  jugadores.value.push({
    nombre: currentGame.value.nombre,
    pokemon: currentGame.value.pokemon.sprite,
    puntos: currentGame.value.puntos
  })

  // agregar jugador al array reactivo
  jugadores.value.sort((a, b) => b.puntos - a.puntos)
  jugadores.value = jugadores.value.slice(0, maxJugadores.value)
  localStorage.setItem('jugadores', JSON.stringify(jugadores.value))
  console.log(jugadores.value)
}


function gameOver() {
  addPlayer()

  // limpia formulario
  userName.value = ''
  selectedPokemon.value = ''
  pokemon.value = null
  currentGame.value = {
    nombre: '',
    pokemon: null,
    puntos: 0,
    jugando: false
  }

  // Cambiar pantalla
  screen.value = 'start'
}


console.log(localStorage.getItem('jugadores'))

async function fetchPokemonData() {
  fetch(selectedPokemon.value)
    .then(response => response.json())
    .then(data => {
      pokemon.value = {
        name: data.name,
        sprite: data.sprites.front_default
      }
    })
    .catch(error => {
      console.error(error)
      error.value = 'Error al cargar Pokémon'
    })
}

</script>

<style>
#game {
  width: 400px;
  height: 600px;
  border: 2px solid black;
  margin: auto;
  position: relative;
  overflow: hidden;
  font-family: Arial, sans-serif;
}

.start-screen {
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 12px;
  text-align: center;
}

.start-screen input,
.start-screen select {
  padding: 10px;
  font-size: 16px;
  width: 85%;
}

.start-screen button {
  margin-top: 10px;
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
}

.start-screen button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.pokemon-card img {
  width: 96px;
}

.error {
  color: red;
  font-size: 14px;
}

.game-screen {
  padding: 20px;
  text-align: center;
}
</style>
