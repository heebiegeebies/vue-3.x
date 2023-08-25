<template>
  <VueDaumPostcode @complete="$emit('onComplete')">
    <template #loading>
      <div>...Loading...</div>
    </template>
  </VueDaumPostcode>
</template>

<script>
export default {
  data() {
    return {
      addr: "",
      detAddr: "",
    };
  },
  methods: {
    onComplete(newResult) {
      console.log("result is being emitted");

      this.daumPostcode.result = newResult;
      this.daumPostcode.isOpen = false;
      this.formData.zip_code = newResult.zonecode;
      this.formData.addr = newResult.addr;
      //   return newResult;
    },
  },
};
</script>
