<template>
  <Test @complete="callback"></Test>
  <div>
    <p>{{ addr }}</p>
  </div>
</template>
<script>
import VueDaumPostcodeComponent from "../components/VueDaumPostcodeComponent.vue";
export default {
  data() {
    return {
      addr: {},
    };
  },
  components: {Test: VueDaumPostcodeComponent},
  methods: {
    callback(res) {
      console.log(`emitted data!`);
      console.log(res);
      this.addr = res;
    },
  },
};
</script>
