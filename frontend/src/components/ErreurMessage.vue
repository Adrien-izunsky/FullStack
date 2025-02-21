<template>
  <div class="erreur-message">
    <p>{{ message }}</p>
  </div>
</template>

<script>
export default {
  props: {
    message: String,
  },
};
</script>

<style scoped>
.erreur-message {
  color: red;
  font-weight: bold;
}
</style>
