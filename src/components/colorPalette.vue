<script>
// import _ from "lodash";
import * as d3 from "d3";
// import legend from 'd3-svg-legend';
import { legendColor } from "d3-svg-legend";
export default {
  name: "ColorPalette",
  props: {
    myInfo: {
      type: Object,
      default: function() {
        return {
          id: "myLegendId",
          class: "myLegendClass"
        };
      }
    },
    colorScheme: {
      type: Array,
      default: function() {
        return ["#5ee7df", "#b490ca"];
      }
    },
    canvas: {
      type: Object,
      default: function() {
        return {
          width: "300px",
          height: "500px"
        };
      }
    },
    legendStats: {
      type: Object,
      default: function() {
        return {
          legendTextFormat: d3.format(".0f"),
          titleWidth: 200,
          scaleType: "quantile",
          moveStep: {
            x: 20,
            y: 20
          }
        };
      }
    },
    rawData: {
      type: Object,
      default: function() {
        return {
          key: null,
          value: []
        };
      }
    }
  },
  data() {
    return {
      height: 120,
      margin: { top: 20, right: 20, bottom: 20, left: 20 },
      quantize: function() {},
      threshold: []
    };
  },
  mounted() {
    this.calculateData();
    this.calculateScales();
    this.initLegend();
  },
  computed: {},
  methods: {
    calculateScales: function() {
      const minVal = d3.min(this.threshold);
      const maxVal = d3.max(this.threshold);
      const xDomain = [minVal, maxVal];
      // this.xScale = d3.scaleLinear().domain(this.threshold).nice()
      //                 .range([0,200])
      this.quantize = d3
        .scaleQuantize()
        .domain(xDomain)
        .range(this.colorScheme);
    },
    calculateData: function() {
      const myKey = this.rawData.key;
      this.threshold = this.rawData.value.map(d => d[myKey]);
      return this.threshold;
    },

    initLegend() {
      let legend = legendColor()
        .labelFormat(this.legendStats.legendTextFormat)
        .title("Legend")
        .titleWidth(this.legendStats.titleWidth)
        .scale(this.quantize);
      d3.select(this.$refs.legend).call(legend);
    }
  }
};
</script>
<style scoped>
.legendQuant {
}
</style>
<template>
  <svg
    :width="canvas.width"
    :height="canvas.height"
    :id="myInfo.id"
    :class="myInfo.class"
  >
    <g
      class="legendQuant"
      ref="legend"
      :width="canvas.width"
      :height="canvas.height"
      :transform="
        `translate(${legendStats.moveStep.x}, ${legendStats.moveStep.y})`
      "
    />
    <!-- <defs>
      <linearGradient id="linear-gradient" x1="0%" y1="0%" x2="100%" y2="0">
        <stop v-for="(d,i) in colorScheme" :key="i" :offset="i/colorScheme.length" :stop-color="d" />
      </linearGradient>
    </defs>
    <rect :width="canvas.width" :height="canvas.height" fill="url('#linear-gradient')" />
    <g ref="xAxis" :transform="`translate(0, ${height - margin.bottom})`" />-->
  </svg>
</template>
