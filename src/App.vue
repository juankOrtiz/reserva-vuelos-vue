<template>
  <div id="app-container">
    <h1>Formulario de Reserva de Vuelos (Vue.js)</h1>
    <progress :value="pasoActual" max="3"></progress>
    <p>Paso {{ pasoActual }} de 3</p>

    <div class="top-buttons-container">
      <div class="reset-button-container" v-if="pasoActual >= 2">
        <button type="button" @click="confirmarReset" class="reset-btn">Borrar y Reiniciar</button>
      </div>

      <div class="show-data-container" v-if="pasoActual >= 2">
        <button type="button" @click.stop="toggleTooltip" class="show-data-btn">Datos actuales</button>
        <div v-if="showTooltip" class="data-tooltip">
          <h4>Datos actuales de la reserva:</h4>
          <p><strong>Pasajero:</strong></p>
          <ul>
            <li>Nombre: {{ datosReservaGlobal.pasajero.nombre || 'N/A' }}</li>
            <li>Documento: {{ datosReservaGlobal.pasajero.tipoDocumento || 'N/A' }} - {{ datosReservaGlobal.pasajero.numeroDocumento || 'N/A' }}</li>
          </ul>

          <p><strong>Origen:</strong></p>
          <ul>
            <li>Provincia: {{ getNombreProvinciaTooltip(datosReservaGlobal.destino.provinciaOrigen) }}</li>
            <li>Ciudad: {{ getNombreCiudadTooltip(datosReservaGlobal.destino.ciudadOrigen, datosReservaGlobal.destino.provinciaOrigen) }}</li>
          </ul>

          <p><strong>Destino:</strong></p>
          <ul>
            <li>Provincia: {{ getNombreProvinciaTooltip(datosReservaGlobal.destino.provinciaDestino) }}</li>
            <li>Ciudad: {{ getNombreCiudadTooltip(datosReservaGlobal.destino.ciudadDestino, datosReservaGlobal.destino.provinciaDestino) }}</li>
          </ul>

          <p v-if="datosReservaGlobal.vueloSeleccionado">
            <strong>Vuelo:</strong> {{ datosReservaGlobal.vueloSeleccionado.aerolinea }} - {{ datosReservaGlobal.vueloSeleccionado.numeroVuelo }}
          </p>
          <p v-else><strong>Vuelo:</strong> No seleccionado</p>

        </div>
      </div>
    </div>

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
      todasLasCiudades: {}, // Datos maestros de ciudades
      showTooltip: false // Controla la visibilidad del tooltip de datos actuales
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
  mounted() {
    // Añadir listener global para cerrar el tooltip al hacer clic fuera
    document.addEventListener('click', this.closeTooltipOnClickOutside);
  },
  beforeUnmount() {
    // Remover listener global antes de que el componente se desmonte
    document.removeEventListener('click', this.closeTooltipOnClickOutside);
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
    confirmarReset() {
      if (window.confirm('¿Estás seguro de que quieres borrar toda la reserva y empezar de nuevo? Esta acción es irreversible.')) {
        this.resetearReserva();
        console.log('Reserva borrada y reiniciada por el usuario.');
      } else {
        console.log('Reseteo cancelado por el usuario.');
      }
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
    },
    // Método para alternar la visibilidad del tooltip
    toggleTooltip() {
      this.showTooltip = !this.showTooltip;
    },
    // Método para cerrar el tooltip al hacer clic fuera de él
    closeTooltipOnClickOutside(event) {
      if (this.showTooltip && !this.$el.querySelector('.data-tooltip').contains(event.target) && !this.$el.querySelector('.show-data-btn').contains(event.target)) {
        this.showTooltip = false;
      }
    },
    // Métodos auxiliares para mostrar nombres en el tooltip
    getNombreProvinciaTooltip(id) {
        const provincia = this.provincias.find(p => p.id == id);
        return provincia ? provincia.nombre : 'N/A';
    },
    getNombreCiudadTooltip(id, provinciaId) {
        const ciudadesProvincia = this.todasLasCiudades[provinciaId] || [];
        const ciudad = ciudadesProvincia.find(c => c.id == id);
        return ciudad ? ciudad.nombre : 'N/A';
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

/* Contenedor para alinear los botones superiores */
.top-buttons-container {
    display: flex;
    justify-content: space-between; /* Alinea los elementos a los extremos */
    align-items: flex-start; /* Alinea al inicio verticalmente */
    margin-bottom: 20px;
    position: relative; /* Para el posicionamiento absoluto del tooltip */
}

/* Estilos para el botón de reseteo */
.reset-button-container {
  text-align: left;
}

.reset-btn {
  background-color: #dc3545; /* Rojo para indicar una acción destructiva */
  color: white;
  padding: 8px 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.9em;
  margin-left: 10px; /* Para separarlo de otros elementos si los hubiera */
}

.reset-btn:hover {
  background-color: #c82333;
}

/* Estilos para el botón "Datos actuales" y el tooltip */
.show-data-container {
    position: relative; /* Esencial para que el tooltip se posicione en relación a este contenedor */
    text-align: right; /* Alinea a la derecha dentro de su contenedor flex */
}

.show-data-btn {
    background-color: #17a2b8; /* Un color distinto, como un azul claro */
    color: white;
    padding: 8px 12px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 0.9em;
}

.show-data-btn:hover {
    background-color: #138496;
}

.data-tooltip {
    position: absolute; /* Posicionamiento absoluto respecto a .show-data-container */
    top: calc(100% + 10px); /* 10px debajo del botón */
    right: 0; /* Alineado a la derecha del botón */
    width: 300px; /* Ancho fijo para el tooltip */
    background-color: white;
    border: 1px solid #ddd;
    border-radius: 8px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
    padding: 15px;
    z-index: 1000; /* Asegura que esté por encima de otros elementos */
    text-align: left; /* Alinea el texto del tooltip a la izquierda */
}

.data-tooltip h4 {
    margin-top: 0;
    margin-bottom: 10px;
    color: #333;
    text-align: left; /* Asegura que el título también esté a la izquierda */
}

.data-tooltip p, .data-tooltip ul {
    font-size: 0.9em;
    color: #555;
    margin-bottom: 5px;
    text-align: left; /* Asegura que el texto esté a la izquierda */
}

.data-tooltip ul {
    list-style: none; /* Quita viñetas por defecto */
    padding-left: 10px; /* Indentación para la lista */
    margin-top: 0;
    margin-bottom: 10px;
}

.data-tooltip ul li {
    margin-bottom: 3px;
}
</style>
