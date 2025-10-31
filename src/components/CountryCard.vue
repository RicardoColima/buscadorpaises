<template>
  <div class="bg-white rounded-xl shadow-lg hover:shadow-xl transition-all duration-300 transform hover:-translate-y-1 overflow-hidden border border-gray-100">
    <!-- Imagen de la bandera -->
    <div class="relative h-48 overflow-hidden">
      <img 
        :src="country.flags.png" 
        :alt="`Bandera de ${country.name.common}`"
        class="w-full h-full object-cover"
      />
      <div class="absolute inset-0 bg-gradient-to-t from-black/20 to-transparent"></div>
    </div>
    
    <!-- Contenido de la card -->
    <div class="p-6">
      <!-- Nombre del país -->
      <h3 class="text-xl font-bold text-gray-900 mb-2 line-clamp-1">
        {{ country.name.common }}
      </h3>
      
      <!-- Información básica -->
      <div class="space-y-2 mb-4">
        <div class="flex items-center text-sm text-gray-600">
          <span class="w-2 h-2 bg-blue-500 rounded-full mr-2"></span>
          <span class="font-medium">Región:</span>
          <span class="ml-1">{{ country.region }}</span>
        </div>
        
        <div v-if="country.capital && country.capital[0]" class="flex items-center text-sm text-gray-600">
          <span class="w-2 h-2 bg-green-500 rounded-full mr-2"></span>
          <span class="font-medium">Capital:</span>
          <span class="ml-1">{{ country.capital[0] }}</span>
        </div>
        
        <div v-if="country.population" class="flex items-center text-sm text-gray-600">
          <span class="w-2 h-2 bg-purple-500 rounded-full mr-2"></span>
          <span class="font-medium">Población:</span>
          <span class="ml-1">{{ formatPopulation(country.population) }}</span>
        </div>
      </div>
      
      <!-- Botón para abrir modal -->
      <button 
        @click="openModal"
        class="w-full bg-gradient-to-r from-blue-600 to-purple-600 hover:from-blue-700 hover:to-purple-700 text-white font-semibold py-3 px-4 rounded-lg transition-all duration-300 transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 shadow-md hover:shadow-lg"
      >
        <span class="flex items-center justify-center">
          <svg class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path>
          </svg>
          Ver más información
        </span>
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'CountryCard',
  props: {
    country: {
      type: Object,
      required: true
    }
  },
  emits: ['open-modal'],
  methods: {
    formatPopulation(population) {
      if (population >= 1000000000) {
        return (population / 1000000000).toFixed(1) + 'B';
      } else if (population >= 1000000) {
        return (population / 1000000).toFixed(1) + 'M';
      } else if (population >= 1000) {
        return (population / 1000).toFixed(0) + 'K';
      }
      return population.toLocaleString();
    },
    
    openModal() {
      this.$emit('open-modal', this.country);
    }
  }
}
</script>

<style scoped>
.line-clamp-1 {
  display: -webkit-box;
  -webkit-line-clamp: 1;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>