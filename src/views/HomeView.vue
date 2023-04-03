<template>
  <main>
    <Pin :length="4" @complete="onComplete" />
  </main>
</template>

<script setup lang="ts">
import Pin from '@/components/Pin.vue'

const onComplete = (value: string) => {
  alert(`Congratulation, your pin is: ${value}`)
}
</script>
