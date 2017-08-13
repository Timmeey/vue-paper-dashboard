<template>
  <div class="card">
    <div class="content">
      <div class="row">
        <div class="col-xs-5">
          <div class="icon-big text-center"
               v-bind:class="{ 'icon-warning': switchedOn }">
            <i :class="`ti-light-bulb`"></i>
            <toggle-button :sync="true"
                           :labels="{checked:name +' On', unchecked:name +' Off'}"
                           @change="toggle($event.value)"
                           :width="60"
                           :value="switchedOn"
                           :color="hexColor"
            />
          </div>
        </div>
        <div class="col-xs-7">
          <div name="content">
            <color-picker :change="updateColor"
                          :active="switchedOn"></color-picker>


          </div>
        </div>
      </div>
      <div class="footer">
        <hr/>
        <div name="footer"></div>
      </div>
    </div>
  </div>
</template>

<script>
  import ColorPicker from "../ColorPicker";
  import axios from 'axios'

  export default {
    components: {ColorPicker},
    name: 'light-card',
    props: {
      name: {
        type: String,
        default: 'all'
      },
      restUrl: String
    },
    data () {
      return {
        switchedOn: true,
        color: Array,
        hexColor: ""
      }
    },
    methods: {
      toggle (value) {
        this.switchedOn = value
      },
      updateColor (event) {
        this.color = event.color
        this.hexColor = rgbToHex(this.color.r, this.color.g, this.color.b)
        this.sendColor(this.color)
      },
      sendColor(color, fading) {
        if (fading === undefined) {
          fading = 250
        }
        axios.post(this.restUrl, {
          "red": color.r,
          "green": color.g,
          "blue": color.b,
          "fading": fading
        })
          .then(function (response) {
          })
      }
    },
    watch: {
      switchedOn: function (switchedOn) {
        if (!switchedOn) {
          this.sendColor({"r": 0, "g": 0, "b": 0})
        } else {
          this.sendColor(this.color)
        }
      }
    }

  }
  function componentToHex(c) {
    var hex = c.toString(16);
    return hex.length === 1 ? "0" + hex : hex;
  }

  function rgbToHex(r, g, b) {
    return "#" + componentToHex(r) + componentToHex(g) + componentToHex(b);
  }
</script>

<style>

</style>
