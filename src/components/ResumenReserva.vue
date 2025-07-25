<template>
  <div>
    <h2>Paso 3: Resumen de la Reserva</h2>
    <h3>Datos del Pasajero:</h3>
    <p><strong>Nombre:</strong> {{ datosReserva.pasajero.nombre }}</p>
    <p><strong>Documento:</strong> {{ datosReserva.pasajero.tipoDocumento }} - {{ datosReserva.pasajero.numeroDocumento }}</p>

    <h3>Datos del Destino:</h3>
    <p><strong>Origen:</strong> {{ getNombreCiudad(datosReserva.destino.ciudadOrigen, datosReserva.destino.provinciaOrigen) }}, {{ getNombreProvincia(datosReserva.destino.provinciaOrigen) }}</p>
    <p><strong>Destino:</strong> {{ getNombreCiudad(datosReserva.destino.ciudadDestino, datosReserva.destino.provinciaDestino) }}, {{ getNombreProvincia(datosReserva.destino.provinciaDestino) }}</p>

    <h3>Vuelo Seleccionado:</h3>
    <div v-if="vueloSeleccionado">
        <p><strong>Aerolínea:</strong> {{ vueloSeleccionado.aerolinea }}</p>
        <p><strong>Número de Vuelo:</strong> {{ vueloSeleccionado.numeroVuelo }}</p>
        <p><strong>Fecha:</strong> {{ vueloSeleccionado.fecha }}</p>
        <p><strong>Precio:</strong> ${{ vueloSeleccionado.precio }}</p>
    </div>
    <p v-else>No se ha seleccionado ningún vuelo.</p>


    <ListaVuelos @vuelo-seleccionado="handleVueloSeleccionado" />

    <p id="confirmacionMessage" :class="{ 'success': confirmacionExitosa, 'error': !confirmacionExitosa, 'hidden': !mensajeConfirmacion }">
        {{ mensajeConfirmacion }}
    </p>

    <div class="button-group">
      <template v-if="!reservaCompletadaExitosamente">
        <button type="button" @click="$emit('paso-anterior')" class="prev-btn" :disabled="procesandoReserva">Anterior</button>
        <button
          type="button"
          @click="confirmarReserva"
          :disabled="!vueloSeleccionado || procesandoReserva"
          class="confirm-btn">
          {{ procesandoReserva ? 'Confirmando...' : 'Confirmar Reserva' }}
        </button>
      </template>
      <template v-else>
        <button type="button" @click="$emit('iniciar-nueva-reserva')" class="new-reservation-btn">Iniciar Nueva Reserva</button>
      </template>
    </div>
  </div>
</template>

<script>
import ListaVuelos from './ListaVuelos.vue'; // Importa el nuevo componente

export default {
  name: 'ResumenReserva',
  components: {
    ListaVuelos
  },
  props: {
    // Estas props vendrán de App.vue
    datosReserva: {
      type: Object,
      required: true
    },
    provinciasMaestro: { // Necesitamos los datos maestros para mostrar nombres
        type: Array,
        required: true
    },
    ciudadesMaestro: { // Necesitamos los datos maestros para mostrar nombres
        type: Object,
        required: true
    }
  },
  data() {
    return {
      vueloSeleccionado: null, // Para almacenar el vuelo que el usuario selecciona en ListaVuelos
      mensajeConfirmacion: '',
      confirmacionExitosa: false,
      reservaCompletadaExitosamente: false,
      procesandoReserva: false
    };
  },
  methods: {
    // Métodos de ayuda para obtener nombres legibles de IDs (similar a Vanilla JS)
    getNombreProvincia(id) {
        const provincia = this.provinciasMaestro.find(p => p.id == id); // Usar == para comparar string con number
        return provincia ? provincia.nombre : 'Desconocido';
    },
    getNombreCiudad(id, provinciaId) {
        const ciudadesProvincia = this.ciudadesMaestro[provinciaId] || [];
        const ciudad = ciudadesProvincia.find(c => c.id == id); // Usar ==
        return ciudad ? ciudad.nombre : 'Desconocido';
    },
    handleVueloSeleccionado(vuelo) {
        // Este método se llama cuando ListaVuelos emite 'vuelo-seleccionado'
        this.vueloSeleccionado = vuelo;
        // Si ya había una confirmación exitosa, re-habilitar el botón si selecciona otro vuelo
        // para un caso hipotético donde un usuario pueda querer cambiar su selección post-éxito.
        // Opcional: podrías decidir resetear todo aquí.
        if (this.reservaCompletadaExitosamente) {
            this.reservaCompletadaExitosamente = false; // Permite un nuevo intento de confirmación
            this.mensajeConfirmacion = ''; // Limpia el mensaje anterior
        }
    },
    async confirmarReserva() {
      this.mensajeConfirmacion = 'Enviando reserva...';
      this.confirmacionExitosa = false;
      this.ocultarConfirmacionMensaje(); // Limpiar la clase hidden

      if (!this.vueloSeleccionado) {
          this.mensajeConfirmacion = 'Debe seleccionar un vuelo antes de confirmar la reserva.';
          this.confirmacionExitosa = false;
          return;
      }

      try {
        // Simulación de una llamada AJAX para enviar datos
        console.log('Enviando datos de reserva:', {
            pasajero: this.datosReserva.pasajero,
            destino: this.datosReserva.destino,
            vuelo: this.vueloSeleccionado
        });

        const response = await new Promise((resolve, reject) => {
          setTimeout(() => {
            if (Math.random() > 0.3) { // 70% de éxito
              resolve({ success: true, message: '¡Reserva confirmada con éxito!' });
            } else {
              reject({ success: false, message: 'Hubo un error al confirmar la reserva. Intente nuevamente.' });
            }
          }, 1500);
        });

        this.mensajeConfirmacion = response.message;
        this.confirmacionExitosa = response.success;

        if (response.success) {
            this.reservaCompletadaExitosamente = true;
        }

      } catch (error) {
        this.mensajeConfirmacion = error.message || 'Error desconocido al procesar la reserva.';
        this.confirmacionExitosa = false;
        this.reservaCompletadaExitosamente = false;
      } finally {
        this.procesandoReserva = false; // Siempre restablecer el estado de procesamiento
      }
    },
    ocultarConfirmacionMensaje() {
        // Asegurarse de que el mensaje esté visible antes de agregar clases
        document.getElementById('confirmacionMessage').classList.remove('hidden');
    }
  }
};
</script>

<style scoped>
/* Estilos copiados de Vanilla JS */
h2, h3 {
  text-align: center;
  margin-top: 20px;
  margin-bottom: 15px;
  color: #333;
}
p {
  text-align: left;
  margin-bottom: 8px;
}

button {
  background-color: #007bff;
  color: white;
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1em;
  margin-right: 10px;
  margin-top: 15px;
}

button:hover {
  background-color: #0056b3;
}

.prev-btn {
    background-color: #6c757d;
}

.prev-btn:hover {
    background-color: #5a6268;
}

#confirmacionMessage {
    text-align: center;
    margin-top: 20px;
    padding: 10px;
    border-radius: 4px;
    border: 1px solid;
}

.success {
    color: green;
    border-color: green;
    background-color: #e6ffe6;
}
.error {
    color: red;
    border-color: red;
    background-color: #ffe6e6;
}
.hidden {
    display: none !important; /* Asegurar que se oculte */
}
.button-group {
    margin-top: 25px;
    text-align: center; /* Centra los botones dentro del grupo */
}

.new-reservation-btn {
    background-color: #28a745; /* Color verde para "Nueva Reserva" */
}
.new-reservation-btn:hover {
    background-color: #218838;
}

/* Agregado para deshabilitar el botón de Confirmar si no hay vuelo seleccionado */
button:disabled {
    background-color: #cccccc;
    cursor: not-allowed;
}

.confirm-btn {
  background-color: #007bff;
}
.confirm-btn:hover {
  background-color: #0056b3;
}
.confirm-btn:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}
</style>
