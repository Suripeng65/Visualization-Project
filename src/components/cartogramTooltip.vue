<script>
export default {
  name: "TooltipTemplate",
  props: {
    tooltipData: {
      type: Object,
      default: function() {
        return {};
      }
    },
    dataKey: {
      type: String,
      default: "score_st"
    },
    gridKey: {
      type: String,
      default: "state"
    }
  }
};
</script>
<style scoped>
.cartogram-tooltip-container {
  min-width: 400px;
  background-color: white;
  border: 1px solid #ccc;
  z-index: 1000001;
  box-shadow: 2px 2px 6px rgba(0, 0, 0, 0.5);
  padding-bottom: 4px;
}
.cartogram-tooltip-container .cartogram-tooltip-table {
  min-width: 400px;
  background-color: white;
}

.cartogram-tooltip-table
  .cartogram-tooltip-header-row
  .cartogram-tooltip-header {
  text-align: center;
  font: 400 11px;
  text-transform: capitalize;
  border-bottom: 1px solid #e2e2e2;
  vertical-align: middle;
  color: #999;
  font-size: 11px;
  /* padding-bottom: 4px; */
  min-width: 60px;
}
.cartogram-tooltip-table .cartogram-tooltip-row .cartogram-tooltip-cell {
  font-size: 12px;
  min-width: 60px;
  height: auto;
  padding: 4px 5px 4px 5px;
  text-align: center;
  vertical-align: middle;
}
</style>
<template>
  <div class="cartogram-tooltip-container">
    <table class="cartogram-tooltip-table">
      <thead>
        <tr class="cartogram-tooltip-header-row">
          <td class="cartogram-tooltip-header">District</td>
          <td class="cartogram-tooltip-header"># in District</td>
          <td class="cartogram-tooltip-header">% in District</td>
          <td class="cartogram-tooltip-header"># in State</td>
        </tr>
      </thead>
      <tbody>
        <tr class="cartogram-tooltip-row">
          <td class="cartogram-tooltip-cell">{{ tooltipData[gridKey] }}</td>
          <td class="cartogram-tooltip-cell">{{ tooltipData["score_jd"] }}</td>
          <td class="cartogram-tooltip-cell">{{ tooltipData["score_st"] }}</td>
          <td class="cartogram-tooltip-cell">
            {{ tooltipData["score_jd"] + tooltipData["score_st"] }}
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>
