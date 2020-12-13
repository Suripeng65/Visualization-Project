<script>
// @ is an alias to /src
import CartogramWrapper from "@/components/cartogramWrapper.vue";
import TooltipTemplate from "@/components/TooltipTemplate.vue";
// import ColorPalette from "@/components/colorPalette.vue";
import cartogram_grid from "../config/cartogram_grid.js";
import cartogram_data from "../config/cartogram_data.js";
import CartogramTooltip from "@/components/cartogramTooltip.vue";
// import * as d3 from "d3";

export default {
  name: "CartogramPage",
  components: {
    CartogramWrapper,
    // ColorPalette,
    TooltipTemplate,
    CartogramTooltip
  },
  data() {
    return {
      myId: "myCartogramId",
      hoveredData: null,
      cartogram_grid: cartogram_grid,
      cartogramData: cartogram_data,
      hoveredElement: null,
      hovered: false,
      chartView: "state",
      dataKey: "score_st",
      savedCache: { state: {}, jd: {} }
    };
  },
  computed: {
    gridArrayData() {
      return cartogram_grid[this.chartView];
    }
  },
  mount() {},
  methods: {
    hoverOverCartogramBox(e) {
      this.hovered = true;
      this.hoveredData = e[0];
      this.hoveredElement = e[1];
      //receive hover over selection, data,
      //can be used for other components that required interactions with map
    },
    switchView(view) {
      this.chartView = view;
      if (view === "jd") this.dataKey = "score_jd";
      else if (view === "state") this.dataKey = "score_st";
    },
    setCache(data) {
      this.savedCache[data.key] = data;
    }
  }
};
</script>
<style scoped>
/* .tooltip{
    background-color:white;
    border: solid;
    border-width: 2px;
    border-radius:5px;
    padding:5px;
    opacity:0
  } */
</style>
<template>
  <div id="cartogram-area">
    <!-- <Cartogram></Cartogram> -->
    <button type="button" @click="switchView('state')" id="btn-st">
      State
    </button>
    <button type="button" @click="switchView('jd')" id="btn-jd">JD</button>
    <CartogramWrapper
      v-if="chartView === 'state'"
      key="state"
      :id="myId"
      :gridKey="chartView"
      :dataKey="dataKey"
      :gridArray="gridArrayData"
      :cartogramData="cartogramData"
      :regionCache="savedCache"
      :canvas="{ width: 500, height: 400 }"
      @hoverOverBox="(...args) => this.hoverOverCartogramBox(...args)"
      @hoverOutBox="() => (hovered = false)"
      @saveCache="(...args) => this.setCache(...args)"
    >
      <TooltipTemplate
        v-if="hoveredData"
        slot="tooltipTemplate"
        :hovered="hovered"
        :tooltipSelection="hoveredElement"
        :tooltipData="hoveredData"
        :ifTooltip="true"
      >
        <CartogramTooltip
          slot="customTooltip"
          :tooltipData="hoveredData"
          :gridKey="chartView"
          :dataKey="dataKey"
        ></CartogramTooltip>
      </TooltipTemplate>
    </CartogramWrapper>
    <CartogramWrapper
      key="jd"
      v-if="chartView === 'jd'"
      :id="myId"
      :gridKey="chartView"
      :dataKey="dataKey"
      :gridArray="gridArrayData"
      :cartogramData="cartogramData"
      :regionCache="savedCache"
      :canvas="{ width: 800, height: 400 }"
      @hoverOverBox="(...args) => this.hoverOverCartogramBox(...args)"
      @hoverOutBox="() => (hovered = false)"
      @saveCache="(...args) => this.setCache(...args)"
    >
      <TooltipTemplate
        v-if="hoveredData"
        slot="tooltipTemplate"
        :hovered="hovered"
        :tooltipSelection="hoveredElement"
        :tooltipData="hoveredData"
        :ifTooltip="true"
      >
        <CartogramTooltip
          slot="customTooltip"
          :tooltipData="hoveredData"
          :gridKey="chartView"
          :dataKey="dataKey"
        ></CartogramTooltip>
      </TooltipTemplate>
    </CartogramWrapper>
    <ColorPalette
      v-if="chartView === 'jd'"
      :rawData="{ key: dataKey, value: cartogramData }"
    ></ColorPalette>
    <ColorPalette
      v-if="chartView === 'state'"
      :rawData="{ key: dataKey, value: cartogramData }"
    ></ColorPalette>
  </div>
</template>
