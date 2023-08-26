<script setup lang="ts">
const router = useRouter()
</script>

<template>
  <div class="container-404">
    <span>Not Found</span>
    <button btn @click="router.back()">
      Back
    </button>
  </div>
</template>

<style scoped>
.container-404 {
  padding: 4rem;
  text-align: center;
}

span {
  margin-right: 1rem;
}
</style>

<route lang="yaml">
meta:
  layout: 404
</route>
