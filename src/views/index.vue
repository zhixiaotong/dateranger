<template>
  <div id="app">
    <date-ranger
      ref="dateRanger"
      v-model="dateOption.value"
      class="date-ranger"
      v-bind="dateOption"
    />
  </div>
</template>
<script>
import DateRanger from "@/components/DateRanger";
import moment from "moment";
/* eslint-disable */
const dateOption = {
  min: moment().subtract(1, "months").format("YYYY-MM-DD"),
  max: moment().format("YYYY-MM-DD"),
  value: {
    min: moment().subtract(10, "days").format("YYYY-MM-DD"),
    max: moment().format("YYYY-MM-DD")
  },
  limit: {
    // 最小限制区间：一天
    min: 1
  }
};
export default {
  components: {
    DateRanger
  },
  data() {
    return {
      dateOption
    };
  }
};
</script>
