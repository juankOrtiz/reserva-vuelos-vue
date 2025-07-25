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

const STORAGE_KEY = 'reservaFormDataVue'; // Clave para los datos de la reserva
const STEP_KEY = 'reservaCurrentStepVue'; // Clave para el paso actual

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
    // 1. Cargar datos maestros (provincias y ciudades) primero
    await this.cargarDatosMaestros();

    // 2. Intentar cargar los datos de la reserva y el paso desde LocalStorage
    this.cargarEstadoDesdeLocalStorage();
  },
  watch: {
    datosReservaGlobal: {
      handler(newVal) {
        localStorage.setItem(STORAGE_KEY, JSON.stringify(newVal));
        console.log('Datos de reserva guardados en LocalStorage.');
      },
      deep: true // Observar cambios anidados dentro del objeto
    },
    pasoActual: {
      handler(newVal) {
        localStorage.setItem(STEP_KEY, newVal);
        console.log('Paso actual guardado en LocalStorage.');
      }
    }
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
    cargarEstadoDesdeLocalStorage() {
        const savedData = localStorage.getItem(STORAGE_KEY);
        const savedStep = localStorage.getItem(STEP_KEY);

        if (savedData) {
            try {
                const parsedData = JSON.parse(savedData);
                // Fusionar los datos guardados con los datos iniciales por defecto.
                // Esto es importante para mantener la reactividad si los objetos no se inicializan por completo.
                Object.assign(this.datosReservaGlobal.pasajero, parsedData.pasajero || {});
                Object.assign(this.datosReservaGlobal.destino, parsedData.destino || {});
                this.datosReservaGlobal.vueloSeleccionado = parsedData.vueloSeleccionado || null;

                console.log('Datos de reserva recuperados de LocalStorage:', this.datosReservaGlobal);
            } catch (e) {
                console.error('Error al parsear datos de LocalStorage:', e);
                localStorage.removeItem(STORAGE_KEY); // Limpiar datos corruptos
            }
        }

        if (savedStep) {
            const parsedStep = parseInt(savedStep, 10);
            // Asegurarse de que el paso sea válido (entre 1 y 3)
            if (parsedStep >= 1 && parsedStep <= 3) {
                this.pasoActual = parsedStep;
                console.log('Paso actual recuperado de LocalStorage:', this.pasoActual);
            } else {
                this.pasoActual = 1; // Si el paso guardado es inválido, vuelve al paso 1
            }
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
      // ¡NUEVO! Limpiar LocalStorage cuando se inicia una nueva reserva
      localStorage.removeItem(STORAGE_KEY);
      localStorage.removeItem(STEP_KEY);
      console.log('LocalStorage limpiado.');
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
