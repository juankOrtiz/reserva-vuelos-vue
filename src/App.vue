<template>
  <div id="app-container">
    <h1>Formulario de Reserva de Vuelos (Vue.js)</h1>
    <progress :value="pasoActual" max="3"></progress>
    <p>Paso {{ pasoActual }} de 3</p>

    <DatosPasajero v-if="pasoActual === 1"
        :datosIniciales="datosReservaGlobal.pasajero"
        @siguiente-paso="avanzarPaso" />
    <DatosDestino
      v-if="pasoActual === 2"
      :datosIniciales="datosReservaGlobal.destino"
      @siguiente-paso="avanzarPaso"
      @paso-anterior="retrocederPaso"
      :provinciasMaestro="provincias"
      :ciudadesMaestro="todasLasCiudades"/>
    <ResumenReserva
      v-if="pasoActual === 3"
      @paso-anterior="retrocederPaso"
      @iniciar-nueva-reserva="resetearReserva"
      :datosReserva="datosReservaGlobal"
      :provinciasMaestro="provincias"
      :ciudadesMaestro="todasLasCiudades"/>
    </div>
</template>

<script>
import DatosPasajero from './components/DatosPasajero.vue';
import DatosDestino from './components/DatosDestino.vue';
import ResumenReserva from './components/ResumenReserva.vue';

export default {
  name: 'App',
  components: {
    DatosPasajero,
    DatosDestino,
    ResumenReserva
  },
  data() {
    return {
      pasoActual: 1, // Controla qué paso del formulario se muestra
      // Aquí podríamos almacenar los datos generales de la reserva que se recogen de los pasos
      datosReservaGlobal: {
        pasajero: {
          nombre: '',
          tipoDocumento: '',
          numeroDocumento: ''
        },
        destino: {
          provinciaOrigen: '',
          ciudadOrigen: '',
          provinciaDestino: '',
          ciudadDestino: ''
        },
        vueloSeleccionado: null
      },
      provincias: [], // Datos maestros de provincias
      todasLasCiudades: {} // Datos maestros de ciudades
    };
  },
  // 'created' hook: se ejecuta antes de que el componente sea montado en el DOM
  // Es un buen lugar para cargar datos iniciales que los hijos puedan necesitar como props
  async created() {
    await this.cargarDatosMaestros();
  },
  methods: {
    async cargarDatosMaestros() {
      try {
        // Cargar provincias
        const resProvincias = await fetch('/data/provincias.json');
        this.provincias = await resProvincias.json();
        console.log('Provincias cargadas:', this.provincias);

        // Cargar todas las ciudades
        const resCiudades = await fetch('/data/ciudades.json');
        this.todasLasCiudades = await resCiudades.json();
        console.log('Ciudades cargadas:', this.todasLasCiudades);

      } catch (error) {
        console.error('Error al cargar datos maestros:', error);
        // Podrías mostrar un mensaje de error al usuario aquí
      }
    },
    // Método llamado cuando un componente hijo emite 'siguiente-paso'
    avanzarPaso(datosDelPaso) {
      // Guardamos los datos del paso actual en el estado global
      if (this.pasoActual === 1) {
        Object.assign(this.datosReservaGlobal.pasajero, datosDelPaso);
      } else if (this.pasoActual === 2) {
        Object.assign(this.datosReservaGlobal.destino, datosDelPaso);
      }
      this.pasoActual++; // Avanzamos al siguiente paso
      console.log('Datos globales hasta ahora:', this.datosReservaGlobal);
    },
    // Método llamado cuando un componente hijo emite 'paso-anterior'
    retrocederPaso() {
      this.pasoActual--; // Retrocedemos un paso
    },
    resetearReserva() {
      console.log('Reserva completada. Reiniciando formulario...');
      this.pasoActual = 1; // Volver al primer paso
      this.datosReservaGlobal = { // Limpiar todos los datos de la reserva
        pasajero: { nombre: '', tipoDocumento: '', numeroDocumento: '' },
        destino: { provinciaOrigen: '', ciudadOrigen: '', provinciaDestino: '', ciudadDestino: '' },
        vueloSeleccionado: null
      };
      // Aquí, idealmente, también resetearías el estado de los componentes hijos si tuvieran estado interno
      // que no se limpiara con el cambio de v-if. Para nuestro caso, al cambiar de paso, se vuelven a renderizar
      // o se inicializan con datos vacíos si no se pasan props, lo cual es suficiente.
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
