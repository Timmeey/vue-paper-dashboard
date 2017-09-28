<template>
  <div class="row">
    <div v-for="station in stations" class="col-xs-12 col-sm-6 col-md-5">
      <div class="card">
        <oeffi-table :station="station[0]" :walkTime="station[1]"
                     :vehicles="station[2]">
        </oeffi-table>
      </div>
    </div>
  </div>
</template>
<script>
  import OeffiTable from 'components/UIComponents/OeffiTable.vue'

  export default {
    components: {
      OeffiTable
    },
    data () {
      return {
        stations: [['Björnsonstr. (Berlin)', 3, ['Tram']],
          ['S Bornholmer Str. (Berlin)', 7, ['S']]]
      }
    }
  }

</script>
<style>

</style>
