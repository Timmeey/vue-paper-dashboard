<template id="color-picker-template">

  <div class="color-picker">
    <transition name="fade">
      <div class="color-picker__inner" v-show="active">
        <div class="control" v-bind:style="gradientH">
          <input type="range" min="0" max="360" v-model="h"/>
        </div>
        <div class="control" v-bind:style="gradientL">
          <input type="range" min="0" max="100" v-model="l"/>
        </div>
      </div>
      {{initialHue}},{{initialLightness}}
    </transition>
    <div v-bind:style="{'background': color}"></div>
  </div>
</template>
<script>
  export default {
    name: 'color-picker',
    template: "#color-picker-template",
    props: ['change', 'initialRGB', "active"],
    data: function () {
      return {
        isVisible: true,
        h: this.initialHue,
        s: 100,
        l: this.initialLightness,
        loaded: false
      }
    },
    computed: {
      initialHue: function () {
        var hue = rgbToHsl(this.initialRGB[0], this.initialRGB[1], this.initialRGB[2]).h;
        this.h = hue;
        return hue;
      },
      initialLightness: function () {
        var lightness = rgbToHsl(this.initialRGB[0], this.initialRGB[1], this.initialRGB[2]).l * 2;
        this.l = lightness;
        return lightness;
      },
      color: function () {
        var hsl = hsb2hsl(parseFloat(this.h) / 360, parseFloat(this.s) / 100, parseFloat(this.l) / 100)

        // var c = hsl.h + ", " + hsl.s + "%, " + hsl.l + "%";

        var s = hslToRgb(hsl.h / 360, hsl.s / 100, hsl.l / 100)
        this.change({
          color: s
        });

        return s;
      },
      colorString: function () {
        var c = this.h + ", " + this.s + "%, " + this.l + "%"
        return c;
      },
      gradientH: function () {
        var stops = [];
        for (var i = 0; i < 7; i++) {
          var h = i * 60;

          var hsl = hsb2hsl(parseFloat(h / 360), parseFloat(this.s) / 100, parseFloat(100 / 100))

          var c = hsl.h + ", " + hsl.s + "%, " + hsl.l + "%"
          stops.push("hsl(" + c + ")")
        }

        return {
          backgroundImage: "linear-gradient(to right, " + stops.join(', ') + ")"
        }
      },
      gradientS: function () {
        var stops = [];
        var c;
        var hsl = hsb2hsl(parseFloat(this.h / 360), 0, parseFloat(this.l / 100))
        c = hsl.h + ", " + hsl.s + "%, " + hsl.l + "%"
        stops.push("hsl(" + c + ")")

        var hsl2 = hsb2hsl(parseFloat(this.h / 360), 1, parseFloat(this.l / 100))
        c = hsl2.h + ", " + hsl2.s + "%, " + hsl2.l + "%"
        stops.push("hsl(" + c + ")")

        return {
          backgroundImage: "linear-gradient(to right, " + stops.join(', ') + ")"
        }
      },

      gradientL: function () {
        var stops = [];
        var c;

        var hsl = hsb2hsl(parseFloat(this.h / 360), 0, 0)
        c = hsl.h + ", " + hsl.s + "%, " + hsl.l + "%"
        stops.push("hsl(" + c + ")")

        var hsl2 = hsb2hsl(parseFloat(this.h / 360), parseFloat(this.s / 100), 1)

        c = hsl2.h + ", " + hsl2.s + "%, " + hsl2.l + "%"
        stops.push("hsl(" + c + ")")

        return {
          backgroundImage: "linear-gradient(to right, " + stops.join(', ') + ")"

        }
      }
    },
    methods: {

      show: function () {
        this.isVisible = true;
      },
      hide: function () {
        this.isVisible = false;
      },
      toggle: function () {
        this.isVisible = !this.isVisible;
      }
    },

    mounted: function () {
    }
  }

  function hsb2hsl(h, s, b) {
    var hsl = {
      h: h
    };
    hsl.l = (2 - s) * b;
    hsl.s = s * b;

    if (hsl.l <= 1 && hsl.l > 0) {
      hsl.s /= hsl.l;
    } else {
      hsl.s /= 2 - hsl.l;
    }

    hsl.l /= 2;

    if (hsl.s > 1) {
      hsl.s = 1;
    }

    if (!hsl.s > 0) hsl.s = 0

    hsl.h *= 360;
    hsl.s *= 100;
    hsl.l *= 100;

    return hsl;
  }

  function hslToRgb(h, s, l) {
    var r, g, b;

    if (s === 0) {
      r = g = b = l; // achromatic
    } else {
      var hue2rgb = function hue2rgb(p, q, t) {
        if (t < 0) t += 1;
        if (t > 1) t -= 1;
        if (t < 1 / 6) return p + (q - p) * 6 * t;
        if (t < 1 / 2) return q;
        if (t < 2 / 3) return p + (q - p) * (2 / 3 - t) * 6;
        return p;
      }

      var q = l < 0.5 ? l * (1 + s) : l + s - l * s;
      var p = 2 * l - q;
      r = hue2rgb(p, q, h + 1 / 3);
      g = hue2rgb(p, q, h);
      b = hue2rgb(p, q, h - 1 / 3);
    }

    return {
      r: Math.round(r * 255),
      g: Math.round(g * 255),
      b: Math.round(b * 255)
    };
  }

  function rgbToHsl(r, g, b) {
    r /= 255
    g /= 255
    b /= 255
    var max = Math.max(r, g, b), min = Math.min(r, g, b);
    var h, s, l = (max + min) / 2;

    if (max === min) {
      h = s = 0; // achromatic
    } else {
      var d = max - min;
      s = l > 0.5 ? d / (2 - max - min) : d / (max + min);
      switch (max) {
        case r:
          h = (g - b) / d + (g < b ? 6 : 0);
          break;
        case g:
          h = (b - r) / d + 2;
          break;
        case b:
          h = (r - g) / d + 4;
          break;
      }
      h /= 6;
    }
    h *= 360
    s *= 100
    l *= 100
    return {h: h, s: s, l: l};
  }
</script>
<style>
  .color-picker__inner {
    padding: 1.5rem 1rem;
  }

  .control {
    width: 100%;
    height: 12px;
    border-radius: 12px;
    border: 1px solid #ddd;
  }

  .control + .control {
    margin-top: 1rem;
  }

  .control input {
    width: 100%;
    margin: 0;
  }

  .control input[type=range] {
    -webkit-appearance: none;
    width: 100%;
    background: transparent;
  }

  .control input[type=range]:focus {
    outline: none;
  }

  .control input[type=range]::-ms-track {
    width: 100%;
    cursor: pointer;
    background: transparent;
    border-color: transparent;
    color: transparent;
  }

  .control input[type=range]::-webkit-slider-thumb {
    -webkit-appearance: none;
    border: 1px solid #ddd;
    height: 20px;
    width: 20px;
    border-radius: 50px;
    background: #ffffff;
    cursor: pointer;
    box-shadow: 0px 1px 2px rgba(0, 0, 0, 0.12);
    margin-top: -4px;
  }

  .swatch {
    width: 24px;
    height: 24px;
    margin: 1rem auto 0 auto;
    border: 4px solid white;
    box-shadow: 0px 0px 1px rgba(0, 0, 0, 0.3);
    cursor: pointer;
  }

  .swatch:hover {
    border: 4px solid white;
    opacity: 0.8;
    box-shadow: 0px 0px 1px rgba(0, 0, 0, 0.3);
  }

  .pop-enter-active,
  .pop-leave-active {
    transition: transform .5s, opacity .5s;
    transition-timing-function: cubic-bezier(.8, .3, 0.25, 1.75);
    transform: scale(1);
  }

  .pop-enter,
  .pop-leave-active {
    opacity: 0;
    transform: scale(0);
  }
</style>
