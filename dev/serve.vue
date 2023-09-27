<script>
import Vue from 'vue'
import bunnyImage from '@/bunny-image.vue'

export default Vue.extend({
  name: 'ServeDev',
  components: {
    bunnyImage,
  },
})
</script>

<template>
  <div id="app">
    <div>
      <bunny-image
        src="/sample.jpg"
        crop="fill"
        :width="200"
        :aspect-ratio="1"
        :transforms="{
          a: '90',
        }"
      />
    </div>

    <div>
      <bunny-image
        :style="{
          width: '100px',
          height: '200px',
          objectFit: 'cover',
        }"
        src="/sample.jpg"
        crop="fill"
        :aspect-ratio="2"
        :transforms="{
          a: '90',
        }"
      />
    </div>
  </div>
</template>
