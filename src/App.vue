<template>
  <div class="min-h-screen bg-gray-50">
    <!-- Header -->
    <header class="bg-white shadow-sm">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-6">
        <h1 class="text-3xl font-bold text-gray-900 text-center">
          🌍 Buscador de Países
        </h1>
      </div>
    </header>

    <!-- Main Content -->
    <main class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
      <!-- Search Bar -->
      <div class="mb-8">
        <div class="max-w-md mx-auto">
          <input
            v-model="searchQuery"
            type="text"
            placeholder="Buscar país..."
            class="w-full px-4 py-3 border border-gray-300 rounded-lg shadow-sm focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-colors"
          />
        </div>
      </div>

      <!-- Loading State -->
      <div v-if="isLoading" class="text-center py-12">
        <div class="inline-block animate-spin rounded-full h-8 w-8 border-b-2 border-blue-500"></div>
        <p class="mt-4 text-gray-600">Cargando países...</p>
      </div>

      <!-- Error State -->
      <div v-else-if="hasError" class="text-center py-12">
        <div class="text-6xl mb-4">😔</div>
        <p class="text-gray-600 text-lg">{{ errorMessage }}</p>
      </div>

      <!-- No Results -->
      <div v-else-if="filteredCountries.length === 0 && !isLoading" class="text-center py-12">
        <div class="text-6xl mb-4">🔍</div>
        <p class="text-gray-600 text-lg">No se encontraron resultados</p>
        <p class="text-gray-500 mt-2">Intenta con otro término de búsqueda</p>
      </div>

      <!-- Countries Grid -->
      <div v-else>
        <div class="flex justify-center">
          <div 
            class="grid gap-8 justify-items-center"
            :class="{
              'grid-cols-1 max-w-sm': filteredCountries.length === 1,
              'grid-cols-1 md:grid-cols-2 max-w-4xl': filteredCountries.length === 2,
              'grid-cols-1 md:grid-cols-2 lg:grid-cols-3 max-w-6xl': filteredCountries.length === 3,
              'grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4': filteredCountries.length >= 4
            }"
          >
            <div
              v-for="country in filteredCountries"
              :key="country.name.common"
              class="transform transition-transform duration-200 w-full max-w-sm"
            >
              <CountryCard :country="country" @open-modal="openModal" />
            </div>
          </div>
         </div>
       </div>
     </main>

    <!-- Modal Global -->
    <CountryModal 
      v-if="selectedCountry"
      :country="selectedCountry" 
      :isOpen="isModalOpen" 
      @close="closeModal" 
    />
  </div>
</template>

<script>
import { ref, onMounted, watch } from 'vue'
import CountryCard from './components/CountryCard.vue'
import CountryModal from './components/CountryModal.vue'

export default {
  name: 'App',
  components: {
    CountryCard,
    CountryModal
  },
  setup() {
    // Estados reactivos
    const countries = ref([])
    const filteredCountries = ref([])
    const searchQuery = ref('')
    const isLoading = ref(false)
    const hasError = ref(false)
    const errorMessage = ref('')
    
    // Estados del modal
    const isModalOpen = ref(false)
    const selectedCountry = ref(null)



    // Función para mezclar array aleatoriamente
    const shuffleArray = (array) => {
      const shuffled = [...array]
      for (let i = shuffled.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]]
      }
      return shuffled
    }

    // Función para cargar todos los países
    const fetchAllCountries = async () => {
      isLoading.value = true
      hasError.value = false
      
      try {
        // Primera llamada - datos básicos
        const response1 = await fetch('https://restcountries.com/v3.1/all?fields=name,flags,region,capital,population,languages,currencies,translations,tld,independent')
        
        if (!response1.ok) {
          throw new Error('Error al cargar los países')
        }
        
        const basicData = await response1.json()
        
        // Segunda llamada - datos adicionales
        const response2 = await fetch('https://restcountries.com/v3.1/all?fields=name,subregion,area,latlng,landlocked,borders')
        
        if (!response2.ok) {
          throw new Error('Error al cargar datos adicionales')
        }
        
        const additionalData = await response2.json()
        
        // Tercera llamada - más datos adicionales
        const response3 = await fetch('https://restcountries.com/v3.1/all?fields=name,timezones,unMember,status,car')
        
        if (!response3.ok) {
          throw new Error('Error al cargar datos políticos')
        }
        
        const politicalData = await response3.json()
        
        // Cuarta llamada - códigos de país
        const response4 = await fetch('https://restcountries.com/v3.1/all?fields=name,cca2,cca3,ccn3,cioc')
        
        if (!response4.ok) {
          throw new Error('Error al cargar códigos de país')
        }
        
        const codesData = await response4.json()
        
        // Combinar los datos usando el nombre común como clave
        const combinedData = basicData.map(country => {
          const additional = additionalData.find(add => add.name.common === country.name.common)
          const political = politicalData.find(pol => pol.name.common === country.name.common)
          const codes = codesData.find(code => code.name.common === country.name.common)
          return {
            ...country,
            ...additional,
            ...political,
            ...codes
          }
        })
        
        const data = combinedData
        
        // Procesar países para usar nombres en español cuando estén disponibles
        const processedCountries = data.map(country => {
          // Intentar obtener el nombre en español
          let spanishName = country.name.common
          if (country.translations && country.translations.spa && country.translations.spa.common) {
            spanishName = country.translations.spa.common
          }
          
          return {
            ...country,
            name: {
              ...country.name,
              common: spanishName
            }
          }
        })
        
        // Mezclar países aleatoriamente y asignar
        const shuffledCountries = shuffleArray(processedCountries)
        countries.value = shuffledCountries
        filteredCountries.value = shuffledCountries
        
      } catch (error) {
        hasError.value = true
        errorMessage.value = 'Error al cargar los países. Intenta recargar la página.'
        console.error('Error fetching countries:', error)
      } finally {
        isLoading.value = false
      }
    }



    // Función para buscar países
    const searchCountries = async (query) => {
      if (!query.trim()) {
        filteredCountries.value = countries.value
        return
      }

      isLoading.value = true
      hasError.value = false
      
      try {
        // Primera llamada - datos básicos
        const response1 = await fetch(`https://restcountries.com/v3.1/name/${encodeURIComponent(query)}?fields=name,flags,region,capital,population,languages,currencies,translations,tld,independent`)
        
        if (!response1.ok) {
          if (response1.status === 404) {
            filteredCountries.value = []
            return
          } else {
            throw new Error('Error en la búsqueda')
          }
        }
        
        const basicData = await response1.json()
        
        // Segunda llamada - datos geográficos
        const response2 = await fetch(`https://restcountries.com/v3.1/name/${encodeURIComponent(query)}?fields=name,subregion,area,latlng,landlocked,borders`)
        const additionalData = response2.ok ? await response2.json() : []
        
        // Tercera llamada - datos políticos
        const response3 = await fetch(`https://restcountries.com/v3.1/name/${encodeURIComponent(query)}?fields=name,timezones,unMember,status,car`)
        const politicalData = response3.ok ? await response3.json() : []
        
        // Cuarta llamada - códigos de país
        const response4 = await fetch(`https://restcountries.com/v3.1/name/${encodeURIComponent(query)}?fields=name,cca2,cca3,ccn3,cioc`)
        const codesData = response4.ok ? await response4.json() : []
        
        // Combinar los datos usando el nombre común como clave
        const combinedData = basicData.map(country => {
          const additional = additionalData.find(add => add.name.common === country.name.common)
          const political = politicalData.find(pol => pol.name.common === country.name.common)
          const codes = codesData.find(code => code.name.common === country.name.common)
          return {
            ...country,
            ...additional,
            ...political,
            ...codes
          }
        })
        
        // Procesar países de búsqueda para usar nombres en español
        const processedSearchResults = combinedData.map(country => {
          let spanishName = country.name.common
          if (country.translations && country.translations.spa && country.translations.spa.common) {
            spanishName = country.translations.spa.common
          }
          
          return {
            ...country,
            name: {
              ...country.name,
              common: spanishName
            }
          }
        })
        
        filteredCountries.value = processedSearchResults
        
      } catch (error) {
        hasError.value = true
        errorMessage.value = 'Error al buscar países. Intenta nuevamente.'
        console.error('Error searching countries:', error)
      } finally {
        isLoading.value = false
      }
    }

    // Función de debounce
    const debounce = (func, delay) => {
      let timeoutId
      return (...args) => {
        clearTimeout(timeoutId)
        timeoutId = setTimeout(() => func.apply(null, args), delay)
      }
    }

    // Crear función de búsqueda con debounce
    const debouncedSearch = debounce(searchCountries, 300)

    // Funciones del modal
    const openModal = (country) => {
      selectedCountry.value = country
      isModalOpen.value = true
    }

    const closeModal = () => {
      isModalOpen.value = false
      selectedCountry.value = null
    }

    // Watcher para el campo de búsqueda
    watch(searchQuery, (newQuery) => {
      debouncedSearch(newQuery)
    })

    // Cargar países al montar el componente
    onMounted(() => {
      fetchAllCountries()
    })

    return {
      countries,
      filteredCountries,
      searchQuery,
      isLoading,
      hasError,
      errorMessage,
      isModalOpen,
      selectedCountry,
      openModal,
      closeModal
    }
  }
}
</script>