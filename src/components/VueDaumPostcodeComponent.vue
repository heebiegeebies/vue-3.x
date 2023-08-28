<template>
  <VueDaumPostcode @complete="onComplete">
    <template #loading>
      <div>...Loading...</div>
    </template>
  </VueDaumPostcode>
</template>

<script>
import {closeModal} from "jenesius-vue-modal";

export default {
  data() {
    return {
      address: "",
      detAddr: "",
      zoneCode: "",
    };
  },
  methods: {
    onComplete(newResult) {
      this.address = newResult.address;
      this.zoneCode = newResult.zonecode;

      closeModal();
    },
  },
};
</script>
