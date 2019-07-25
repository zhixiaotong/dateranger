<template>
  <div class="wrapper">
    <h3>基础使用</h3>
    <date-ranger
      class="date-ranger"
      v-model="dateOption1.value"
      v-bind="dateOption1"
    />
    <h3>使用momentjs初始化</h3>
    <date-ranger
      class="date-ranger"
      v-model="dateOption2.value"
      v-bind="dateOption2"
    />
    <h3>不带默认时间</h3>
    <el-button size="small" type="primary" @click="initializeDate"
      >初始化时间</el-button
    >
    <date-ranger
      ref="instance2"
      class="date-ranger"
      v-model="dateOption3.value"
      v-bind="dateOption3"
    />
    <h3>最小区间限制</h3>
    <date-ranger
      class="date-ranger"
      v-model="dateOption4.value"
      v-bind="dateOption4"
    />
    <h3>最大区间限制</h3>
    <date-ranger
      class="date-ranger"
      v-model="dateOption5.value"
      v-bind="dateOption5"
    />
    <h3>只读</h3>
    <date-ranger
      class="date-ranger"
      v-model="dateOption6.value"
      v-bind="dateOption6"
    />
    <h3>显示时间范围边界</h3>
    <date-ranger
      class="date-ranger"
      v-model="dateOption7.value"
      v-bind="dateOption7"
    />
    <h3>可编辑时间</h3>
    <date-ranger
      class="date-ranger"
      v-model="dateOption8.value"
      v-bind="dateOption8"
    />
  </div>
</template>
<script>
import DateRanger from "@/components/DateRanger";
import moment from "moment";
/* eslint-disable */
const dateOption1 = {
  min: '2019-07-01',
  max: '2019-07-31',
  value: {
    min: '2019-07-01',
    max: '2019-07-10'
  }
};
const dateOption2 = {
  min: moment('2019-07-01', "YYYY-MM-DD"),
  max: moment('2019-07-31', "YYYY-MM-DD"),
  value: {
    min: moment('2019-07-01', "YYYY-MM-DD"),
    max: moment('2019-07-10', "YYYY-MM-DD")
  }
};
const dateOption3 = {
  min: moment('2019-07-01', "YYYY-MM-DD"),
  max: moment('2019-07-31', "YYYY-MM-DD"),
  value: null
};
const dateOption4 = {
  min: moment('2019-07-01', "YYYY-MM-DD"),
  max: moment('2019-07-31', "YYYY-MM-DD"),
  value: {
    min: moment('2019-07-01', "YYYY-MM-DD"),
    max: moment('2019-07-10', "YYYY-MM-DD")
  },
  limit: {
    min: 3
  }
};
const dateOption5 = {
  min: moment('2019-07-01', "YYYY-MM-DD"),
  max: moment('2019-07-31', "YYYY-MM-DD"),
  value: {
    min: moment('2019-07-01', "YYYY-MM-DD"),
    max: moment('2019-07-10', "YYYY-MM-DD")
  },
  limit: {
    max: 10
  }
};
const dateOption6 = {
  min: moment('2019-07-01', "YYYY-MM-DD"),
  max: moment('2019-07-31', "YYYY-MM-DD"),
  value: {
    min: moment('2019-07-01', "YYYY-MM-DD"),
    max: moment('2019-07-10', "YYYY-MM-DD")
  },
  readonly: true
};
const dateOption7 = {
  min: moment('2019-07-01', "YYYY-MM-DD"),
  max: moment('2019-07-31', "YYYY-MM-DD"),
  value: {
    min: moment('2019-07-01', "YYYY-MM-DD"),
    max: moment('2019-07-10', "YYYY-MM-DD")
  },
  showRangeLabel: true
};
const dateOption8 = {
  min: moment('2019-07-01', "YYYY-MM-DD"),
  max: moment('2019-07-31', "YYYY-MM-DD"),
  value: {
    min: moment('2019-07-01', "YYYY-MM-DD"),
    max: moment('2019-07-10', "YYYY-MM-DD")
  },
  showTime: true
};
export default {
  components: {
    DateRanger
  },
  data() {
    return {
      dateOption1,
      dateOption2,
      dateOption3,
      dateOption4,
      dateOption5,
      dateOption6,
      dateOption7,
      dateOption8
    };
  },
  methods: {
    initializeDate() {
      this.$refs.instance2.setValue({
        min: '2019-07-01',
        max: '2019-07-10'
      })
    }
  }
};
</script>
<style scoped>
.wrapper {
  padding: 0 120px;
}
h3 {
  text-align: left;
}
.date-ranger {
  margin-bottom: 20px;
}
</style>
