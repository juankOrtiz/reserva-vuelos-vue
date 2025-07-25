<template>
    <div>
        <h2>Paso 1: Datos del Pasajero</h2>

        <div class="form-group">
            <label for="nombre">Nombre:</label>
            <input type="text" id="nombre" v-model="pasajero.nombre" @blur="validarCampo('nombre')" />
            <p v-if="errores.nombre" class="error-message">{{ errores.nombre }}</p>
        </div>

        <div class="form-group">
            <label for="tipoDocumento">Tipo de Documento:</label>
            <select id="tipoDocumento" v-model="pasajero.tipoDocumento" @change="handleTipoDocumentoChange">
                <option value="">Seleccione</option>
                <option value="DNI">DNI</option>
                <option value="Pasaporte">Pasaporte</option>
            </select>
            <p v-if="errores.tipoDocumento" class="error-message">{{ errores.tipoDocumento }}</p>
        </div>

        <div class="form-group" v-if="pasajero.tipoDocumento">
            <label for="numeroDocumento">Número de Documento:</label>
            <input
                type="text"
                id="numeroDocumento"
                v-model="pasajero.numeroDocumento"
                :disabled="!pasajero.tipoDocumento"
                @blur="validarCampo('numeroDocumento')"
            />
            <p v-if="errores.numeroDocumento" class="error-message">{{ errores.numeroDocumento }}</p>
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
        nombre: '',
        tipoDocumento: '',
        numeroDocumento: '',
      },
      errores: {
        nombre: '',
        tipoDocumento: '',
        numeroDocumento: '',
      }
    };
  },
  methods: {
    validarCampo(campo) {
        // Validar Nombre
        if (campo === 'nombre') {
            if (!this.pasajero.nombre.trim()) {
                this.errores.nombre = 'El nombre es obligatorio.';
            } else if (this.pasajero.nombre.length < 3) {
                this.errores.nombre = 'El nombre debe tener al menos 3 caracteres.';
            } else {
                this.errores.nombre = ''; // Limpiar el error si es válido
            }
        }

        // Validar Tipo de Documento
        if (campo === 'tipoDocumento') {
            if (!this.pasajero.tipoDocumento) {
                this.errores.tipoDocumento = 'Debe seleccionar un tipo de documento.';
            } else {
                this.errores.tipoDocumento = '';
            }
        }

        // Validar Número de Documento (solo si se ha seleccionado un tipo)
        if (campo === 'numeroDocumento' && this.pasajero.tipoDocumento) {
            if (!this.pasajero.numeroDocumento.trim()) {
                this.errores.numeroDocumento = 'El número de documento es obligatorio.';
            } else if (this.pasajero.tipoDocumento === 'DNI' && !/^\d{7,8}$/.test(this.pasajero.numeroDocumento)) {
                this.errores.numeroDocumento = 'El DNI debe contener 7 u 8 dígitos numéricos.';
            } else if (this.pasajero.tipoDocumento === 'Pasaporte' && !/^[A-Z0-9]{6,15}$/.test(this.pasajero.numeroDocumento)) {
                this.errores.numeroDocumento = 'El pasaporte debe tener entre 6 y 15 caracteres alfanuméricos.';
            } else {
                this.errores.numeroDocumento = '';
            }
        }
    },
    handleTipoDocumentoChange() {
        // Cuando el tipo de documento cambia, validamos el tipo
        this.validarCampo('tipoDocumento');

        // Si el tipo de documento deja de ser válido, limpiamos el numeroDocumento y su error.
        // Esto es similar a lo que hacíamos en Vanilla JS para limpiar el campo.
        if (!this.pasajero.tipoDocumento) {
            this.pasajero.numeroDocumento = '';
            this.errores.numeroDocumento = '';
        } else {
            // Si selecciona un tipo, y el campo de numero ya tiene algo, re-validamos
            if (this.pasajero.numeroDocumento) {
                 this.validarCampo('numeroDocumento');
            }
        }
    },
    siguientePaso() {
      // Validar todos los campos antes de avanzar
      this.validarCampo('nombre');
      this.validarCampo('tipoDocumento');
      // Solo validar numeroDocumento si el tipo está seleccionado
      if (this.pasajero.tipoDocumento) {
          this.validarCampo('numeroDocumento');
      }

      // Comprobar si hay errores en cualquier campo
      const tieneErrores = Object.values(this.errores).some(error => error !== '');

      if (!tieneErrores) {
        console.log('Datos del pasajero válidos:', this.pasajero);
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

input[type="text"]:disabled,
select:disabled {
    background-color: #eee;
    cursor: not-allowed;
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
