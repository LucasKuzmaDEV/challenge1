<template>
  <div class="gif-list">
    <div class="title-container">
      <h1>Gifs do Giphy</h1>
      <div class="search-bar">
        <input
          v-model="searchTerm"
          type="text"
          placeholder="Digite sua pesquisa"
        />
        <button @click="searchGifs">Pesquisar</button>
      </div>
    </div>
    <div v-if="loading">Carregando...</div>
    <div v-if="gifs.length === 0 && !loading && searchTerm">
      Nenhum gif encontrado.
    </div>
    <div class="gif-items-container">
      <div
        v-for="gif in gifs"
        :key="gif.id"
        :class="{ selected: gif.id === selectedGifId }"
        class="gif-item"
        @click="selectGif(gif.id)"
      >
        <img :src="gif.images.fixed_height.url" alt="gif" />
      </div>
    </div>
    <div v-if="selectedGifId" class="overlay" @click="selectGif(null)"></div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      gifs: [],
      loading: false,
      offset: 0,
      limit: 20, // Número de gifs por requisição
      apiKey: "dV4nETEfJhtzMo64gh7UvonGFN96D4xW",
      searchTerm: "",
      selectedGifId: null, // ID do gif selecionado
    };
  },
  mounted() {
    this.fetchGifs();
    window.addEventListener("scroll", this.handleScroll);
  },
  beforeUnmount() {
    window.removeEventListener("scroll", this.handleScroll);
  },
  methods: {
    async fetchGifs() {
      if (this.loading) return; // Evita múltiplas chamadas enquanto estiver carregando
      this.loading = true;

      try {
        let url = `https://api.giphy.com/v1/gifs/trending`;
        if (this.searchTerm) {
          url = `https://api.giphy.com/v1/gifs/search`;
        }

        const response = await axios.get(url, {
          params: {
            api_key: this.apiKey,
            q: this.searchTerm,
            limit: this.limit,
            offset: this.offset,
          },
        });
        this.gifs = response.data.data;
        this.offset += this.limit;
        this.loading = false;
      } catch (error) {
        console.error("Erro ao buscar gifs:", error);
        this.loading = false;
      }
    },
    handleScroll() {
      if (window.innerHeight + window.scrollY >= document.body.offsetHeight) {
        this.fetchGifs();
      }
    },
    searchGifs() {
      this.offset = 0; // Reinicia o offset para a nova pesquisa
      this.gifs = []; // Limpa a lista de gifs
      this.fetchGifs();
    },
    selectGif(gifId) {
      this.selectedGifId = gifId === this.selectedGifId ? null : gifId;
    },
  },
};
</script>

<style scoped>
.gif-list {
  padding: 20px;
}

.title-container {
  margin-bottom: 20px;
}

.search-bar {
  display: flex;
  align-items: center;
}

.search-bar input {
  margin-right: 1rem;
}

.gif-items-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 5px;
}

.gif-item {
  margin-bottom: 5px;
  position: relative;
}

.gif-item img {
  width: 100%;
  height: auto;
}

.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(
    0,
    0,
    0,
    0.7
  ); /* Altere a opacidade conforme necessário */
  z-index: 999; /* Garante que a sobreposição esteja na parte superior */
  cursor: pointer; /* Altera o cursor para indicar que é clicável */
}

.gif-item.selected img {
  position: fixed;
  top: 50%;
  left: 50%;
  width: 500px;
  height: 500px;
  transform: translate(-50%, -50%); /* Centraliza o gif */
  z-index: 1000; /* Garante que o gif selecionado esteja na parte superior da sobreposição */
}
</style>
