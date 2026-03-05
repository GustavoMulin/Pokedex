<template>
  <div class="pokemon-stats">
    <h4>Status</h4>

    <div v-for="s in stats" :key="s.stat.name" class="stat">
      <span class="nome">
        {{ formatarNome(s.stat.name) }}
      </span>

      <div class="barra">
        <div class="valor" :style="{ width: s.base_stat + '%' }"></div>
      </div>

      <span class="numero">
        {{ s.base_stat }}
      </span>
    </div>
  </div>
</template>

<script>
export default {
  name: "PokemonStats",

  props: {
    stats: {
      type: Array,
      required: true,
    },
  },

  methods: {
    formatarNome(nome) {
      const nomes = {
        hp: "HP",
        attack: "Ataque",
        defense: "Defesa",
        "special-attack": "Ataque Esp.",
        "special-defense": "Defesa Esp.",
        speed: "Velocidade",
      };

      return nomes[nome] || nome;
    },
  },
};
</script>

<style scoped>
.pokemon-stats {
  margin-top: 10px;
}

.stat {
  display: flex;
  align-items: center;
  margin-bottom: 6px;
}

.nome {
  width: 110px;
  font-size: 12px;
}

.barra {
  flex: 1;
  height: 8px;
  background: #ddd;
  border-radius: 10px;
  overflow: hidden;
}

.valor {
  height: 100%;
  background: #4caf50;
}

.numero {
  width: 35px;
  text-align: right;
  font-size: 12px;
}
</style>
