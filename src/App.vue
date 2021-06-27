<template>
  <FooBar />
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import FooBar from './components/FooBar.vue'

export default defineComponent({
  components: {
    FooBar
  },
})
</script>
