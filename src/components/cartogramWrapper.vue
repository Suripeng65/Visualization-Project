<script>
import _ from "lodash";
import * as d3 from "d3";

const width = 500;
const height = 500;
const margin = { top: 20, right: 20, bottom: 20, left: 20 };
export default {
  name: "cartogramWrapper",
  components: {},
  props: {
    myid: {
      stype: String,
      default: "myCartogramId"
    },
    canvas: {
      type: Object,
      default: function() {
        return {
          width: 600,
          height: 600
        };
      }
    },
    gridKey: {
      type: String,
      default: "state"
    },
    gridArray: {
      type: Array,
      default: function() {
        return [];
      }
    },
    dataKey: {
      type: String,
      default: "score_st"
    },
    cartogramData: {
      type: Array,
      default: function() {
        return [];
      }
    },
    // colorPick: {
    //   type: Array,
    //   default: function() {
    //     return ["purple", "orange"];
    //   }
    // },
    colorScalePick: {
      type: Object,
      default: function() {
        return {
          scaleTyle: "sequential", //can take value of sequential/quantile
          colorSchema: ["#88d3ce", "#6e45e2"]
        };
      }
    },
    regionCache: {
      type: Object,
      default: function() {
        return {};
      }
    },
    // quantileColorScalePick: {
    //   type: Object,
    //   default: function() {
    //     return {
    //       on: true,
    //       colorSchema: ["rgb(244, 151, 11)", "rgb(201, 94, 55)", "rgb(190, 80, 66)", "purple"]
    //     };
    //   }
    // },
    ifTooltip: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      width,
      height,
      margin,
      step: null,
      position: [],
      boxData: [],
      colorScale: [],
      savedId: {},
      selectedBox: [],
      hovered: null,
      mouse: {
        x: 0,
        y: 0
      },
      element_bbox: {},
      empty_fill: "#808080",
      shape: "circle"
    };
  },
  computed: {},
  mounted() {
    this.calculateScales();
    this.calculateData();
    if (this.regionCache[this.gridKey]) {
      if (this.regionCache[this.gridKey].selection) {
        this.$nextTick(() => {
          this.retrieveCache(this.regionCache[this.gridKey]);
        });
      }
    }
  },
  watch: {
    cartogramData() {
      this.calculateScales();
      this.calculateData();
    },
    gridArray() {
      this.calculateScales();
      this.calculateData();
    }
  },
  methods: {
    calculateData() {
      if (!this.gridArray.length) return;
      const boxWidth =
        (this.canvas.width - margin.left - margin.right) /
        this.gridArray[0].length;
      const boxHeight =
        (this.canvas.height - margin.top - margin.bottom) /
        this.gridArray.length;
      const boxSide = Math.min(boxWidth, boxHeight);

      // let x = 0;
      // let y = 0;
      let result = [];
      this.step = this.gridArray.reduce(function(
        accumulator,
        currentValue,
        currentIndex
      ) {
        currentValue.reduce(function(a, cv, ci) {
          if (cv.substring(0, 1) !== "_")
            result.push({ region: cv, xStep: ci, yStep: currentIndex });
        }, []);
        return result;
      },
      []);
      const vm = this;
      this.position = this.step.map((d, i) => {
        const xMove = d.xStep * boxSide;
        const yMove = d.yStep * boxSide;
        // const { state, ...item } = d;
        return {
          id: i,
          ...d,
          x: xMove,
          y: yMove,
          width: boxSide,
          height: boxSide,
          ...vm.cartogramData.find(item => {
            return item[this.gridKey] === d.region && item;
          })
        };
      });
      this.boxData = this.position.map(d => {
        let fill;
        if (d[this.dataKey] > 0) fill = this.colorScale(d[this.dataKey]);
        //edge case: assign color for missing data
        else fill = this.empty_fill;
        return {
          ...d,
          fill
        };
      });

      return this.boxData;
    },
    calculateScales() {
      if (!this.gridArray.length) return;
      // if (this.quantileColorScalePick.on) {
      if (this.colorScalePick.type === "quantile") {
        let data = this.cartogramData.map(d => {
          return d[this.dataKey];
        });
        let quantile = d3
          .scaleQuantile()
          .domain(data) // pass the whole dataset to a scaleQuantile’s domain
          .range(this.colorScalePick.colorSchema);
        this.colorScale = quantile;
      } else {
        let colorDomain = d3.extent(this.cartogramData, d => d[this.dataKey]);
        // const colorPalette = this.colorPick;
        const colorPalette = this.colorScalePick.colorSchema;
        this.colorScale = d3
          .scaleSequential()
          .domain(colorDomain)
          .interpolator(d3.interpolate(colorPalette[0], colorPalette[1]))
          .nice();
      }
      // console.log(d3.interpolateViridis)
    },
    minHeight(d1, d2) {
      return Math.min(d1, d2);
    },
    hoverBox(event, d) {
      let thisBox = d3.select(".cartogram rect#" + d[this.gridKey]);
      // let mybox = document.getElementById(`${d.state}`).getBoundingClientRect();
      this.$emit("hoverOverBox", [d, thisBox]);
      // FIXME: reference for update function of tooltip position
      // let supportPageOffset = window.pageXOffset !== undefined;
      // this.mouse.x = event.screenX;
      // this.mouse.y = event.screenY;
      // this.element_bbox.left = mybox.left;
      // this.element_bbox.right = mybox.right;
      // this.element_bbox.top = mybox.top;
      // this.element_bbox.bottom = mybox.bottom;
      // this.element_bbox.width = mybox.width;
      // this.element_bbox.height = mybox.height;
      //  this.element_bbox.top=mybox.top+scrollY
      //  this.element_bbox.left=mybox.left+scrollX
      return [d, thisBox];
    },
    hoverOutBox() {
      this.hovered = null;
      this.$emit("hoverOutBox", this.hovered);
    },
    clickBox(d) {
      //if this box is not clicked yet
      if (!this.savedId[d[this.gridKey]]) {
        d3.select(".cartogram rect#" + d[this.gridKey]).classed(
          "cartogram-clicked",
          true
        );
        d3.select(".cartogram rect#" + d[this.gridKey]).raise();
        d3.select(".cartogram text#" + d[this.gridKey] + "text").raise();
        this.savedId[d[this.gridKey]] = true;
        this.selectedBox.push(d[this.gridKey]);
        // let thisBox = d3.select(".cartogram rect#" + d[this.gridKey]);
        // let mybox = document.getElementById(`${d.state}`).getBoundingClientRect();
      } else {
        d3.select(".cartogram rect#" + d[this.gridKey]).classed(
          "cartogram-clicked",
          false
        );
        this.savedId[d[this.gridKey]] = false;
        this.selectedBox.indexOf(d[this.gridKey]) > -1
          ? this.selectedBox.splice(
              this.selectedBox.indexOf(d[this.gridKey]),
              1
            )
          : false;
      }
      return;
    },
    retrieveCache(selected) {
      let validCache = _.pickBy(selected.selection, d => {
        return d;
      });
      let selectedArray = Object.keys(validCache);
      // setTimeout(() => {
      selectedArray.forEach(d => {
        d3.select(".cartogram rect#" + d).classed("cartogram-clicked", true);
        d3.select(".cartogram rect#" + d).raise();
        d3.select(".cartogram text#" + d + "text").raise();
        this.savedId[d] = true;
      });
      // }, 300);
      return;
    },
    clearSavedId() {
      this.savedId = {};
      return this.savedId;
    },
    switchShape(shape) {
      // this.chartView = shape;
      if (shape === "circle") this.shape = "circle";
      else if (shape === "rect") this.shape = "rect";
    }
  },
  beforeDestroy() {
    this.$emit("saveCache", { key: this.gridKey, selection: this.savedId });
    this.clearSavedId();
  }
};
</script>
<style scoped>
.cartogram-clicked {
  stroke: rgb(120, 218, 74);
  stroke-width: 3px;
}
.hovered {
  position: absolute;
  padding: 5px;
  transform: translate(-50%);
  pointer-events: none;
  /* background-color: #fff;
  box-shadow: 0 0 5px #cfcfcf;
  line-height: 1.6;
  width: 240px; */
}
</style>
<template>
  <div>
    <button type="button" @click="switchShape('circle')" id="btn-circle">
      Circle
    </button>
    <button type="button" @click="switchShape('rect')" id="btn-rect">
      Rectangle
    </button>
    <div v-if="shape === 'circle'">
      <svg
        class="cartogram"
        :width="canvas.width"
        :height="minHeight(canvas.height, canvas.width * 0.8)"
        :id="myid"
      >
        <g class="boxes">
          <!-- <div v-if="shape === 'circle'"> -->
          <circle
            class="cartogram-box"
            v-for="d in boxData"
            :key="d.id"
            :id="d[gridKey]"
            :r="d.width / 2.5"
            :cx="d.x + d.width / 2"
            :cy="d.y + d.width / 2"
            :fill="d.fill"
            :stroke="d.fill"
            @mouseover="hoverBox($event, d)"
            @click="clickBox(d)"
            @mouseenter="() => (hovered = d)"
            @mouseleave="hoverOutBox()"
          />
          <!--  -->
          <text
            style="pointer-events:none"
            class="rect-text"
            v-for="d in boxData"
            :key="d[gridKey]"
            :id="d[gridKey] + 'text'"
            :x="d.x + d.width / 2"
            :y="d.y + d.height / 2"
            :font-size="d.width / 3"
            dominant-baseline="middle"
            text-anchor="middle"
            fill="white"
          >
            {{ d[gridKey] }}
          </text>
        </g>
      </svg>
    </div>
    <div v-if="shape === 'rect'">
      <svg
        class="cartogram"
        :width="canvas.width"
        :height="minHeight(canvas.height, canvas.width * 0.8)"
        :id="myid"
      >
        <g class="boxes">
          <!-- <div v-if="shape === 'circle'"> -->
          <!-- </div> -->
          <rect
            v-for="d in boxData"
            :key="d.id"
            :id="d[gridKey]"
            class="cartogram-box"
            :width="d.width"
            :x="d.x"
            :height="d.height"
            :y="d.y"
            :fill="d.fill"
            :stroke="d.fill"
            @mouseover="hoverBox($event, d)"
            @click="clickBox(d)"
            @mouseenter="() => (hovered = d)"
            @mouseleave="hoverOutBox()"
          />
          <!--  -->
          <text
            style="pointer-events:none"
            class="rect-text"
            v-for="d in boxData"
            :key="d[gridKey]"
            :id="d[gridKey] + 'text'"
            :x="d.x + d.width / 2"
            :y="d.y + d.height / 2"
            :font-size="d.width / 3"
            dominant-baseline="middle"
            text-anchor="middle"
            fill="white"
          >
            {{ d[gridKey] }}
          </text>
        </g>
      </svg>
    </div>
    <slot name="tooltipTemplate"></slot>
    <!-- <div
      v-if="hovered && ifTooltip"
      class="hovered"
      :style="{
      top: `${element_bbox.top + element_bbox.height}px`,
      left: `${element_bbox.left}px`,
      }"
    >
      <slot name='tooltip-template'>
      </slot>
    </div>-->
  </div>
  <!-- :fill=d.fill :stroke='d.fill' -->
</template>
