<template>
  <div id="app-container">
    <h1>Formulario de Reserva de Vuelos (Vue.js)</h1>
    <progress :value="pasoActual" max="3"></progress>
    <p>Paso {{ pasoActual }} de 3</p>

    <DatosPasajero v-if="pasoActual === 1" @siguiente-paso="avanzarPaso" />
    <DatosDestino
      v-if="pasoActual === 2"
      @siguiente-paso="avanzarPaso"
      @paso-anterior="retrocederPaso"
    />
    </div>
</template>

<script>
import DatosPasajero from './components/DatosPasajero.vue';
import DatosDestino from './components/DatosDestino.vue';
// import ResumenReserva from './components/ResumenReserva.vue'; // Todavía no lo tenemos

export default {
  name: 'App',
  components: {
    DatosPasajero,
    DatosDestino
    // ResumenReserva
  },
  data() {
    return {
      pasoActual: 1, // Controla qué paso del formulario se muestra
      // Aquí podríamos almacenar los datos generales de la reserva que se recogen de los pasos
      datosReservaGlobal: {
        pasajero: {},
        destino: {}
      }
    };
  },
  methods: {
    // Método llamado cuando un componente hijo emite 'siguiente-paso'
    avanzarPaso(datosDelPaso) {
      // Guardamos los datos del paso actual en el estado global
      if (this.pasoActual === 1) {
        this.datosReservaGlobal.pasajero = datosDelPaso;
      } else if (this.pasoActual === 2) {
        this.datosReservaGlobal.destino = datosDelPaso;
      }
      this.pasoActual++; // Avanzamos al siguiente paso
      console.log('Datos globales hasta ahora:', this.datosReservaGlobal);
    },
    // Método llamado cuando un componente hijo emite 'paso-anterior'
    retrocederPaso() {
      this.pasoActual--; // Retrocedemos un paso
    }
  }
}
</script>

<style>
/* Los estilos de App.vue del concepto anterior ya están aquí */
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

progress {
    width: 100%;
    margin-bottom: 10px;
}

p {
    text-align: center;
    margin-bottom: 20px;
}
</style>
