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

    <button type="button" @click="$emit('paso-anterior')" class="prev-btn">Anterior</button>
    <button type="button" @click="confirmarReserva">Confirmar Reserva</button>
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
      confirmacionExitosa: false
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

      } catch (error) {
        this.mensajeConfirmacion = error.message || 'Error desconocido al procesar la reserva.';
        this.confirmacionExitosa = false;
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
</style>
