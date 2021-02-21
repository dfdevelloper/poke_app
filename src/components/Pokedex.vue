<template>
  <div>
    <v-row dense style="max-height: 80px">
      <v-col cols="6" style="height: 80px" class="mt-2 ml-2">
        <v-text-field
          v-model="search"
          color="red"
          label="Buscar com nome"
          append-icon="mdi-magnify"
          @input="filterPokemon"
          @keyup.enter="filterPokemon"
        ></v-text-field>
      </v-col>
    </v-row>

    <v-container fluid class="pokedex-box" @scroll="onScroll">
      <div v-if="filteredList.length" id="content">
        <v-card
          elevation="3"
          v-for="(pokemon, i) in filteredList"
          :key="`poke-${i}`"
          class="my-2"
        >
          <v-row dense justify="space-between">
            <v-col cols="4">
              <img
                @click="toggleShiny(pokemon, i)"
                height="150px"
                width="150px"
                :src="
                  pokemon.showShiny
                    ? pokemon.sprites.front_shiny
                    : pokemon.sprites.front_default
                "
              />
            </v-col>
            <v-col cols="6">
              <span class="text-capitalize"
                ><b>Nome:</b> {{ pokemon.name }} #{{ pokemon.order }}</span
              >
              <br />
              <span class="text-capitalize"
                ><b>Altura:</b> {{ pkHeight(pokemon.height) }}</span
              >
              <br />
              <v-chip
                class="mt-2 mr-2"
                v-for="(item, idx) in pokemon.types"
                :key="idx"
                :color="colors[item.type.name]"
                >{{ item.type.name }}</v-chip
              >
            </v-col>
          </v-row>
        </v-card>
        <v-card elevation="3" class="my-2" v-if="loading">
          <v-row dense justify="space-between">
            <v-col cols="12">
              <span>Carregando...</span>
            </v-col>
          </v-row>
        </v-card>
      </div>
      <v-row dense v-else>
        <v-col cols="12">
          <v-card elevation="3">
            <v-card-text
              >Nenhum Pokémon encontrado, tente buscar pelo nome
              completo</v-card-text
            >
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import Http from "../plugins/http";
import debounce from "lodash/debounce";
import Vue from "vue";
export default {
  name: "Pokedex",
  data: () => ({
    baseUrl: "https://pokeapi.co/api/v2/pokemon",
    search: "",
    nextPage: "limit=10",
    pokemons: [],
    filteredList: [],
    loading: false,
    btnLoading: false,
    selectedPokemon: {},
    evolutions: [],
    selectedPkmEvols: [],
    colors: {
      poison: "#ab6ac8",
      grass: "#63bb5b",
      ground: "#fdda96",
      fighting: "#ce4069",
      flying: "#8fa8dd",
      rock: "#d1c17d",
      bug: "#90c12c",
      ghost: "#5269ac",
      steel: "#5a8ea1",
      fire: "#db4249",
      water: "#4d90d5",
      electric: "#f3d23b",
      psychic: "#f97176",
      ice: "#6db5ba",
      dragon: "#0a6dc4",
      dark: "#5a5366",
      fairy: "#ec8fe6",
      normal: "#c6c6a7",
    },
  }),

  watch: {
    search(val) {
      !val.length && (this.filteredList = [...this.pokemons]);
    },
  },

  created() {
    this.getPokemons();
  },

  methods: {
    pkHeight(height = 0) {
      let h = height / 10;
      const n = (h + "").split(".");
      return `${n[0] > 0 ? n[0] + "m" : ""} ${n[1] ? n[1] + "cm" : ""}`;
    },
    async findPokemon(obj) {
      try {
        const infos = await fetch(obj.url).then((resp) => resp.json());
        const subInfos = await fetch(
          `https://pokeapi.co/api/v2/pokemon-species/${obj.name}`
        ).then((resp) => resp.json());
        this.selectedPokemon = { ...infos, ...subInfos };
        console.log(this.selectedPokemon);
      } catch (error) {
        console.log(error);
      }
    },
    toggleShiny(poke, index) {
      poke.showShiny = !poke.showShiny;
      Vue.set(this.filteredList, index, { ...poke });
    },
    onScroll(e) {
      const content = document.getElementById("content");
      const scrollH = e.target.offsetHeight + e.target.scrollTop;
      if (content && content.offsetHeight - scrollH <= 100 && !this.loading)
        this.getPokemons();
    },
    async getPokemons() {
      try {
        this.loading = true;
        debugger;
        const firstPokes = await Http.get(`pokemon?${this.nextPage}`).then(
          (resp) => resp.data
        );
        const splited = firstPokes.next.split("?");
        this.nextPage = splited[1];
        this.getFullInfo(firstPokes.results).then((pokemons) => {
          debugger;
          console.log(pokemons);
          this.pokemons.push(...pokemons);
          console.log("pok: ", this.pokemons);
          this.filteredList = [...this.pokemons];
          console.log("filtred: ", this.filteredList);
        });
      } catch (error) {
        console.log(error);
      }
      this.loading = false;
    },
    async getFullInfo(list = []) {
      const apiCalls = [];
      list.forEach((item) => {
        apiCalls.push(Http.get(`pokemon/${item.name}`));
      });
      const resps = await Promise.all(apiCalls).then((resp) =>
        resp.map((r) => r.data)
      );
      return resps;
    },

    filterPokemon: debounce(function () {
      this.filteredList = this.search.length
        ? this.pokemons.filter((p) => p.name.includes(this.search))
        : this.pokemons;
      if (!this.filteredList.length) {
        Http.get(`${this.baseUrl}/${this.search}`).then((resp) =>
          this.filteredList.push(resp.data)
        );
      }
    }, 500),
  },
};
</script>

<style scoped>
.pokedex-box {
  height: calc(100vh - 50px);
  overflow: auto;
}
</style>