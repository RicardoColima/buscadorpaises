# 🌍 Buscador de Países - Aplicación Vue.js

🚀 **Demo**: [https://ricardocolima.github.io/buscadorpaises/](Ver Demo)


## 📋 Descripción del Proyecto

Como desarrollador frontend, he creado una aplicación web interactiva para explorar información detallada de países de todo el mundo. La aplicación utiliza Vue 3 con Composition API y está diseñada con un enfoque moderno, responsivo y centrado en la experiencia del usuario.

### 🎯 Objetivos del Proyecto
- Proporcionar una interfaz intuitiva para explorar países
- Mostrar información completa y organizada de cada país
- Implementar búsqueda en tiempo real con debounce
- Crear una experiencia visual atractiva y responsiva
- Utilizar nombres en español para mejor accesibilidad local

## ✨ Características Principales

### 🔍 Funcionalidades Core
- **Visualización aleatoria**: Los países se muestran en orden aleatorio en cada carga
- **Búsqueda inteligente**: Sistema de búsqueda con debounce (300ms) para optimizar rendimiento
- **Modal detallado**: Información completa organizada en 5 pestañas temáticas
- **Nombres en español**: Utiliza traducciones oficiales cuando están disponibles
- **Grid responsivo**: Adaptación automática según el número de resultados
- **Estados de carga**: Indicadores visuales para carga, error y sin resultados

### 🎨 Características de UI/UX
- **Diseño moderno**: Cards con efectos hover y transiciones suaves
- **Gradientes atractivos**: Uso de gradientes azul-púrpura en elementos clave
- **Iconografía consistente**: Emojis y iconos SVG para mejor comprensión visual
- **Responsividad completa**: Adaptación desde móvil hasta desktop
- **Accesibilidad**: Focus states y navegación por teclado

## 🛠️ Stack Tecnológico

### Frontend
- **Vue 3** (v3.4.15) - Framework principal con Composition API
- **Vite** (v5.0.12) - Build tool y dev server
- **Tailwind CSS** (v3.4.1) - Framework de utilidades CSS
- **JavaScript ES6+** - Sintaxis moderna

### Herramientas de Desarrollo
- **ESLint** - Linting de código
- **PostCSS** - Procesamiento de CSS
- **Autoprefixer** - Compatibilidad de navegadores

### API Externa
- **REST Countries API v3.1** - Fuente de datos de países


## 🧩 Arquitectura de Componentes

### App.vue - Componente Principal
**Responsabilidades:**
- Gestión del estado global de la aplicación
- Manejo de la lógica de búsqueda y filtrado
- Coordinación entre componentes hijos
- Gestión de estados de carga y error

**Estados Reactivos:**
```javascript
const countries = ref([])           // Todos los países cargados
const filteredCountries = ref([])   // Países filtrados por búsqueda
const searchQuery = ref('')         // Término de búsqueda actual
const isLoading = ref(false)        // Estado de carga
const hasError = ref(false)         // Estado de error
const selectedCountry = ref(null)   // País seleccionado para modal
const isModalOpen = ref(false)      // Estado del modal
```

### CountryCard.vue - Tarjeta de País
**Responsabilidades:**
- Mostrar información básica del país
- Formatear datos (población, etc.)
- Emitir eventos para abrir modal
- Efectos visuales y hover states

**Props:**
- `country` (Object): Datos del país a mostrar

**Eventos:**
- `open-modal`: Emitido al hacer clic en "Ver más información"

### CountryModal.vue - Modal de Información Detallada
**Responsabilidades:**
- Mostrar información completa en pestañas organizadas
- Formatear datos complejos (coordenadas, área, etc.)
- Gestionar navegación entre pestañas
- Renderizado con Teleport para mejor UX

**Props:**
- `country` (Object): Datos completos del país
- `isOpen` (Boolean): Estado de visibilidad del modal

**Eventos:**
- `close`: Emitido al cerrar el modal

## 🔌 Integración con API

### Estrategia de Múltiples Llamadas
Debido a las limitaciones de longitud de URL de la API, implementé una estrategia de múltiples llamadas para obtener todos los datos necesarios:

```javascript
// 1. Datos básicos
const response1 = await fetch('https://restcountries.com/v3.1/all?fields=name,flags,region,capital,population,languages,currencies,translations,tld,independent')

// 2. Datos geográficos
const response2 = await fetch('https://restcountries.com/v3.1/all?fields=name,subregion,area,latlng,landlocked,borders')

// 3. Datos políticos
const response3 = await fetch('https://restcountries.com/v3.1/all?fields=name,timezones,unMember,status,car')

// 4. Códigos de país
const response4 = await fetch('https://restcountries.com/v3.1/all?fields=name,cca2,cca3,ccn3,cioc')
```

### Procesamiento de Datos
- **Combinación de datos**: Merge de múltiples respuestas usando `name.common` como clave
- **Traducción a español**: Uso de `country.translations.spa.common` cuando está disponible
- **Aleatorización**: Implementación de algoritmo Fisher-Yates para mezclar países

## 🎯 Funcionalidades Clave Implementadas

### 1. Sistema de Búsqueda Inteligente
```javascript
// Implementación de debounce para optimizar rendimiento
const debouncedSearch = debounce(searchCountries, 300)

// Watcher reactivo para búsqueda en tiempo real
watch(searchQuery, (newQuery) => {
  debouncedSearch(newQuery)
})
```

### 2. Grid Responsivo Dinámico
```javascript
// Clases CSS dinámicas basadas en número de resultados
:class="{
  'grid-cols-1 max-w-sm': filteredCountries.length === 1,
  'grid-cols-1 md:grid-cols-2 max-w-4xl': filteredCountries.length === 2,
  'grid-cols-1 md:grid-cols-2 lg:grid-cols-3 max-w-6xl': filteredCountries.length === 3,
  'grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4': filteredCountries.length >= 4
}"
```

### 3. Modal con Pestañas Organizadas
- **General**: Información básica y estadísticas
- **Geografía**: Ubicación, territorio y fronteras
- **Cultura**: Idiomas y monedas
- **Política**: Estado político y membresías
- **Digital**: Dominios, códigos y zonas horarias

### 4. Formateo Inteligente de Datos
```javascript
// Formateo de población
formatPopulation(population) {
  if (population >= 1000000000) return (population / 1000000000).toFixed(1) + 'B'
  if (population >= 1000000) return (population / 1000000).toFixed(1) + 'M'
  if (population >= 1000) return (population / 1000).toFixed(0) + 'K'
  return population.toLocaleString()
}

// Formateo de coordenadas
formatCoordinates(latlng) {
  const [lat, lng] = latlng
  const latDir = lat >= 0 ? 'N' : 'S'
  const lngDir = lng >= 0 ? 'E' : 'O'
  return `${Math.abs(lat).toFixed(2)}°${latDir}, ${Math.abs(lng).toFixed(2)}°${lngDir}`
}
```

