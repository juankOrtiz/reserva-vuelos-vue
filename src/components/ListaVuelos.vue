<template>
  <div>
    <h3>Vuelos Disponibles</h3>
    <input type="text" v-model="filtroAerolinea" placeholder="Filtrar por aerolínea..." class="filter-input" />

    <table v-if="vuelosFiltrados.length">
      <thead>
        <tr>
          <th>Aerolínea</th>
          <th>Número de Vuelo</th>
          <th>Fecha</th>
          <th>Precio</th>
          <th>Estado</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="vuelo in vuelosFiltrados" :key="vuelo.id" :class="{ 'vuelo-economico': vuelo.precio < 200 }">
          <td>{{ vuelo.aerolinea }}</td>
          <td>{{ vuelo.numeroVuelo }}</td>
          <td>{{ vuelo.fecha }}</td>
          <td>${{ vuelo.precio }}</td>
          <td>
            <span class="pill" :class="claseEstado(vuelo.estado)">{{ vuelo.estado }}</span>
          </td>
          <td>
            <button @click="seleccionarVuelo(vuelo)" :disabled="vuelo.seleccionado" class="select-btn">
              {{ vuelo.seleccionado ? 'Seleccionado' : 'Seleccionar' }}
            </button>
          </td>
        </tr>
      </tbody>
    </table>
    <p v-else id="noVuelosMessage">No hay vuelos disponibles con este filtro.</p>
  </div>
</template>

<script>
export default {
  name: 'ListaVuelos',
  data() {
    return {
      vuelos: [],
      filtroAerolinea: '', // Dato reactivo para el campo de filtro
      vueloSeleccionadoLocal: null // Para gestionar la selección dentro de este componente
    };
  },
  async mounted() {
    try {
      const resVuelos = await fetch('/data/vuelos.json');
      this.vuelos = await resVuelos.json();
      console.log('Vuelos cargados:', this.vuelos);
    } catch (error) {
      console.error('Error al cargar vuelos:', error);
      // Podrías mostrar un mensaje de error aquí
    }
  },
  // Propiedades computadas: se recalculan SOLO cuando sus dependencias cambian y su resultado se cachea.
  computed: {
    vuelosFiltrados() {
      // Depende de this.vuelos y this.filtroAerolinea
      if (!this.filtroAerolinea) {
        return this.vuelos; // Si no hay filtro, devuelve todos los vuelos
      }
      const filtroMinusculas = this.filtroAerolinea.toLowerCase();
      return this.vuelos.filter(vuelo =>
        vuelo.aerolinea.toLowerCase().includes(filtroMinusculas)
      );
    }
  },
  methods: {
    seleccionarVuelo(vuelo) {
        // Deseleccionar el vuelo previamente seleccionado si existe
        if (this.vueloSeleccionadoLocal && this.vueloSeleccionadoLocal.id !== vuelo.id) {
            const prevVuelo = this.vuelos.find(v => v.id === this.vueloSeleccionadoLocal.id);
            if (prevVuelo) {
                prevVuelo.seleccionado = false; // Desmarcar el anterior
            }
        }

        // Marcar el vuelo actual como seleccionado
        vuelo.seleccionado = true;
        this.vueloSeleccionadoLocal = vuelo; // Actualizar el vuelo seleccionado localmente

        console.log('Vuelo seleccionado:', vuelo);
        // Emitir el vuelo seleccionado al componente padre (App.vue)
        this.$emit('vuelo-seleccionado', vuelo);
    },
    // Método para devolver la clase CSS adecuada para la píldora de estado
    claseEstado(estado) {
        return {
            'pill-confirmado': estado === 'Confirmado',
            'pill-demorado': estado === 'Demorado',
            'pill-cancelado': estado === 'Cancelado'
        };
    }
  }
};
</script>

<style scoped>
/* Estilos de la tabla, copiados de Vanilla JS */
table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}
th, td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}
th {
  background-color: #f2f2f2;
}
.vuelo-economico {
  background-color: #e6ffe6; /* Fondo verde claro para vuelos económicos */
}
.pill {
    padding: 3px 8px;
    border-radius: 15px;
    font-size: 0.8em;
    font-weight: bold;
    color: white;
    margin-left: 5px;
    display: inline-block; /* Para que la píldora se vea bien */
}
.pill-confirmado { background-color: green; }
.pill-demorado { background-color: orange; }
.pill-cancelado { background-color: red; }

.filter-input {
    width: calc(100% - 20px);
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 4px;
    box-sizing: border-box;
    font-size: 1em;
    margin-bottom: 15px;
}

.select-btn {
  background-color: #28a745; /* Color verde para seleccionar */
}

.select-btn:hover {
  background-color: #218838;
}

.select-btn:disabled {
    background-color: #6c757d; /* Gris para deshabilitado */
    cursor: not-allowed;
}
#noVuelosMessage {
  text-align: center;
  margin-top: 20px;
  color: #666;
}
</style>
