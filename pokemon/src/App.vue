<template>
  <div class="app">
    <header class="header">
      <img src="/src/assets/pikachu.png" alt="Eevee y Pikachu" class="img1"/>
      <h1>Pokémon Info</h1>
      <img src="https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/54.png" alt="Psyduck" />
    </header>

    <section class="buscador">
      <input
        v-model="search"
        @keyup.enter="buscarPokemon"
        type="text"
        placeholder="Escribe nombre o número..."
      />
      <button @click="buscarPokemon">Buscar</button>
    </section>

    <main v-if="pokemon" class="contenedor">
      <!-- Sección izquierda -->
      <section class="col izq">
        <div class="pokemon-card">
          <h2>{{ capitalizar(pokemon.name) }}</h2>
          <img :src="pokemonImg" alt="Pokemon" />
          <p>Altura: {{ (pokemon.height / 10).toFixed(1) }} m</p>
          <p>Peso: {{ (pokemon.weight / 10).toFixed(1) }} kg</p>
        </div>
      </section>

      <!-- Sección central -->
      <section class="col centro">
        <h2 class="numero"># {{ pokemon.id }}</h2>

        <div class="tipos">
          <h3>Tipos del Pokémon</h3>
          <div class="tags">
            <span v-for="t in pokemon.types" :key="t.type.name" class="type">
              {{ t.type.name }}
            </span>
          </div>
        </div>

        <div class="debilidades" v-if="weaknesses.length">
          <h3>Debilidades del Pokémon</h3>
          <div class="tags">
            <span v-for="w in weaknesses" :key="w" class="weak">{{ w }}</span>
          </div>
        </div>
      </section>

      <!-- Sección derecha -->
      <section class="col der">
        <h3 class="titulo-stats">Estadísticas</h3>
        <div v-for="s in pokemon.stats" :key="s.stat.name" class="stat">
          <label>{{ capitalizar(s.stat.name) }}: {{ s.base_stat }}/255</label>
          <div class="bar">
            <div class="fill" :style="{ width: (s.base_stat / 255 * 100) + '%' }"></div>
          </div>
        </div>
      </section>
    </main>

    <p v-else class="mensaje">Busca un Pokémon por nombre o número para ver su información.</p>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";

const search = ref("ditto");
const pokemon = ref(null);
const pokemonImg = ref("");
const weaknesses = ref([]);

function capitalizar(str) {
  return str.charAt(0).toUpperCase() + str.slice(1);
}

async function buscarPokemon() {
  const query = search.value.trim().toLowerCase();
  if (!query) return;

  try {
    const res = await axios.get(`https://pokeapi.co/api/v2/pokemon/${query}`);
    pokemon.value = res.data;
    pokemonImg.value = res.data.sprites.other["official-artwork"].front_default;

    const tipos = res.data.types.map((t) => t.type.url);
    const debilidadesSet = new Set();

    for (const url of tipos) {
      const typeRes = await axios.get(url);
      const dobles = typeRes.data.damage_relations.double_damage_from;
      dobles.forEach((d) => debilidadesSet.add(d.name));
    }

    weaknesses.value = Array.from(debilidadesSet);
  } catch (error) {
    pokemon.value = null;
    weaknesses.value = [];
    alert("Pokémon no encontrado 😢");
  }
}

onMounted(buscarPokemon);
</script>

<style scoped>
/* 🔹 Reset general */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

/* 🔹 Estructura base */
html, body, #app {
  width: 100%;
  height: 100%;
  background-color: #f6f8fb;
  font-family: "Comic Sans MS", sans-serif;
  color: #222;
}

/* 🔹 App principal */
.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  text-align: center;
}

/* 🔹 Encabezado */
.header {
  display: flex;
  align-items: center;
  justify-content: space-around;
  width: 100%;
  background-color: #fff6d6;
  padding: 1rem 0;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.header h1 {
  font-size: 2.4rem;
  color: #c9a02e;
}

.img1{
  width: 50px;
  height: 50px;
}
/* 🔹 Buscador */
.buscador {
  margin: 1.5rem 0;
  display: flex;
  justify-content: center;
  gap: 0.5rem;
}

.buscador input {
  padding: 0.7rem 1rem;
  border-radius: 8px;
  border: 1px solid #aaa;
  font-size: 1rem;
  width: 260px;
}

.buscador button {
  background: #f2c94c;
  border: none;
  padding: 0.7rem 1.2rem;
  border-radius: 8px;
  cursor: pointer;
  font-weight: bold;
  color: #333;
  transition: background 0.2s;
}

.buscador button:hover {
  background: #e0b93c;
}

/* 🔹 Contenedor principal */
.contenedor {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  width: 90%;
  /* max-width: 1200px; */
  margin-top: 1rem;
  background: #fff;
  border-radius: 20px;
  box-shadow: 0 4px 18px rgba(0, 0, 0, 0.1);
  padding: 2rem;
  gap: 2rem;
  flex-wrap: wrap; /* Responsive */
}

/* 🔹 Columnas */
.col {
  flex: 1;
}

/* 🔹 Izquierda (tarjeta del Pokémon) */
.izq {
  display: flex;
  justify-content: center;
}

.pokemon-card {
  background: #d9b45d;
  padding: 1.5rem;
  border-radius: 15px;
  width: 100%;
  max-width: 900px;
  text-align: center;
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.15);
  font-size: 1.3rem;
}

/* 🔹 Imagen del Pokémon */
.pokemon-card img {
  width: 500px;
  height: 500px;
  object-fit: contain;
  transition: transform 0.3s ease;
}

.pokemon-card img:hover {
  transform: scale(1.05);
}

/* 🔹 Centro (info general) */
.centro {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  font-size: 1.8rem;
}

.numero {
  font-size: 5rem;
  color: #c9a02e;
  margin-bottom: 1rem;
}

.tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  justify-content: center;
  margin-bottom: 1rem;
}

.type {
  background: #eabf47;
  color: #fff;
  padding: 0.4rem 0.8rem;
  border-radius: 8px;
  text-transform: uppercase;
  font-weight: bold;
}

.weak {
  background: #7ec8e3;
  color: #fff;
  padding: 0.4rem 0.8rem;
  border-radius: 8px;
  text-transform: uppercase;
  font-weight: bold;
}

/* 🔹 Derecha (estadísticas) */
.der {
  text-align: left;
  min-width: 300px;
}

.titulo-stats {
  font-size: 3rem;
  color: #222;
  margin-bottom: 1rem;
}

.stat {
  margin-bottom: 2rem;
  font-weight: bold;
  font-size: 1.5rem;
}

.bar {
  width: 100%;
  height: 14px;
  background: #eee;
  border-radius: 6px;
  overflow: hidden;
}

.fill {
  height: 100%;
  background: #d9b45d;
  border-radius: 6px;
}

/* 🔹 Mensaje de error o sin resultados */
.mensaje {
  font-size: 1.2rem;
  margin-top: 2rem;
  color: #666;
}



</style>



