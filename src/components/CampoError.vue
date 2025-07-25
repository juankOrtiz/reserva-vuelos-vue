<template>
  <p v-show="mensaje" class="error-feedback">{{ mensaje }}</p>
</template>

<script>
export default {
  name: 'CampoError',
  props: {
    mensaje: {
      type: String,
      default: '' // Por defecto, no hay mensaje
    }
  }
};
</script>

<style scoped>
.error-feedback {
  color: red;
  font-size: 0.85em;
  margin: 0;
  padding-top: 5px;
  height: 20px; /* Altura fija para el espacio del mensaje */
  line-height: 1.2;
  overflow: hidden;
  opacity: 1;
  visibility: visible;
  transition: opacity 0.3s ease-in-out, visibility 0.3s ease-in-out;
}

.error-feedback[style*="display: none"] {
  opacity: 0;
  visibility: hidden;
}
</style>
