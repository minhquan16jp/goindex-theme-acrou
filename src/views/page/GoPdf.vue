<template>
  <div></div>
</template>

<script>
import { decode64 } from "@utils/AcrouUtil";

export default {
  mounted() {
    if (this.$route.params.path) {
      window.location.href = decode64(this.$route.params.path);
    }
  }
};
</script>