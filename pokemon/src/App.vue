<template>
  <div class="app">
    <header class="header" :style="headerStyle">
      <img src="/src/assets/pikachu.png" alt="Eevee y Pikachu" class="img1" />
      <img src="/src/assets/logo.png" alt="logo" class="logo"/>
      <img src="/src/assets/Psyduck.png" alt="Psyduck" class="img2"/>
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
      <section class="col izq">
        <div class="pokemon-card" :style="{ backgroundColor: fondoColor }">
          <h2>{{ capitalizar(pokemon.name) }}</h2>
          <img :src="pokemonImg" alt="Pokemon" />
          <p>Altura: <strong>{{ (pokemon.height / 10).toFixed(1) }} m</strong></p>
          <p>Peso: <strong>{{ (pokemon.weight / 10).toFixed(1) }} kg</strong></p>
        </div>
      </section>

      <section class="col centro">
        <h2 class="numero" :style="{ color: fondoColor }">
          # {{ pokemon.id.toString().padStart(3, '0') }}
        </h2>

        <div class="tipos">
          <h3>Tipos del Pokémon</h3>
          <div class="tags">
            <span
              v-for="t in pokemon.types"
              :key="t.type.name"
              class="type"
              :style="{ backgroundColor: typeColors[t.type.name] }"
            >
              {{ t.type.name.toUpperCase() }}
            </span>
          </div>
        </div>

        <div class="debilidades" v-if="weaknesses.length">
          <h3>Debilidades del Pokémon</h3>
          <div class="tags">
            <span
              v-for="w in weaknesses"
              :key="w"
              class="weak"
              :style="{ backgroundColor: typeColors[w] || '#999' }"
            >
              {{ w.toUpperCase() }}
            </span>
          </div>
        </div>
      </section>

      <section class="col der">
        <h3 class="titulo-stats">Estadísticas</h3>
        <div v-for="s in pokemon.stats" :key="s.stat.name" class="stat">
          <label>{{ capitalizar(s.stat.name) }}: {{ s.base_stat }}/255</label>
          <div class="bar">
            <div
              class="fill"
              :style="{
                width: (s.base_stat / 255) * 100 + '%',
                backgroundColor: fondoColor,
              }"
            ></div>
          </div>
        </div>
      </section>
    </main>

    <p v-else class="mensaje">
      Busca un Pokémon por nombre o número para ver su información.
    </p>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";

const search = ref("growlithe");
const pokemon = ref(null);
const pokemonImg = ref("");
const weaknesses = ref([]);
const fondoColor = ref("#d9b45d");

const headerStyle = ref({
  backgroundColor: "#fff6d6",
});

const typeColors = {
  normal: "#A8A77A",
  fire: "#EE8130",
  water: "#6390F0",
  electric: "#F7D02C",
  grass: "#7AC74C",
  ice: "#96D9D6",
  fighting: "#C22E28",
  poison: "#A33EA1",
  ground: "#E2BF65",
  flying: "#A98FF3",
  psychic: "#F95587",
  bug: "#A6B91A",
  rock: "#B6A136",
  ghost: "#735797",
  dragon: "#6F35FC",
  dark: "#705746",
  steel: "#B7B7CE",
  fairy: "#D685AD",
};

function capitalizar(str) {
  return str.charAt(0).toUpperCase() + str.slice(1);
}

function actualizarHeader(types) {
  if (!types || types.length === 0) {
    headerStyle.value.background = "#fff6d6";
    return;
  }

  if (types.length === 1) {
    const c1 = typeColors[types[0]] || "#fff6d6";
    headerStyle.value = {
      background: c1,
    };
  } else {
    const c1 = typeColors[types[0]] || "#fff6d6";
    const c2 = typeColors[types[1]] || "#fff6d6";

    headerStyle.value = {
      background: `linear-gradient(90deg, ${c1}, ${c2})`,
    };
  }
}

async function buscarPokemon() {
  const query = search.value.trim().toLowerCase();
  if (!query) return;

  try {
    const res = await axios.get(`https://pokeapi.co/api/v2/pokemon/${query}`);
    pokemon.value = res.data;
    pokemonImg.value =
      res.data.sprites.other["official-artwork"].front_default;

    const tipoPrincipal = res.data.types[0].type.name;
    fondoColor.value = typeColors[tipoPrincipal] || "#ccc";

    const listaTipos = res.data.types.map((t) => t.type.name);
    actualizarHeader(listaTipos);

    const tiposURL = res.data.types.map((t) => t.type.url);
    const debilidadesSet = new Set();

    for (const url of tiposURL) {
      const typeRes = await axios.get(url);
      const dobles = typeRes.data.damage_relations.double_damage_from;
      dobles.forEach((d) => debilidadesSet.add(d.name));
    }

    weaknesses.value = Array.from(debilidadesSet);
  } catch (error) {
    pokemon.value = null;
    weaknesses.value = [];
    alert("Pokémon no encontrado");
  }
}

onMounted(buscarPokemon);
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html,body {
  width: 100%;
  height: 100%;
  overflow: hidden;
  background-color: #f6f8fb;
  font-family: "Comic Sans MS", sans-serif;
  color: #222;
}

.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  text-align: center;
}

.header {
  display: flex;
  align-items: center;
  justify-content: space-around;
  width: 100%;
  padding: 1rem 0;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  transition: background 0.5s ease;
}

.header h1 {
  font-size: 2.4rem;
  color: white;
}

.img1 {
  width: 200px;
  height: 110px;
}

.logo{
  width: 400;
  height: 130px;
}

.img2{
  width: 130px;
  height: 130px;
}

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

.contenedor {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  width: 90%;
  background: #fff;
  border-radius: 20px;
  box-shadow: 0 4px 18px rgba(0, 0, 0, 0.1);
  padding: 1rem;
  gap: 2rem;
  flex-wrap: wrap;
}

.col {
  flex: 1;
}

.izq {
  display: flex;
  justify-content: center;
}

.pokemon-card {
  padding: 1.5rem;
  border-radius: 15px;
  width: 100%;
  max-width: 900px;
  text-align: center;
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.15);
  font-size: 1.3rem;
  transition: background-color 0.3s ease;
}

.pokemon-card img {
  width: 500px;
  height: 500px;
  object-fit: contain;
  transition: transform 0.3s ease;
}

.pokemon-card img:hover {
  transform: scale(1.05);
}

.centro {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  font-size: 1.8rem;
}

.numero {
  font-size: 5rem;
  margin-bottom: 1rem;
  transition: color 0.3s ease;
}

.tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  justify-content: center;
  margin-bottom: 1rem;
}

.type,
.weak {
  color: #fff;
  padding: 0.4rem 0.8rem;
  border-radius: 8px;
  text-transform: uppercase;
  font-weight: bold;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.15);
}

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
  border-radius: 6px;
  transition: width 0.3s ease, background-color 0.3s ease;
}

.mensaje {
  font-size: 1.2rem;
  margin-top: 2rem;
  color: #666;
}

@media (max-width: 1200px) {
  .logo {
    width: 300px;
    height: auto;
  }

  .pokemon-card img {
    width: 400px;
    height: 400px;
  }

  .titulo-stats {
    font-size: 2.4rem;
  }
}

@media (max-width: 992px) {
  .contenedor {
    flex-direction: column;
    align-items: center;
  }

  .numero {
    font-size: 4rem;
  }

  .pokemon-card img {
    width: 350px;
    height: 350px;
  }

  .stat {
    font-size: 1.2rem;
  }

  .titulo-stats {
    font-size: 2rem;
  }
}

@media (max-width: 768px) {
  .header {
    flex-direction: column;
    gap: 0.5rem;
  }

  .img1, .img2 {
    width: 120px;
    height: auto;
  }

  .logo {
    width: 260px;
    height: auto;
  }

  .pokemon-card {
    font-size: 1.1rem;
  }

  .pokemon-card img {
    width: 300px;
    height: 300px;
  }

  .numero {
    font-size: 3.2rem;
  }

  .titulo-stats {
    font-size: 1.8rem;
  }

  .buscador input {
    width: 200px;
  }
}

@media (max-width: 480px) {
  .logo {
    width: 200px;
  }

  .pokemon-card img {
    width: 250px;
    height: 250px;
  }

  .buscador input {
    width: 160px;
    font-size: 0.9rem;
  }

  .buscador button {
    font-size: 0.9rem;
    padding: 0.6rem 1rem;
  }

  .titulo-stats {
    font-size: 1.6rem;
  }

  .numero {
    font-size: 2.5rem;
  }
}
</style>
