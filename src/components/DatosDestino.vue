<template>
  <form @submit.prevent="siguientePaso">
    <h2>Paso 2: Destino del Vuelo</h2>
    <div class="form-group">
      <label for="provinciaOrigen">Provincia de Origen:</label>
      <select id="provinciaOrigen" v-model="reserva.provinciaOrigen" @change="marcarTocado('provinciaOrigen'); cargarCiudades('origen', true);">
        <option value="">Seleccione una provincia</option>
        <option v-for="provincia in provincias" :key="provincia.id" :value="provincia.id">
          {{ provincia.nombre }}
        </option>
      </select>
      <p v-if="errores.provinciaOrigen && touched.provinciaOrigen" class="error-message">{{ errores.provinciaOrigen }}</p>
    </div>

    <div class="form-group" v-if="reserva.provinciaOrigen">
      <label for="ciudadOrigen">Ciudad de Origen:</label>
      <select id="ciudadOrigen" v-model="reserva.ciudadOrigen" :disabled="ciudadesOrigen.length === 0" @change="marcarTocado('ciudadOrigen'); validarCampo('ciudadOrigen');">
        <option value="">Seleccione una ciudad</option>
        <option v-for="ciudad in ciudadesOrigen" :key="ciudad.id" :value="ciudad.id">
          {{ ciudad.nombre }}
        </option>
      </select>
      <p v-if="errores.ciudadOrigen && touched.ciudadOrigen" class="error-message">{{ errores.ciudadOrigen }}</p>
    </div>

    <div class="form-group">
      <label for="provinciaDestino">Provincia de Destino:</label>
      <select id="provinciaDestino" v-model="reserva.provinciaDestino" @change="marcarTocado('provinciaDestino'); cargarCiudades('destino', true);">
        <option value="">Seleccione una provincia</option>
        <option v-for="provincia in provincias" :key="provincia.id" :value="provincia.id">
          {{ provincia.nombre }}
        </option>
      </select>
      <p v-if="errores.provinciaDestino && touched.provinciaDestino" class="error-message">{{ errores.provinciaDestino }}</p>
    </div>

    <div class="form-group" v-if="reserva.provinciaDestino">
      <label for="ciudadDestino">Ciudad de Destino:</label>
      <select id="ciudadDestino" v-model="reserva.ciudadDestino" :disabled="ciudadesDestino.length === 0" @change="marcarTocado('ciudadDestino'); validarCampo('ciudadDestino');">
        <option value="">Seleccione una ciudad</option>
        <option v-for="ciudad in ciudadesDestino" :key="ciudad.id" :value="ciudad.id">
          {{ ciudad.nombre }}
        </option>
      </select>
      <p v-if="errores.ciudadDestino && touched.ciudadDestino" class="error-message">{{ errores.ciudadDestino }}</p>
    </div>

    <button type="button" @click="$emit('paso-anterior')" class="prev-btn">Anterior</button>
    <button type="submit">Siguiente</button>
  </form>
</template>

<script>
export default {
  name: 'DatosDestino',
  props: {
    datosIniciales: {
      type: Object,
      default: () => ({ provinciaOrigen: '', ciudadOrigen: '', provinciaDestino: '', ciudadDestino: '' })
    },
    provinciasMaestro: { type: Array, required: true },
    ciudadesMaestro: { type: Object, required: true }
  },
  data() {
    return {
      reserva: {
        ...this.datosIniciales
      },
      provincias: [], // Aquí almacenaremos las provincias obtenidas de la "API"
      ciudadesOrigen: [], // Ciudades filtradas para la provincia de origen
      ciudadesDestino: [], // Ciudades filtradas para la provincia de destino
      errores: {
        provinciaOrigen: '',
        ciudadOrigen: '',
        provinciaDestino: '',
        ciudadDestino: ''
      },
      touched: {
        provinciaOrigen: false,
        ciudadOrigen: false,
        provinciaDestino: false,
        ciudadDestino: false
      }
    };
  },
  watch: {
    datosIniciales: {
      handler(newVal) {
        Object.assign(this.reserva, newVal);
        this.resetearTocado(); // Reiniciar estado tocado
        // Re-validar y cargar ciudades después de la re-hidratación
        if (this.reserva.provinciaOrigen) {
            this.cargarCiudades('origen', false); // No marcar como tocado
        }
        if (this.reserva.provinciaDestino) {
            this.cargarCiudades('destino', false); // No marcar como tocado
        }
        this.validarCampo('provinciaOrigen');
        this.validarCampo('ciudadOrigen');
        this.validarCampo('provinciaDestino');
        this.validarCampo('ciudadDestino');
      },
      deep: true,
      immediate: true
    }
  },
  // mounted es un "hook" del ciclo de vida de Vue, se ejecuta cuando el componente se ha montado en el DOM
  async mounted() {
    this.provincias = this.provinciasMaestro;

    // Después de cargar provincias, si ya hay provincias en los datos iniciales,
    // debemos cargar sus ciudades para que aparezcan en los selects.
    if (this.reserva.provinciaOrigen) {
        await this.cargarCiudades('origen', false);
    }
    if (this.reserva.provinciaDestino) {
        await this.cargarCiudades('destino', false);
    }
  },
  methods: {
    marcarTocado(campo) {
      this.touched[campo] = true;
    },
    // NUEVO: Método para resetear el estado "tocado" de todos los campos
    resetearTocado() {
      this.touched.provinciaOrigen = false;
      this.touched.ciudadOrigen = false;
      this.touched.provinciaDestino = false;
      this.touched.ciudadDestino = false;
    },
    async cargarProvincias() {
      // Usamos la prop 'provinciasMaestro' que viene del padre, no la simulamos aquí
      this.provincias = this.provinciasMaestro;
      // No simulamos AJAX aquí porque App.vue ya las cargó y nos las pasó.
    },
    async cargarCiudades(tipo, markAsTouched = false) { // Añadir parámetro opcional
      const provinciaId = tipo === 'origen' ? this.reserva.provinciaOrigen : this.reserva.provinciaDestino;

      if (!provinciaId) {
        if (tipo === 'origen') {
          this.ciudadesOrigen = [];
          this.reserva.ciudadOrigen = '';
          this.errores.ciudadOrigen = ''; // Limpiar errores de ciudad al limpiar provincia
          this.touched.ciudadOrigen = false; // Resetear tocado
        } else {
          this.ciudadesDestino = [];
          this.reserva.ciudadDestino = '';
          this.errores.ciudadDestino = '';
          this.touched.ciudadDestino = false;
        }
        // Validar provincia solo si se marcó como tocado
        if (markAsTouched) {
            this.validarCampo(tipo === 'origen' ? 'provinciaOrigen' : 'provinciaDestino');
        }
        return;
      }

      const ciudades = this.ciudadesMaestro[provinciaId] || [];
      await new Promise(resolve => setTimeout(resolve, 300)); // Simular delay

      if (tipo === 'origen') {
        this.ciudadesOrigen = ciudades;
        if (!ciudades.some(c => c.id === this.reserva.ciudadOrigen)) {
             this.reserva.ciudadOrigen = '';
        }
        // Validar y marcar como tocado si corresponde
        if (markAsTouched) {
            this.marcarTocado('provinciaOrigen'); // Marcar la provincia como tocada si el cambio es del usuario
            this.marcarTocado('ciudadOrigen'); // Marcar la ciudad como tocada al cargar
        }
        this.validarCampo('provinciaOrigen');
        this.validarCampo('ciudadOrigen');
      } else {
        this.ciudadesDestino = ciudades;
        if (!ciudades.some(c => c.id === this.reserva.ciudadDestino)) {
            this.reserva.ciudadDestino = '';
        }
        if (markAsTouched) {
            this.marcarTocado('provinciaDestino');
            this.marcarTocado('ciudadDestino');
        }
        this.validarCampo('provinciaDestino');
        this.validarCampo('ciudadDestino');
      }
    },
    validarCampo(campo) {
      // Lógica de validación para provincias y ciudades
      // Esta lógica es similar a la de DatosPasajero, adaptada a los campos de destino
      if (campo === 'provinciaOrigen') {
        this.errores.provinciaOrigen = this.reserva.provinciaOrigen ? '' : 'Debe seleccionar una provincia de origen.';
      } else if (campo === 'ciudadOrigen') {
        this.errores.ciudadOrigen = (this.reserva.provinciaOrigen && !this.reserva.ciudadOrigen) ? 'Debe seleccionar una ciudad de origen.' : '';
      } else if (campo === 'provinciaDestino') {
        this.errores.provinciaDestino = this.reserva.provinciaDestino ? '' : 'Debe seleccionar una provincia de destino.';
      } else if (campo === 'ciudadDestino') {
        this.errores.ciudadDestino = (this.reserva.provinciaDestino && !this.reserva.ciudadDestino) ? 'Debe seleccionar una ciudad de destino.' : '';
      }
    },
    siguientePaso() {
      // Al intentar avanzar, marcamos TODOS los campos como tocados
      this.marcarTocado('provinciaOrigen');
      this.marcarTocado('ciudadOrigen');
      this.marcarTocado('provinciaDestino');
      this.marcarTocado('ciudadDestino');

      // Validar todos los campos del destino antes de avanzar
      this.validarCampo('provinciaOrigen');
      this.validarCampo('ciudadOrigen');
      this.validarCampo('provinciaDestino');
      this.validarCampo('ciudadDestino');

      const tieneErrores = Object.values(this.errores).some(error => error !== '');

      if (!tieneErrores) {
        console.log('Datos de destino válidos:', this.reserva);
        // Cuando este componente está listo, emitirá un evento al componente padre
        // indicando que puede avanzar de paso y enviando sus datos.
        this.$emit('siguiente-paso', this.reserva);
      }
    }
  }
};
</script>

<style scoped>
/* Estilos copiados de Vanilla JS para que se vea bien */
.form-group {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
  color: #555;
}

input[type="text"],
select {
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

.prev-btn {
    background-color: #6c757d;
}

.prev-btn:hover {
    background-color: #5a6268;
}
</style>
