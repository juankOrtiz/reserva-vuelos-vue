<template>
  <div id="app-container">
    <h1>Formulario de Reserva de Vuelos (Vue.js)</h1>
    <DatosPasajero />
  </div>
</template>

<script>
// Importa el componente que acabamos de crear
import DatosPasajero from './components/DatosPasajero.vue';

export default {
  name: 'App', // El nombre de este componente raíz
  components: {
    // Registra el componente DatosPasajero para que pueda ser usado en la plantilla
    DatosPasajero
  }
}
</script>

<style>
/* Estilos globales para la aplicación */
body {
  font-family: Arial, sans-serif;
  background-color: #f4f4f4;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  min-height: 100vh;
  margin: 20px;
  box-sizing: border-box;
}

#app-container {
  background-color: #fff;
  padding: 30px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  width: 100%;
  max-width: 700px;
  box-sizing: border-box;
}

h1 {
  color: #333;
  text-align: center;
  margin-bottom: 20px;
}
</style>
