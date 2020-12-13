<template>
  <div class="home">
    <ComponentLib msg="Visualization Component in Vue" />
  </div>
</template>

<script>
// @ is an alias to /src
import ComponentLib from "@/components/ComponentLib.vue";

export default {
  name: "Home",
  components: {
    ComponentLib
  }
};
</script>
