<template>
  <div class="main-container">
    <div class="info-container">
      <h2 class="info-title">Evaluación del Tutor 2023B</h2>

      <div v-if="isLoading" class="loading-message">
        <p>Cargando...</p>
      </div>
      <div v-if="!isLoading">
        <div class="eval-info">
          <h3>Detalles de la Evaluación</h3>
          <ul>
            <li v-for="(value, key) in statusEvalInfo" :key="key" class="eval-item">
              <span>{{ key }}:</span> {{ key === 'inicio' || key === 'fin' ? formatDate(value) : value }}
            </li>
          </ul>
        </div>
      </div>
    </div>

    <div class="ing-info-container">
      <div class="ing-info">
        <h3>Información por Ingeniería</h3>
        <ul>
          <li v-for="(value, key) in statusEvalIngInfo" :key="key" class="ing-item">
            <span class="ing-carrera">{{ getFullCarreraName(key) }}:</span>
            <ul class="ing-details">
              <li>
                <span>Listas:</span> {{ value.listas }}
              </li>
              <li>
                <span>Completado:</span>
                <div class="progress-bar-container">
                  <div class="progress-bar" :style="{ width: calculatePercentage(value) + '%' }"></div>
                </div>
                <p>{{ calculateCompletedPercentage(value) }}% Presentado</p>
                <p>{{ calculateRemainingPercentage(value) }}% Pendiente</p>
              </li>
              <li>
                <span>Faltantes:</span> {{ value.faltantes }}
              </li>
            </ul>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const statusEvalInfo = ref({});
const statusEvalIngInfo = ref({});
const isLoading = ref(true);
const startDate = ref(null);
const endDate = ref(null);

async function fetchData(url, target) {
  try {
    const response = await fetch(url);
    const data = await response.json();
    console.log(`${target} data:`, data);

    if (target === statusEvalInfo) {
      startDate.value = data.inicio;
      endDate.value = data.fin;
    }

    target.value = data;
  } catch (error) {
    console.error(`Error al obtener datos (${target}):`, error);
  }
}

function calculatePercentage(category) {
  const totalListas = category.listas || 0;
  const completed = totalListas - (category.faltantes || 0);
  return totalListas > 0 ? Math.round((100 * completed) / totalListas) : 0;
}

function calculateCompletedPercentage(category) {
  const totalListas = category.listas || 0;
  const completed = totalListas - (category.faltantes || 0);
  return totalListas > 0 ? Math.round((100 * completed) / totalListas) : 0;
}

function calculateRemainingPercentage(category) {
  const totalListas = category.listas || 0;
  const remaining = category.faltantes || 0;
  return totalListas > 0 ? Math.round((100 * remaining) / totalListas) : 0;
}

onMounted(async () => {
  await Promise.all([
    fetchData('https://sitmotul.com.mx/api/statusEval', statusEvalInfo),
    fetchData('https://sitmotul.com.mx/api/statusEvalIng', statusEvalIngInfo),
  ]);
  isLoading.value = false;
});

function formatDate(dateString) {
  // Función formatDate sigue igual, según tus necesidades
}

const carreraMapping = {
  ing: 'Ingeniería',
  ISC: 'Sistemas computacionales',
  IEM: 'Electromecánica',
  IER: 'Energías renovables',
  II: 'Industrial',
  IE: 'Electrónica',
  // Agrega más abreviaturas según sea necesario
};

function getFullCarreraName(abbreviation) {
  return carreraMapping[abbreviation] || abbreviation;
}
</script>

<style scoped>
.main-container {
  display: flex;
  flex-direction: column;
  align-items: center; /* Centrar horizontalmente */
  justify-content: center; /* Centrar verticalmente */
  padding: 20px; /* Usa unidades absolutas (px) para el relleno */
}

.info-container,
.ing-info-container {
  background-color: #008080; /* Teal */
  color: #ffffff; /* Texto blanco sobre teal */
  padding: 15px;
  border-radius: 20px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  margin: 10px;
  width: 80%; /* Ajusta el ancho según tus necesidades */
  max-width: 600px; /* Establece un ancho máximo */
}

.info-title {
  font-size: 1.5rem;
  font-weight: bold;
  margin-bottom: 20px;
  font-family: 'Arial Black', Arial;
}

.loading-message {
  text-align: center;
  font-size: 16px;
  color: #ffffff; /* Texto blanco sobre teal */
}

.eval-info,
.ing-info {
  margin-top: 20px;
}

.eval-info h3,
.ing-info h3 {
  font-size: 1.5rem;
  font-weight: bold;
  margin-bottom: 10px;
  font-family: 'Arial', Arial, sans-serif;
}

.eval-item,
.ing-item {
  margin-bottom: 20px;
  font-family: 'Arial', Arial;
}

.ing-details {
  background-color: #794A03; /* Orange */
  padding: 10px;
  border-radius: 8px;
  list-style: none;
  font-family: 'Arial', Arial;
}

.ing-details span {
  font-weight: bold;
  margin-right: 10px;
}

.ing-carrera {
  font-weight: bold;
  margin-right: 10px;
}

.progress-bar-container {
  width: 100%;
  height: 15px;
  background-color: #cfffca; /* Light teal */
  border-radius: 5px;
  overflow: hidden;
  margin-top: 5px;
}

.progress-bar {
  height: 100%;
  background-color: #ff8c00; /* Dark orange */
  border-radius: 10px;
  transition: width 0.5s ease;
}
</style>
