<template>
    <div>
        <h2>Paso 1: Datos del Pasajero</h2>

        <div class="form-group">
            <label for="nombre">Nombre:</label>
            <input type="text" id="nombre" v-model="pasajero.nombre" @blur="validarCampo('nombre')" />
            <p v-if="errores.nombre" class="error-message">{{ errores.nombre }}</p>
        </div>

        <button type="button" @click="siguientePaso">Siguiente</button>
    </div>
</template>

<script>
// Aquí irá la lógica JavaScript de este componente
export default {
  name: 'DatosPasajero', // Un nombre descriptivo para el componente
  data() {
    return {
      // Define los datos de tu componente aquí. Vue los hará reactivos.
      pasajero: {
        nombre: '' // Inicializamos el nombre vacío
      },
      errores: {
        nombre: '' // Para almacenar el mensaje de error del nombre
      }
    };
  },
  methods: {
    validarCampo(campo) {
      if (campo === 'nombre') {
        if (!this.pasajero.nombre.trim()) {
          this.errores.nombre = 'El nombre es obligatorio.';
        } else if (this.pasajero.nombre.length < 3) {
          this.errores.nombre = 'El nombre debe tener al menos 3 caracteres.';
        } else {
          this.errores.nombre = ''; // Limpiar el error si es válido
        }
      }
      // Aquí se agregarán más validaciones para otros campos
    },
    siguientePaso() {
      // Llamamos a la validación antes de avanzar
      this.validarCampo('nombre');

      // Comprobamos si hay errores
      if (!this.errores.nombre) {
        console.log('Datos del pasajero válidos:', this.pasajero.nombre);
        // Aquí eventualmente emitiremos un evento para que el componente padre avance
      }
    }
  }
};
</script>

<style scoped>
/* Aquí irán los estilos CSS específicos para este componente */
/* El atributo 'scoped' es importante, aplica los estilos solo a los elementos dentro de este componente */
.form-group {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
  color: #555;
}

input[type="text"] {
  width: calc(100% - 20px);
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
  font-size: 1em;
}

.error-message {
  color: red;
  font-size: 0.85em;
  margin-top: 5px;
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
</style>
