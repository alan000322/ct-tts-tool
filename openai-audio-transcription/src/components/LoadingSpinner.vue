<template>
  <div class="flex justify-center items-center mt-4">
    <div class="loader ease-linear rounded-full border-8 border-t-8 border-gray-200 h-16 w-16"></div>
  </div>
</template>

<script>
export default {
  name: 'LoadingSpinner',
};
</script>

<style scoped>
.loader {
  border-top-color: #3490dc;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
</style>
