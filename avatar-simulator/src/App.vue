<template>
  <div id="app">
    <AvatarSimulator2D />
  </div>
</template>

<script>
import AvatarSimulator from "./components/AvatarSimulator.vue";
import AvatarSimulator2D from "./components/icons/AvatarSimulator2D.vue";

export default {
  name: "App",
  components: {
    AvatarSimulator2D,
  },
};
</script>

<style>
body {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
}
</style>
