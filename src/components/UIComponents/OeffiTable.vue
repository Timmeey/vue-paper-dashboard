<template>
  <div>
    <div class="header">
      <slot name="header">
        <h4 class="title">{{station}}</h4>
      </slot>
    </div>
    <div class="content table-responsive table-full-width">
      <table class="table table-striped">
        <thead>
        <th v-for="column in columns">{{column}}</th>
        </thead>
        <tbody>
        <tr v-for="item in limitedDepartures">
          <td :class="departureClass(item.departure)">
            {{formatDeparture(item.departure)}}
          </td>
          <td>{{item.name}}</td>
          <td>{{item.destination}}</td>
        </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>
<script>
  import axios from 'axios'
  export default {

    props: {
      vehicles: Array,
      station: {
        type: String,
        default: ''
      },
      walkTime: {
        type: Number,
        default: 3
      }
    },
    data () {
      return {
        columns: ['Depart', 'Line', 'Destination'],
        departures: []
      }
    },
    computed: {
      limitedDepartures: function () {
        var result = []
        for (var i in this.departures) {
          if (this.departures[i].departure > this.walkTime) {
            result.push(this.departures[i])
          }
        }
        return result
      }
    },
    methods: {
      hasValue (item, column) {
        return item[column] !== 'undefined'
      },
      itemValue (item, column) {
        return item[column]
      },
      loadData: function () {
        var tis = this
        const params = new URLSearchParams()
        for (var i in this.vehicles) {
          params.append('vehicle', this.vehicles[i])
        }
        var request = {
          params: params
        }
        axios.get('http://localhost:8081/oeffi/stations/' +
          this.station + '/departures', request)
          .then(function (response) {
            tis.departures = response.data
          })
      },
      formatDeparture (departure) {
        if (departure < 1) {
          return 'NOW'
        } else {
          return departure + 'min'
        }
      },
      departureClass (departure) {
        if (departure < 2) {
          return 'departure-immediate'
        } else {
          return 'departure'
        }
      }
    },
    mounted: function () {
      this.loadData()
      setInterval(function () {
        this.loadData()
      }.bind(this), 5000)
    }
  }

</script>
<style>

</style>
