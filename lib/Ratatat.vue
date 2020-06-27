<template>
  <div
    class="drumroll-container"
    :style="{
      height: height,
      width: width,
    }"
  >
    <div class="drumroll-animation"></div>
  </div>
</template>

<script>
import * as lottie from "lottie-web";
const AnimationTimer = require("animation-timer").AnimationTimer;
const Easer = require("functional-easing").Easer;

export default {
  name: "Ratatat",
  props: {
    animationData: {
      type: Object,
      default: () => {
        return null;
      },
    },
    options: {
      type: Object,
      default: () => {
        return {
          animType: "svg",
          loop: true,
          prerender: true,
          autoplay: true,
        };
      },
    },
    percent: {
      type: Number,
      default: 0,
    },
    duration: {
      type: [Number, String],
      default: 400,
    },
    autoAdjust: {
      type: Boolean,
      default: false,
    },
    timing: {
      type: String,
      default: "in-quart",
    },
    layer: {
      type: String,
      default: "",
    },
    slider: {
      type: String,
      default: "",
    },
    speed: {
      type: Number,
      default: 1,
    },
    direction: {
      type: Number,
      default: 1,
    },
    height: {
      type: String,
      default: "",
    },
    width: {
      type: String,
      default: "",
    },
  },
  data: () => ({
    defOpts: {
      animType: "svg",
      loop: true,
      prerender: true,
      autoplay: true,
    },
    animData: null,
    animAPI: null,
    realPercent: 0,
  }),
  watch: {
    percent(val, lastVal) {
      if (isNaN(lastVal)) lastVal = 0;
      if (lastVal !== 100) {
        this.createAnimationTimer(val, lastVal);
      } else this.reset();
    },
    realPercent(val) {
      this.$emit("update", val);
    },
  },
  computed: {
    realOpts() {
      let temp = this.options;
      Object.assign(temp, this.defOpts);
      temp["wrapper"] = this.$el.children[0];
      temp["animationData"] = this.animationData
        ? this.animationData
        : require("../animations/wings.json");
      return temp;
    },
  },
  mounted() {
    require("./lottie_api.js");
    this.init();
    this.$emit("mounted");
  },
  destroyed() {
    this.animData.destroy();
    this.$emit("destroyed");
  },
  methods: {
    reset() {
      this.realPercent = 0;
    },
    createAnimationTimer(to, from) {
      const self = this;
      let easer;

      if (this.timing.length) {
        let temp = this.timing;
        easer = new Easer().using(temp);
      } else {
        easer = new Easer().using("in-out-quart");
      }
      const animation = new AnimationTimer()
        .duration(this.duration)
        .on(
          "tick",
          easer((time) => {
            if (time <= 0) this.$emit("start");
            else if (time >= 1) this.$emit("end");
            let temp = Math.floor((to - from) * time);
            let real = temp + from;
            if (self.realPercent !== real) self.realPercent = real;
          })
        )
        .play();
    },
    init() {
      if (!this.animationData) return null;
      this.animData = this.buildAnimation();
      this.animAPI = lottie_api.createAnimationApi(this.animData);
      let tempSlider = this.animAPI.getKeyPath(
        `${this.layer},Effects,${this.slider},0`
      );
      this.animAPI.addValueCallback(tempSlider, (currentVal) => {
        return this.realPercent;
      });
      this.registerAllEvents();
      this.animData.setSpeed(this.speed);

      this.animData.play();
    },
    buildAnimation() {
      return lottie.loadAnimation(this.realOpts);
    },
    registerAllEvents() {
      const self = this;
      let evts = ["complete", "DOMLoaded"];
      evts.forEach((evt) => {
        this.animData.addEventListener(evt, (val) => {
          self.$emit(evt, val);
        });
      });
    },
  },
};
</script>

<style>
.drumroll-container > svg {
  width: 100%;
}

.drumroll-container {
  min-width: 1px;
  min-height: 1px;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
}
</style>
