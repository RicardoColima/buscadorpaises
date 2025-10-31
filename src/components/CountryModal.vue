<template>
  <!-- Teleport para renderizar el modal en el body -->
  <Teleport to="body">
    <div v-if="isOpen" class="fixed inset-0 z-[9999] overflow-y-auto">
      <!-- Overlay -->
      <div 
        class="fixed inset-0 bg-black bg-opacity-50 transition-opacity duration-300"
        @click="closeModal"
      ></div>
      
      <!-- Modal -->
      <div class="flex min-h-screen items-center justify-center p-4">
      <div 
        class="relative bg-white rounded-2xl shadow-2xl max-w-4xl w-full max-h-[90vh] overflow-hidden transform transition-all duration-300 scale-100"
        @click.stop
      >
        <!-- Header del Modal -->
        <div class="bg-gradient-to-r from-blue-600 to-purple-700 text-white p-6 relative">
          <button 
            @click="closeModal"
            class="absolute top-4 right-4 text-white hover:text-gray-200 transition-colors duration-200"
          >
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
            </svg>
          </button>
          
          <div class="flex items-center space-x-4">
            <img 
              :src="country.flags.png" 
              :alt="`Bandera de ${country.name.common}`"
              class="w-16 h-12 object-cover rounded-lg shadow-lg"
            />
            <div>
              <h2 class="text-2xl font-bold">{{ country.name.common }}</h2>
              <p v-if="country.name.official !== country.name.common" class="text-blue-100 text-sm">
                {{ country.name.official }}
              </p>
              <div class="flex items-center mt-2 space-x-2">
                <span class="bg-white bg-opacity-20 px-2 py-1 rounded text-xs font-mono">
                  {{ country.cca2 }}
                </span>
                <span class="text-blue-100 text-sm">{{ country.region }}</span>
              </div>
            </div>
          </div>
        </div>

        <!-- Pestañas -->
        <div class="border-b border-gray-200">
          <nav class="flex space-x-0 overflow-x-auto">
            <button
              v-for="tab in tabs"
              :key="tab.id"
              @click="activeTab = tab.id"
              :class="[
                'px-6 py-4 text-sm font-medium transition-all duration-200 border-b-2 whitespace-nowrap',
                activeTab === tab.id
                  ? 'border-blue-500 text-blue-600 bg-blue-50'
                  : 'border-transparent text-gray-500 hover:text-gray-700 hover:border-gray-300'
              ]"
            >
              <span class="mr-2">{{ tab.icon }}</span>
              {{ tab.name }}
            </button>
          </nav>
        </div>

        <!-- Contenido de las pestañas -->
        <div class="p-6 max-h-[60vh] overflow-y-auto">
          
          <!-- Pestaña General -->
          <div v-if="activeTab === 'general'" class="space-y-6">
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
              <div class="bg-gray-50 p-4 rounded-lg">
                <h4 class="font-semibold text-gray-900 mb-3 flex items-center">
                  <span class="text-xl mr-2">🏛️</span>
                  Información Básica
                </h4>
                <div class="space-y-2 text-sm">
                  <div v-if="country.capital && country.capital[0]">
                    <span class="font-medium text-gray-700">Capital:</span>
                    <span class="ml-2 text-gray-900">{{ country.capital[0] }}</span>
                  </div>
                  <div v-if="country.population">
                    <span class="font-medium text-gray-700">Población:</span>
                    <span class="ml-2 text-gray-900">{{ formatPopulation(country.population) }}</span>
                  </div>
                  <div>
                    <span class="font-medium text-gray-700">Región:</span>
                    <span class="ml-2 text-gray-900">{{ country.region }}</span>
                  </div>
                  <div v-if="country.subregion">
                    <span class="font-medium text-gray-700">Subregión:</span>
                    <span class="ml-2 text-gray-900">{{ country.subregion }}</span>
                  </div>
                </div>
              </div>
              
              <div class="bg-gray-50 p-4 rounded-lg">
                <h4 class="font-semibold text-gray-900 mb-3 flex items-center">
                  <span class="text-xl mr-2">📊</span>
                  Estadísticas
                </h4>
                <div class="space-y-2 text-sm">
                  <div v-if="country.area">
                    <span class="font-medium text-gray-700">Área:</span>
                    <span class="ml-2 text-gray-900">{{ formatArea(country.area) }}</span>
                  </div>
                  <div v-if="country.population && country.area">
                    <span class="font-medium text-gray-700">Densidad:</span>
                    <span class="ml-2 text-gray-900">{{ getDensity() }} hab/km²</span>
                  </div>
                  <div v-if="country.borders">
                    <span class="font-medium text-gray-700">Fronteras:</span>
                    <span class="ml-2 text-gray-900">{{ country.borders.length }} países</span>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Pestaña Geografía -->
          <div v-if="activeTab === 'geografia'" class="space-y-6">
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
              <div class="bg-green-50 p-4 rounded-lg border-l-4 border-green-400">
                <h4 class="font-semibold text-green-800 mb-3 flex items-center">
                  <span class="text-xl mr-2">🗺️</span>
                  Ubicación y Territorio
                </h4>
                <div class="space-y-3 text-sm">
                  <div v-if="country.area">
                    <span class="font-medium text-green-700">📏 Área total:</span>
                    <span class="text-green-800 ml-2">{{ formatArea(country.area) }}</span>
                  </div>
                  <div v-if="country.latlng && country.latlng.length === 2">
                    <span class="font-medium text-green-700">📍 Coordenadas:</span>
                    <span class="text-green-800 ml-2">{{ formatCoordinates(country.latlng) }}</span>
                  </div>
                  <div v-if="country.landlocked !== undefined">
                    <span class="font-medium text-green-700">🏝️ Acceso al mar:</span>
                    <span class="text-green-800 ml-2">{{ country.landlocked ? 'Sin salida al mar' : 'Con salida al mar' }}</span>
                  </div>
                </div>
              </div>
              
              <div class="bg-green-50 p-4 rounded-lg border-l-4 border-green-400">
                <h4 class="font-semibold text-green-800 mb-3 flex items-center">
                  <span class="text-xl mr-2">🔗</span>
                  Fronteras
                </h4>
                <div v-if="country.borders && country.borders.length > 0" class="space-y-2">
                  <p class="text-sm text-green-700">
                    <span class="font-medium">Países limítrofes:</span>
                    <span class="ml-2">{{ country.borders.length }} países</span>
                  </p>
                  <div class="flex flex-wrap gap-1">
                    <span 
                      v-for="border in country.borders.slice(0, 8)" 
                      :key="border"
                      class="bg-green-100 text-green-800 px-2 py-1 rounded text-xs font-mono"
                    >
                      {{ border }}
                    </span>
                    <span v-if="country.borders.length > 8" class="text-green-600 text-xs">
                      +{{ country.borders.length - 8 }} más
                    </span>
                  </div>
                </div>
                <div v-else class="text-sm text-green-700">
                  No tiene fronteras terrestres (isla o archipiélago)
                </div>
              </div>
            </div>
          </div>

          <!-- Pestaña Cultura -->
          <div v-if="activeTab === 'cultura'" class="space-y-6">
            <div class="grid grid-cols-1 gap-6">
              <div class="bg-blue-50 p-4 rounded-lg border-l-4 border-blue-400">
                <h4 class="font-semibold text-blue-800 mb-3 flex items-center">
                  <span class="text-xl mr-2">🗣️</span>
                  Idiomas
                </h4>
                <div v-if="getLanguages().length > 0" class="flex flex-wrap gap-2">
                  <span 
                    v-for="language in getLanguages()" 
                    :key="language"
                    class="bg-blue-100 text-blue-800 px-3 py-1 rounded-full text-sm"
                  >
                    {{ language }}
                  </span>
                </div>
                <div v-else class="text-sm text-blue-700">No hay información de idiomas disponible</div>
              </div>

              <div class="bg-blue-50 p-4 rounded-lg border-l-4 border-blue-400">
                <h4 class="font-semibold text-blue-800 mb-3 flex items-center">
                  <span class="text-xl mr-2">💰</span>
                  Monedas
                </h4>
                <div v-if="getCurrencies().length > 0" class="space-y-2">
                  <div 
                    v-for="currency in getCurrencies()" 
                    :key="currency.code"
                    class="bg-blue-100 p-3 rounded-lg"
                  >
                    <div class="flex items-center justify-between">
                      <span class="font-medium text-blue-900">{{ currency.name }}</span>
                      <span class="text-blue-700 font-mono">{{ currency.code }}</span>
                    </div>
                    <div class="text-2xl text-blue-800 mt-1">{{ currency.symbol }}</div>
                  </div>
                </div>
                <div v-else class="text-sm text-blue-700">No hay información de monedas disponible</div>
              </div>

              <div class="bg-blue-50 p-4 rounded-lg border-l-4 border-blue-400">
                <h4 class="font-semibold text-blue-800 mb-3 flex items-center">
                  <span class="text-xl mr-2">🕐</span>
                  Zonas Horarias
                </h4>
                <div v-if="country.timezones && country.timezones.length > 0" class="flex flex-wrap gap-2">
                  <span 
                    v-for="timezone in country.timezones" 
                    :key="timezone"
                    class="bg-blue-100 text-blue-800 px-3 py-1 rounded text-sm font-mono"
                  >
                    {{ timezone }}
                  </span>
                </div>
                <div v-else class="text-sm text-blue-700">No hay información de zonas horarias disponible</div>
              </div>
            </div>
          </div>

          <!-- Pestaña Política -->
          <div v-if="activeTab === 'politica'" class="space-y-6">
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
              <div class="bg-purple-50 p-4 rounded-lg border-l-4 border-purple-400">
                <h4 class="font-semibold text-purple-800 mb-3 flex items-center">
                  <span class="text-xl mr-2">🏛️</span>
                  Estado Político
                </h4>
                <div class="space-y-3 text-sm">
                  <div v-if="country.independent !== undefined">
                    <span class="font-medium text-purple-700">🗳️ Independiente:</span>
                    <span class="text-purple-800 ml-2">{{ country.independent ? 'Sí' : 'No' }}</span>
                  </div>
                  <div v-if="country.unMember !== undefined">
                    <span class="font-medium text-purple-700">🌐 Miembro ONU:</span>
                    <span class="text-purple-800 ml-2">{{ country.unMember ? 'Sí' : 'No' }}</span>
                  </div>
                  <div v-if="country.status">
                    <span class="font-medium text-purple-700">📋 Estado:</span>
                    <span class="text-purple-800 ml-2">{{ getStatusInSpanish(country.status) }}</span>
                  </div>
                </div>
              </div>
              
              <div class="bg-purple-50 p-4 rounded-lg border-l-4 border-purple-400">
                <h4 class="font-semibold text-purple-800 mb-3 flex items-center">
                  <span class="text-xl mr-2">🚗</span>
                  Información de Tránsito
                </h4>
                <div class="space-y-3 text-sm">
                  <div v-if="country.car && country.car.side">
                    <span class="font-medium text-purple-700">Lado de conducción:</span>
                    <span class="text-purple-800 ml-2">{{ country.car.side === 'right' ? 'Derecha' : 'Izquierda' }}</span>
                  </div>
                  <div v-if="country.car && country.car.signs && country.car.signs.length > 0">
                    <span class="font-medium text-purple-700">Señales de tráfico:</span>
                    <div class="flex flex-wrap gap-1 mt-1">
                      <span 
                        v-for="sign in country.car.signs" 
                        :key="sign"
                        class="bg-purple-100 text-purple-800 px-2 py-1 rounded text-xs font-mono"
                      >
                        {{ sign }}
                      </span>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Pestaña Digital -->
          <div v-if="activeTab === 'digital'" class="space-y-6">
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
              <div class="bg-orange-50 p-4 rounded-lg border-l-4 border-orange-400">
                <h4 class="font-semibold text-orange-800 mb-3 flex items-center">
                  <span class="text-xl mr-2">🌐</span>
                  Dominios de Internet
                </h4>
                <div v-if="country.tld && country.tld.length > 0" class="flex flex-wrap gap-2">
                  <span 
                    v-for="domain in country.tld" 
                    :key="domain"
                    class="bg-orange-100 text-orange-800 px-3 py-1 rounded text-sm font-mono"
                  >
                    {{ domain }}
                  </span>
                </div>
                <div v-else class="text-sm text-orange-700">No hay dominios disponibles</div>
              </div>
              
              <div class="bg-orange-50 p-4 rounded-lg border-l-4 border-orange-400">
                <h4 class="font-semibold text-orange-800 mb-3 flex items-center">
                  <span class="text-xl mr-2">🔢</span>
                  Códigos Internacionales
                </h4>
                <div class="space-y-3 text-sm">
                  <div>
                    <span class="font-medium text-orange-700">Código ISO (2 letras):</span>
                    <span class="text-orange-800 ml-2 font-mono">{{ country.cca2 }}</span>
                  </div>
                  <div>
                    <span class="font-medium text-orange-700">Código ISO (3 letras):</span>
                    <span class="text-orange-800 ml-2 font-mono">{{ country.cca3 }}</span>
                  </div>
                  <div v-if="country.ccn3">
                    <span class="font-medium text-orange-700">Código numérico:</span>
                    <span class="text-orange-800 ml-2 font-mono">{{ country.ccn3 }}</span>
                  </div>
                  <div v-if="country.cioc">
                    <span class="font-medium text-orange-700">Código COI:</span>
                    <span class="text-orange-800 ml-2 font-mono">{{ country.cioc }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Pestaña Curiosidades -->
          <div v-if="activeTab === 'curiosidades'" class="space-y-6">
            <div class="bg-yellow-50 p-4 rounded-lg border-l-4 border-yellow-400">
              <h4 class="font-semibold text-yellow-800 mb-3 flex items-center">
                <span class="text-xl mr-2">🤔</span>
                Datos Curiosos e Interesantes
              </h4>
              <div v-if="getCuriousFacts().length > 0" class="space-y-3">
                <div 
                  v-for="(fact, index) in getCuriousFacts()" 
                  :key="index"
                  class="bg-yellow-100 p-3 rounded-lg text-yellow-800 text-sm"
                >
                  <span class="font-medium">{{ index + 1 }}.</span> {{ fact }}
                </div>
              </div>
              <div v-else class="text-sm text-yellow-700">
                No hay datos curiosos disponibles para este país.
              </div>
            </div>
          </div>

        </div>
      </div>
      </div>
    </div>
  </Teleport>
</template>

<script>
export default {
  name: 'CountryModal',
  props: {
    country: {
      type: Object,
      required: true
    },
    isOpen: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      activeTab: 'general',
      tabs: [
        { id: 'general', name: 'General', icon: '📋' },
        { id: 'geografia', name: 'Geografía', icon: '🗺️' },
        { id: 'cultura', name: 'Cultura', icon: '🎭' },
        { id: 'politica', name: 'Política', icon: '🏛️' },
        { id: 'digital', name: 'Digital', icon: '💻' },
        { id: 'curiosidades', name: 'Curiosidades', icon: '🤔' }
      ]
    }
  },
  methods: {
    closeModal() {
      this.$emit('close');
    },
    
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

    formatArea(area) {
      return area.toLocaleString() + ' km²';
    },

    formatCoordinates(latlng) {
      const [lat, lng] = latlng;
      const latDir = lat >= 0 ? 'N' : 'S';
      const lngDir = lng >= 0 ? 'E' : 'O';
      return `${Math.abs(lat).toFixed(2)}°${latDir}, ${Math.abs(lng).toFixed(2)}°${lngDir}`;
    },

    getDensity() {
      if (this.country.population && this.country.area) {
        return (this.country.population / this.country.area).toFixed(1);
      }
      return 'N/A';
    },

    getLanguages() {
      if (!this.country.languages) return [];
      return Object.values(this.country.languages);
    },

    getCurrencies() {
      if (!this.country.currencies) return [];
      return Object.entries(this.country.currencies).map(([code, currency]) => ({
        code,
        name: currency.name,
        symbol: currency.symbol || code
      }));
    },

    getStatusInSpanish(status) {
      const statusMap = {
        'officially-assigned': 'Oficialmente asignado',
        'user-assigned': 'Asignado por usuario',
        'exceptionally-reserved': 'Excepcionalmente reservado',
        'transitionally-reserved': 'Transicionalmente reservado',
        'indeterminately-reserved': 'Indeterminadamente reservado'
      };
      return statusMap[status] || status;
    },

    getCuriousFacts() {
      const facts = [];
      
      if (this.country.population && this.country.area) {
        const density = (this.country.population / this.country.area).toFixed(1);
        facts.push(`Tiene una densidad poblacional de ${density} habitantes por kilómetro cuadrado`);
      }

      if (this.country.timezones && this.country.timezones.length > 1) {
        facts.push(`Abarca ${this.country.timezones.length} zonas horarias diferentes`);
      }

      if (this.country.borders && this.country.borders.length === 0 && !this.country.landlocked) {
        facts.push('Es una isla o archipiélago, completamente rodeado por agua');
      }

      if (this.country.landlocked) {
        facts.push('No tiene salida al mar, está completamente rodeado por tierra');
      }

      if (this.country.area && this.country.area > 10000000) {
        facts.push('Es uno de los países más grandes del mundo por superficie');
      }

      if (this.country.population && this.country.population > 100000000) {
        facts.push('Tiene más de 100 millones de habitantes');
      }

      if (this.country.borders && this.country.borders.length > 10) {
        facts.push(`Tiene fronteras con ${this.country.borders.length} países diferentes`);
      }

      if (this.country.car && this.country.car.side === 'left') {
        facts.push('Se conduce por el lado izquierdo de la carretera');
      }

      if (this.country.tld && this.country.tld.length > 1) {
        facts.push(`Tiene ${this.country.tld.length} dominios de internet diferentes`);
      }

      return facts;
    }
  },
  watch: {
    isOpen(newVal) {
      if (newVal) {
        this.activeTab = 'general';
        document.body.style.overflow = 'hidden';
      } else {
        document.body.style.overflow = '';
      }
    }
  },
  beforeUnmount() {
    document.body.style.overflow = '';
  }
}
</script>

<style scoped>
/* Animaciones personalizadas */
.modal-enter-active, .modal-leave-active {
  transition: opacity 0.3s ease;
}

.modal-enter-from, .modal-leave-to {
  opacity: 0;
}

.modal-enter-active .modal-content,
.modal-leave-active .modal-content {
  transition: transform 0.3s ease;
}

.modal-enter-from .modal-content,
.modal-leave-to .modal-content {
  transform: scale(0.9) translateY(-50px);
}
</style>