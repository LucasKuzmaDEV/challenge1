<template>
  
  <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css" rel="stylesheet">

  <!-- Container principal -->
  <div class="bg-zinc-600 relative">
    
    <div class="max-w-screen-xl mx-auto max-h-screen bg-zinc-700 overflow-hidden">
      <!-- Título da aplicação -->
      <h1 class="title bg-zinc-900 py-4 font-sans font-bold text-white text-center text-4xl mb-5 rounded-md">GIFS BY GIPHY</h1>
      
      <!-- Barra de pesquisa -->
      <div class="flex mb-6 justify-center">
        <!-- Input para pesquisa -->
        <input v-model="searchTerm" @keyup.enter="searchGifs" type="text" placeholder="Pesquise aqui outros GIFs..." class="py-2 px-4 rounded-l-md border-black border-solid bg-zinc-500 focus:outline-none w-full max-w-xl cursor-white border-b-0" />
        <!-- Botão para acionar a pesquisa -->
        <button @click="searchGifs" class="bg-zinc-500 text-white py-2 px-4 rounded-r-md hover:bg-zinc-600 ml-2">Pesquisar</button>
      </div>
      
      <!-- Grid de gifs -->
      <div class="gif-items-container grid grid-cols-5 gap-4 p-5 max-h-screen overflow-y-auto justify-center" ref="gifContainer" @scroll="handleScroll">
        <div v-for="gif in gifs" :key="gif.id" :class="{ 'selected': selectedGifId === gif.id }" class="gif-item mb-5 relative shadow rounded-md overflow-hidden flex justify-center items-center bg-gray-300" @click="selectGif(gif.id)">
          <img :src="gif.images.fixed_height.url" alt="gif" class="w-full h-auto" />
        </div>
      </div>
      
      <!-- Exibe o gif selecionado em um overlay -->
      <div v-if="selectedGifId" class="overlay fixed top-0 left-0 w-full h-full bg-black bg-opacity-70 z-50 cursor-pointer flex justify-center items-center transition-opacity" :class="{ 'opacity-0': !selectedGifUrl, 'opacity-100': selectedGifUrl }">
        <button @click="selectGif(null)" class="absolute top-4 right-4 text-white bg-gray-900 rounded-md px-4 py-2 border border-white font-sans font-bold text-xl">
          <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" strokeWidth="1.5" stroke="currentColor" class="w-6 h-6">
            <path strokeLinecap="round" strokeLinejoin="round" d="M6 18 18 6M6 6l12 12" />
          </svg>
        </button>
        <img :src="selectedGifUrl" alt="gif" class="max-w-4xl max-h-4xl" @click="selectGif(null)">
      </div>
    </div>
    
    <!-- Seta pulsante para rolar para baixo -->
    <div class="absolute bottom-8 right-12 text-white">
      <i class="fas fa-arrow-down fa-3x animate-pulse"></i>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Gifs',
  data() {
    return {
      gifs: [], 
      errorMessage: '', 
      searchTerm: '', 
      selectedGifId: null, 
      selectedGifUrl: '', 
      offset: 0 
    };
  },
  created() {
    this.fetchGifs();
  },
  methods: {
    // Função para buscar gifs
    async fetchGifs() {
      try {
        let url = `https://api.giphy.com/v1/gifs/trending?api_key=dV4nETEfJhtzMo64gh7UvonGFN96D4xW&limit=${this.limit}&offset=${this.offset}`;
        if (this.searchTerm) {
          url = `https://api.giphy.com/v1/gifs/search?api_key=dV4nETEfJhtzMo64gh7UvonGFN96D4xW&q=${this.searchTerm}&limit=${this.limit}&offset=${this.offset}`;
        }
        // Faz a requisição para a API
        const response = await fetch(url);
        if (!response.ok) {
          throw new Error('Erro ao carregar GIFs');
        }
  
        const data = await response.json();
        
        this.gifs = this.gifs.concat(data.data);
        
        this.offset += this.limit;
      } catch (error) {
        
        this.errorMessage = error.message;
      }
    },
    // Função para pesquisar gifs
    searchGifs() {
      
      this.offset = 0;
      this.gifs = [];
      
      this.fetchGifs();
    },
    // Função para selecionar um gif
    selectGif(id) {
      
      const gif = this.gifs.find(gif => gif.id === id);
      if (gif) {
        // Define o ID e a URL do gif selecionado
        this.selectedGifId = gif.id;
        this.selectedGifUrl = gif.images.original.url; 
      } else {
        
        this.selectedGifId = null;
        this.selectedGifUrl = null;
      }
    },
    // Função para lidar com a rolagem do contêiner de gifs
    async handleScroll() {
      const container = this.$refs.gifContainer;
      
      if (container.scrollTop + container.clientHeight >= container.scrollHeight) {
        await this.fetchGifs();
      }
    }
  }
};
</script>


<style>

/* Estilizando a barra de rolagem */
.gif-items-container::-webkit-scrollbar {
  width: 8px; 
}

.gif-items-container::-webkit-scrollbar-track {
  background: #3F3F46; 
}

.gif-items-container::-webkit-scrollbar-thumb {
  background: #c4c4c4; 
}

.gif-items-container::-webkit-scrollbar-thumb:hover {
  background: #a0a0a0; 
}

</style>
