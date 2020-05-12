<template>
  <div class="home">
    <RadarChart></RadarChart>
  </div>
</template>

<script>
// @ is an alias to /src
import RadarChart from '@/components/RadarChart.vue'

export default {
  name: 'home',
  components: {
    RadarChart
  }
}
</script>
