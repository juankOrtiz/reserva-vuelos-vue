<template>
    <div>
        <h2>Paso 1: Datos del Pasajero</h2>

        <div class="form-group">
            <label for="nombre">Nombre:</label>
            <input type="text" id="nombre" v-model="pasajero.nombre" @blur="marcarTocado('nombre'); validarCampo('nombre');" />
            <CampoError :mensaje="errores.nombre && touched.nombre ? errores.nombre : ''" />
        </div>

        <div class="form-group">
            <label for="tipoDocumento">Tipo de Documento:</label>
            <select id="tipoDocumento" v-model="pasajero.tipoDocumento" @change="marcarTocado('tipoDocumento'); handleTipoDocumentoChange();">
                <option value="">Seleccione</option>
                <option value="DNI">DNI</option>
                <option value="Pasaporte">Pasaporte</option>
            </select>
            <CampoError :mensaje="errores.tipoDocumento && touched.tipoDocumento ? errores.tipoDocumento : ''" />
        </div>

        <div class="form-group" v-if="pasajero.tipoDocumento">
            <label for="numeroDocumento">Número de Documento:</label>
            <input
                type="text"
                id="numeroDocumento"
                v-model="pasajero.numeroDocumento"
                :disabled="!pasajero.tipoDocumento"
                @blur="marcarTocado('numeroDocumento'); validarCampo('numeroDocumento');"
            />
            <CampoError :mensaje="errores.numeroDocumento && touched.numeroDocumento ? errores.numeroDocumento : ''" />
        </div>

        <button type="button" @click="siguientePaso">Siguiente</button>
    </div>
</template>

<script>
import CampoError from './CampoError.vue'; // Importa el componente de error

// Aquí irá la lógica JavaScript de este componente
export default {
  name: 'DatosPasajero', // Un nombre descriptivo para el componente
    components: {
        CampoError // Registra el componente CampoError para usarlo en este template
    },
  props: {
    // Define la prop que recibirá los datos iniciales del pasajero
    datosIniciales: {
      type: Object,
      default: () => ({ nombre: '', tipoDocumento: '', numeroDocumento: '' })
    }
  },
  data() {
    return {
      // Define los datos de tu componente aquí. Vue los hará reactivos.
      pasajero: {
        ...this.datosIniciales
      },
      errores: {
        nombre: '',
        tipoDocumento: '',
        numeroDocumento: '',
      },
      // NUEVO: Objeto para rastrear si cada campo ha sido "tocado"
      touched: {
        nombre: false,
        tipoDocumento: false,
        numeroDocumento: false
      }
    };
  },
  watch: {
    // Opcional pero buena práctica: observa si la prop datosIniciales cambia
    // y actualiza los datos internos del componente.
    // Esto es útil si la prop pudiera cambiar mientras el componente ya está montado.
    // En nuestro caso, el componente se desmonta y monta, por lo que `data` se inicializa
    // de nuevo, pero es un buen patrón a conocer.
    datosIniciales: {
      handler(newVal) {
        Object.assign(this.pasajero, newVal);
        // Cuando los datos iniciales cambian (ej. al retroceder),
        // reiniciamos el estado "tocado" para evitar mostrar errores inmediatamente
        this.resetearTocado();
        // Luego, re-validamos para que los datos cargados tengan su estado de error calculado,
        // pero NO se mostrarán hasta que el campo sea tocado o el formulario se intente enviar.
        this.validarCampo('nombre');
        this.validarCampo('tipoDocumento');
        this.handleTipoDocumentoChange(false); // Pasar 'false' para no marcar como tocado al re-hidratar
      },
      deep: true, // Observa cambios anidados dentro del objeto
      immediate: true // Ejecuta el handler inmediatamente al montar
    }
  },
  methods: {
    marcarTocado(campo) {
      this.touched[campo] = true;
    },
    resetearTocado() {
      this.touched.nombre = false;
      this.touched.tipoDocumento = false;
      this.touched.numeroDocumento = false;
    },
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
    handleTipoDocumentoChange(markAsTouched = true) { // Añadir parámetro opcional
        if (markAsTouched) { // Solo marcar como tocado si es un cambio real del usuario
            this.marcarTocado('tipoDocumento');
        }
        this.validarCampo('tipoDocumento');

        // Si el tipo de documento deja de ser válido, limpiamos el numeroDocumento y su error.
        // Esto es similar a lo que hacíamos en Vanilla JS para limpiar el campo.
        if (!this.pasajero.tipoDocumento) {
            this.pasajero.numeroDocumento = '';
            this.errores.numeroDocumento = '';
            this.touched.numeroDocumento = false;
        } else {
            // Si selecciona un tipo, y el campo de numero ya tiene algo, re-validamos
            if (this.pasajero.numeroDocumento) {
                 this.validarCampo('numeroDocumento');
            }
        }
    },
    siguientePaso() {
      // Al intentar avanzar, marcamos TODOS los campos como tocados para forzar la validación
      this.marcarTocado('nombre');
      this.marcarTocado('tipoDocumento');
      // Solo marcar numeroDocumento si está visible (es decir, tipoDocumento seleccionado)
      if (this.pasajero.tipoDocumento) {
          this.marcarTocado('numeroDocumento');
      }

      // Luego, ejecutar todas las validaciones
      this.validarCampo('nombre');
      this.validarCampo('tipoDocumento');
      if (this.pasajero.tipoDocumento) {
          this.validarCampo('numeroDocumento');
      }

      const tieneErrores = Object.values(this.errores).some(error => error !== '');

      if (!tieneErrores) {
        console.log('Datos del pasajero válidos:', this.pasajero);
        this.$emit('siguiente-paso', this.pasajero);
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
