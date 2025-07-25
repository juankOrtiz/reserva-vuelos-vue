<template>
  <form @submit.prevent="siguientePaso">
    <h2>Paso 2: Destino del Vuelo</h2>
    <div class="form-group">
      <label for="provinciaOrigen">Provincia de Origen:</label>
      <select id="provinciaOrigen" v-model="reserva.provinciaOrigen" @change="cargarCiudades('origen')">
        <option value="">Seleccione una provincia</option>
        <option v-for="provincia in provincias" :key="provincia.id" :value="provincia.id">
          {{ provincia.nombre }}
        </option>
      </select>
      <p v-if="errores.provinciaOrigen" class="error-message">{{ errores.provinciaOrigen }}</p>
    </div>

    <div class="form-group" v-if="reserva.provinciaOrigen">
      <label for="ciudadOrigen">Ciudad de Origen:</label>
      <select id="ciudadOrigen" v-model="reserva.ciudadOrigen" :disabled="ciudadesOrigen.length === 0">
        <option value="">Seleccione una ciudad</option>
        <option v-for="ciudad in ciudadesOrigen" :key="ciudad.id" :value="ciudad.id">
          {{ ciudad.nombre }}
        </option>
      </select>
      <p v-if="errores.ciudadOrigen" class="error-message">{{ errores.ciudadOrigen }}</p>
    </div>

    <div class="form-group">
      <label for="provinciaDestino">Provincia de Destino:</label>
      <select id="provinciaDestino" v-model="reserva.provinciaDestino" @change="cargarCiudades('destino')">
        <option value="">Seleccione una provincia</option>
        <option v-for="provincia in provincias" :key="provincia.id" :value="provincia.id">
          {{ provincia.nombre }}
        </option>
      </select>
      <p v-if="errores.provinciaDestino" class="error-message">{{ errores.provinciaDestino }}</p>
    </div>

    <div class="form-group" v-if="reserva.provinciaDestino">
      <label for="ciudadDestino">Ciudad de Destino:</label>
      <select id="ciudadDestino" v-model="reserva.ciudadDestino" :disabled="ciudadesDestino.length === 0">
        <option value="">Seleccione una ciudad</option>
        <option v-for="ciudad in ciudadesDestino" :key="ciudad.id" :value="ciudad.id">
          {{ ciudad.nombre }}
        </option>
      </select>
      <p v-if="errores.ciudadDestino" class="error-message">{{ errores.ciudadDestino }}</p>
    </div>

    <button type="button" @click="$emit('paso-anterior')">Anterior</button>
    <button type="submit">Siguiente</button>
  </form>
</template>

<script>
export default {
  name: 'DatosDestino',
  data() {
    return {
      reserva: {
        provinciaOrigen: '',
        ciudadOrigen: '',
        provinciaDestino: '',
        ciudadDestino: ''
      },
      provincias: [], // Aquí almacenaremos las provincias obtenidas de la "API"
      ciudadesOrigen: [], // Ciudades filtradas para la provincia de origen
      ciudadesDestino: [], // Ciudades filtradas para la provincia de destino
      // Esto simula todas las ciudades mapeadas por ID de provincia, como si vinieran de la BD
      todasLasCiudadesMaestro: {
        1: [{ id: 101, nombre: 'Capital Federal' }, { id: 102, nombre: 'La Plata' }],
        2: [{ id: 201, nombre: 'Córdoba Capital' }, { id: 202, nombre: 'Villa Carlos Paz' }],
        3: [{ id: 301, nombre: 'Rosario' }, { id: 302, nombre: 'Santa Fe Capital' }]
      },
      errores: {
        provinciaOrigen: '',
        ciudadOrigen: '',
        provinciaDestino: '',
        ciudadDestino: ''
      }
    };
  },
  // mounted es un "hook" del ciclo de vida de Vue, se ejecuta cuando el componente se ha montado en el DOM
  async mounted() {
    await this.cargarProvincias();
  },
  methods: {
    async cargarProvincias() {
      // Simulación de una llamada AJAX para obtener provincias
      console.log('Cargando provincias...');
      this.provincias = await new Promise(resolve => {
        setTimeout(() => {
          resolve([
            { id: 1, nombre: 'Buenos Aires' },
            { id: 2, nombre: 'Córdoba' },
            { id: 3, nombre: 'Santa Fe' }
          ]);
        }, 500); // Retraso de 0.5 segundos para simular carga
      });
      console.log('Provincias cargadas:', this.provincias);
    },
    async cargarCiudades(tipo) {
      const provinciaId = tipo === 'origen' ? this.reserva.provinciaOrigen : this.reserva.provinciaDestino;

      if (!provinciaId) {
        // Si no hay provincia seleccionada, limpiar y deshabilitar las ciudades
        if (tipo === 'origen') {
          this.ciudadesOrigen = [];
          this.reserva.ciudadOrigen = ''; // También limpiar la ciudad seleccionada
        } else {
          this.ciudadesDestino = [];
          this.reserva.ciudadDestino = '';
        }
        this.validarCampo(tipo === 'origen' ? 'provinciaOrigen' : 'provinciaDestino');
        return;
      }

      // Simulación de una llamada AJAX para cargar ciudades por provincia
      console.log(`Cargando ciudades para provincia ID: ${provinciaId} (${tipo})...`);
      // En un escenario real, harías una llamada HTTP a tu backend aquí
      // Por ejemplo: const response = await fetch(`/api/ciudades?provinciaId=${provinciaId}`);
      // const ciudades = await response.json();

      // Usamos nuestros datos maestros simulados
      const ciudades = await new Promise(resolve => {
        setTimeout(() => {
          resolve(this.todasLasCiudadesMaestro[provinciaId] || []);
        }, 300); // Retraso de 0.3 segundos para simular carga
      });

      if (tipo === 'origen') {
        this.ciudadesOrigen = ciudades;
        this.reserva.ciudadOrigen = ''; // Resetear ciudad al cambiar provincia
        this.validarCampo('provinciaOrigen'); // Re-validar por si la provincia estaba vacía
        this.validarCampo('ciudadOrigen'); // Re-validar la ciudad por si se había seleccionado algo inválido
      } else {
        this.ciudadesDestino = ciudades;
        this.reserva.ciudadDestino = ''; // Resetear ciudad al cambiar provincia
        this.validarCampo('provinciaDestino');
        this.validarCampo('ciudadDestino');
      }
      console.log(`Ciudades cargadas para ${tipo}:`, ciudades);
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
