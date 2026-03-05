<template>
  <div class="container">
    <div class="row mt-2">
      <!-- início lado esquerdo -->
      <div class="col mb-2">
        <div class="card palco">
          <div class="card-header"></div>

          <div class="card-body bg-pokebola bg-normal">
            <div class="pokemon">
              <transition
                mode="out-in"
                @after-enter="exibirEvolucoesTransicao"
                @before-leave="ocultarEvolucoesTransicao"
                enter-active-class="animate__animated animate__bounceIn"
                leave-active-class="animate__animated animate__bounceOut"
              >
                <img
                  v-if="pokemonSelecionado"
                  :key="pokemonSelecionado.id"
                  :src="`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/${pokemonSelecionado.id}.png`"
                />
              </transition>

              <div class="evolucoes">
                <transition-group name="fade">
                  <img
                    v-for="e in evolucoes"
                    :key="e"
                    v-show="exibirEvolucoes"
                    :src="`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/${e}.png`"
                  />
                </transition-group>
              </div>
            </div>
          </div>

          <div class="detalhes">
            <PokemonStats v-if="pokemonSelecionado" :stats="stats" />
          </div>

          <div class="card-footer">
            <nav class="nav nav-pills nav-fill">
              <!-- menu de navegação -->
            </nav>

            <div class="detalhes">
              <!-- exibe dados de acordo com o menu de navegação -->
            </div>
          </div>
        </div>
      </div>
      <!-- fim lado esquerdo -->

      <!-- início lado direito -->
      <div class="col mb-2 pokedex">
        <div class="row">
          <div class="col">
            <h1>Pokédex</h1>
          </div>
        </div>

        <div class="row">
          <div class="col">
            <select class="form-select" v-model="ordenacao">
              <option value="id-crescente">Id crescente</option>
              <option value="id-decrescente">Id decrescente</option>
              <option value="az">De A - Z</option>
            </select>
          </div>

          <div class="col">
            <input
              type="text"
              class="form-control"
              placeholder="Pesquisar pokémon"
              v-model="busca"
            />
          </div>
        </div>

        <div class="row">
          <div class="pokedex-catalogo">
            <!-- início listagem dinâmica -->
            <div
              v-for="p in pokemonsFiltrados"
              :key="p.id"
              :class="`cartao-pokemon bg-${p.tipo}`"
              @click="selecionarPokemon(p)"
            >
              <h1>{{ p.id }} {{ p.nome }}</h1>
              <span>{{ p.tipo }}</span>
              <div class="cartao-pokemon-img">
                <img
                  :src="`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/${p.id}.png`"
                />
              </div>
            </div>
            <!-- fim listagem dinâmica -->
          </div>
        </div>
      </div>
      <!-- fim lado direito -->
    </div>
  </div>
</template>

<script>
import PokemonStats from "@/components/pokemonStats.vue";
export default {
  name: "HomeView",

  components: {
    PokemonStats,
  },

  data() {
    return {
      exibir: false,
      exibirEvolucoes: false,
      pokemons: [],
      pokemonSelecionado: null,
      evolucoes: [],
      busca: "",
      ordenacao: "id-crescente",
    };
  },

  computed: {
    pokemonsFiltrados() {
      let lista = [...this.pokemons];

      if (this.busca) {
        lista = lista.filter((p) => p.nome.toLowerCase().includes(this.busca.toLowerCase()));
      }

      if (this.ordenacao === "id-crescente") {
        lista.sort((a, b) => a.id - b.id);
      }

      if (this.ordenacao === "id-decrescente") {
        lista.sort((a, b) => b.id - a.id);
      }

      if (this.ordenacao === "az") {
        lista.sort((a, b) => a.nome.localeCompare(b.nome));
      }

      return lista;
    },
  },

  mounted() {
    this.carregarPokemons();
  },

  methods: {
    exibirEvolucoesTransicao() {
      this.exibirEvolucoes = true;
    },

    ocultarEvolucoesTransicao() {
      this.exibirEvolucoes = false;
    },

    async selecionarPokemon(p) {
      const resp = await fetch(`https://pokeapi.co/api/v2/pokemon/${p.id}`);
      const dados = await resp.json();

      this.pokemonSelecionado = p;

      this.stats = dados.stats;

      this.exibir = true;

      await this.carregarEvolucoes(p.id);
    },

    async carregarPokemons() {
      const resposta = await fetch("https://pokeapi.co/api/v2/pokemon?limit=806");

      const dados = await resposta.json();

      const lista = await Promise.all(
        dados.results.map(async (p) => {
          const respPokemon = await fetch(p.url);
          const dadosPokemon = await respPokemon.json();

          let tipoApi = dadosPokemon.types[0].type.name;

          const tipos = {
            grass: "grama",
            fire: "fogo",
            water: "agua",
            bug: "inseto",
            normal: "normal",
            electric: "eletrico",
            poison: "veneno",
            ground: "terra",
            rock: "pedra",
            psychic: "psiquico",
            fighting: "lutador",
            ghost: "fantasma",
            ice: "gelo",
            dragon: "dragao",
            dark: "sombrio",
            steel: "aco",
            fairy: "fada",
            flying: "voador",
          };

          let tipo = tipos[tipoApi] || "normal";

          return {
            id: dadosPokemon.id,
            nome: dadosPokemon.name,
            tipo: tipo,
          };
        }),
      );

      this.pokemons = lista;
    },

    async carregarEvolucoes(id) {
      this.evolucoes = [];

      const respSpecies = await fetch(`https://pokeapi.co/api/v2/pokemon-species/${id}`);

      const species = await respSpecies.json();

      const respEvo = await fetch(species.evolution_chain.url);

      const evoData = await respEvo.json();

      let cadeia = evoData.chain;

      let lista = [];

      lista.push(cadeia.species.url.split("/")[6]);

      if (cadeia.evolves_to[0]) {
        lista.push(cadeia.evolves_to[0].species.url.split("/")[6]);

        if (cadeia.evolves_to[0].evolves_to[0]) {
          lista.push(cadeia.evolves_to[0].evolves_to[0].species.url.split("/")[6]);
        }
      }

      this.evolucoes = lista.filter((e) => e != id);
    },
  },
};
</script>

<style>
body {
  background-color: #dee3eb;
}
</style>

<style scoped>
@import "@/assets/css/animacoes.css";

.pokedex {
  padding: 20px;
  background-color: #ffffff;
  -webkit-box-shadow: 2px 2px 10px rgba(200, 200, 200, 0.77);
  -moz-box-shadow: 2px 2px 10px rgba(200, 200, 200, 0.77);
  box-shadow: 2px 2px 10px rgba(200, 200, 200, 0.77);
  border-radius: 10px;
}

.pokedex-catalogo {
  overflow: auto;
  display: flex;
  flex-wrap: wrap;
  height: 400px;
  width: 98%;
  margin-top: 10px;
}

.cartao-pokemon {
  position: relative;
  margin: 5px;
  width: 150px;
  height: 115px;
  cursor: pointer;
  border-radius: 5px;
  -webkit-box-shadow: 2px 2px 2px rgba(200, 200, 200, 0.77);
  -moz-box-shadow: 2px 2px 2px rgba(200, 200, 200, 0.77);
  box-shadow: 2px 2px 2px rgba(200, 200, 200, 0.77);
}

.cartao-pokemon h1 {
  color: #fff;
  font-size: 14px;
  margin: 5px 0px 0px 5px;
  padding: 0px;
}

.cartao-pokemon span {
  color: #fff;
  position: absolute;
  background: rgba(255, 255, 255, 0.3);
  font-size: 12px;
  margin: 10px 0px 0px 5px;
  padding: 5px 10px 5px 10px;
  border-radius: 25px;
}

.cartao-pokemon img {
  max-width: 60%;
  max-height: 60%;
  float: right;
}

.bg-grama {
  background-color: #2d8f78;
}

.bg-fogo {
  background-color: #e47373;
}

.bg-agua {
  background-color: #5a9ed2;
}

.bg-inseto {
  background-color: #26d3ab;
}

.bg-eletrico {
  background-color: #f7d02c;
}

.bg-veneno {
  background-color: #a33ea1;
}

.bg-terra {
  background-color: #e2bf65;
}

.bg-pedra {
  background-color: #b6a136;
}

.bg-psiquico {
  background-color: #f95587;
}

.bg-lutador {
  background-color: #c22e28;
}

.bg-fantasma {
  background-color: #735797;
}

.bg-gelo {
  background-color: #96d9d6;
}

.bg-dragao {
  background-color: #6f35fc;
}

.bg-sombrio {
  background-color: #705746;
}

.bg-aco {
  background-color: #b7b7ce;
}

.bg-fada {
  background-color: #d685ad;
}

.bg-voador {
  background-color: #a98ff3;
}

.bg-normal {
  background-color: #cecece;
}

.bg-pokebola {
  background-image: url("@/assets/imgs/pokebola.png");
  background-repeat: no-repeat;
  background-position: bottom right;
}

.palco {
  color: #fff;
  background-color: #333;
  -webkit-box-shadow: 2px 2px 10px rgba(230, 223, 223, 0.77);
  -moz-box-shadow: 2px 2px 10px rgba(230, 223, 223, 0.77);
  box-shadow: 2px 2px 10px rgba(230, 223, 223, 0.77);
  border-radius: 10px;
}

.pokemon {
  display: block;
  text-align: center;
  height: 215px;
}

.pokemon img {
  max-width: 100%;
  max-height: 100%;
}

.detalhes {
  margin: 20px 30px 20px 30px;
}

.evolucoes {
  position: absolute;
  top: 0px;
  right: 0px;
  height: 70px;
}
.evolucoes img {
  cursor: pointer;
  max-width: 100%;
  max-height: 100%;
  float: right;
}
</style>
