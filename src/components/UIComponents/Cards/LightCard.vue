<template>
  <div class="card">
    <div class="content" v-show="loaded">
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
                          :active="switchedOn"
                          :initialRGB="initialColor"></color-picker>


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
  import config from "config"

  export default {
    components: {ColorPicker},
    name: 'light-card',
    props: {
      name: {
        type: String,
        default: 'all'
      },
      lightName: String
    },
    data () {
      return {
        switchedOn: false,
        color: Array,
        hexColor: "",
        initialColor: [],
        loaded: false
      }
    },
    methods: {
      toggle (value) {
        this.switchedOn = value
        if (value === false) {
          this.switchOff()
        }
      },
      switchOff (){
        this.sendColor({"r": 0, "g": 0, "b": 0})
      },
      updateColor (event) {
        this.color = event.color
        this.hexColor = rgbToHex(this.color.r, this.color.g, this.color.b)
        if (this.switchedOn && this.loaded) {
          this.sendColor(this.color)
        }
      },
      sendColor(color, fading) {
        if (fading === undefined) {
          fading = 250
        }
        axios.post(config.backendUrl + '/lights/' + this.lightName, {
          "red": color.r,
          "green": color.g,
          "blue": color.b,
          "fading": fading
        }).then(function (response) {
        })
      }
    },
    mounted: function () {
      console.log(this.lightName)
      var tis = this;
      axios.get(config.backendUrl + '/lights/' + this.lightName)
        .then(function (response) {
          var color = response.data.color;
          tis.initialColor = [color.red, color.green, color.blue];
          console.log(tis.initialColor)
          if (tis.initialColor[0] === 0 && tis.initialColor[1] === 0 && tis.initialColor[2] === 0) {
            tis.switchedOn = false
          } else {
            tis.switchedOn = true
          }
          tis.loaded = true;
        })
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
