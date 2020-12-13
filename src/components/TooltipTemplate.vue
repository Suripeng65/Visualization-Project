<script lang="ts">
// import Vue from "vue";
export default {
  name: "TooltipTemplate",
  props: {
    tooltipData: {
      type: Object,
      default: function() {
        return {};
      }
    },
    tooltipSelection: {
      type: Object,
      default: function() {
        return {};
      }
    },
    hovered: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      element_bbox: {}
    };
  },
  computed: {
    ifShowTooltip() {
      let ifShowTooltip = false;
      if (
        this.hovered &&
        this.tooltipSelection &&
        this.tooltipData
        // this.tooltipSelection.nodes()[0] //handle edge case for missing data
      )
        ifShowTooltip = true;
      else ifShowTooltip = false;
      return ifShowTooltip;
    }
  },
  mounted() {},
  watch: {
    tooltipSelection: {
      deep: true,
      handler() {
        this.calculatePosition();
      }
    }
  },
  methods: {
    calculatePosition() {
      let rectDom = this.tooltipSelection.nodes()[0];
      let mybox;
      if (rectDom) {
        //handle edge case for missing selection(data)
        mybox = rectDom.getBoundingClientRect();
        this.element_bbox.left = mybox.left;
        this.element_bbox.right = mybox.right;
        this.element_bbox.top = mybox.top;
        this.element_bbox.bottom = mybox.bottom;
        this.element_bbox.width = mybox.width;
        this.element_bbox.height = mybox.height;
        return this.element_bbox;
      } else return;
    }
  }
};
</script>
<style scoped>
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
  <div
    v-if="ifShowTooltip"
    class="hovered"
    :style="{
      top: `${element_bbox.top + element_bbox.height}px`,
      left: `${element_bbox.left}px`
    }"
  >
    <slot name="customTooltip"></slot>
  </div>
</template>
