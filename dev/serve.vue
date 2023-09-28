<script>
import Vue from 'vue'
import BunnyImage from '@/bunny-image.vue'

export default Vue.extend({
  name: 'ServeDev',
  components: {
    BunnyImage,
  },
})
</script>

<template>
  <div id="app">
    <div>
      <BunnyImage
        src="/assets/Horgen_2019_W_Seegartenstr.-84_80.63.624_8063624_22.jpg"
        crop="fill"
        :width="200"
        :aspect-ratio="1"
      />
    </div>

    <div>
      <BunnyImage
        :style="{
          width: '100px',
          height: '200px',
          objectFit: 'cover',
        }"
        src="/assets/Horgen_2019_W_Seegartenstr.-84_80.63.624_8063624_22.jpg"
        :aspect-ratio="0.5"
        :brightness="30"
        :flop="true"
      />
    </div>

    <div>
      <BunnyImage
        :width="200"
        src="assets/Horgen_2019_W_Seegartenstr.-84_80.63.624_8063624_22.jpg"
        :aspect-ratio="1.5"
      />
    </div>

    <div>
      <BunnyImage
        :width="200"
        src="assets/Horgen_2019_W_Seegartenstr.-84_80.63.624_8063624_22.jpg"
        aspect-ratio="3:2"
      />
    </div>
  </div>
</template>
