<template>
  <div class="wrapper">
    <el-alert class="alert" type="success" :closable="false">
      <i class="el-icon-info" />
      import后注册到组件的components属性上按照以下方式调用即可
    </el-alert>
    <pre>
    &lt;date-ranger v-model="dateOption.value" v-bind="dateOption" /&gt;</pre
    >
    <h3 class="danger">基础使用：日期字符串初始化</h3>
    <date-ranger
      class="date-ranger"
      v-model="dateOption1.value"
      v-bind="dateOption1"
    />
    <pre>
      <span>const dateOption = {</span>
        <span>min: '2019-07-01',</span>
        <span>max: '2019-07-31',</span>
        <span>value: {</span>
          <span>min: '2019-07-01',</span>
          <span>max: '2019-07-10'</span>
        <span>}</span>
      <span>}</span></pre>
    <h3 class="danger">使用momentjs初始化</h3>
    <date-ranger v-model="dateOption2.value" v-bind="dateOption2" />
    <pre>
      <span>const dateOption = {</span>
        <span>min: moment('2019-07-01', "YYYY-MM-DD"),</span>
        <span>max: moment('2019-07-31', "YYYY-MM-DD"),</span>
        <span>value: {</span>
          <span>min: moment('2019-07-01', "YYYY-MM-DD"),</span>
          <span>max: moment('2019-07-10', "YYYY-MM-DD")</span>
        <span>}</span>
      <span>}</span></pre>
    <h3 class="danger">不带默认时间</h3>
    <div class="text-center">
      <el-button size="small" type="primary" @click="initializeDate">
        初始化时间
      </el-button>
    </div>
    <date-ranger
      ref="instance2"
      v-model="dateOption3.value"
      v-bind="dateOption3"
    />
    <pre>
      <span>const dateOption = {</span>
        <span>min: moment('2019-07-01', "YYYY-MM-DD"),</span>
        <span>max: moment('2019-07-31', "YYYY-MM-DD"),</span>
        <span>value: null</span>
      <span>}</span></pre>
    <el-alert class="alert" type="error" :closable="false">
      <i class="el-icon-warning" />
      改变值不能直接赋值给dateOption.value，需采用$ref给组件绑定DOM，调用内部setValue方法
    </el-alert>
    <pre>
      &lt;date-ranger ref="instance" v-model="dateOption.value" v-bind="dateOption" /&gt;

      <span>this.$refs.instance.setValue({</span>
        <span>min: '2019-07-01',</span>
        <span>max: '2019-07-10'</span>
      <span>})</span></pre>
    <h3 class="danger">最小区间限制</h3>
    <date-ranger v-model="dateOption4.value" v-bind="dateOption4" />
    <pre>
      <span>const dateOption = {</span>
        <span>min: moment('2019-07-01', "YYYY-MM-DD"),</span>
        <span>max: moment('2019-07-31', "YYYY-MM-DD"),</span>
        <span>value: {</span>
          <span>min: moment('2019-07-01', "YYYY-MM-DD"),</span>
          <span>max: moment('2019-07-10', "YYYY-MM-DD")</span>
        <span>},</span>
        <span>limit: {</span>
          <span>min: 3</span>
        <span>}</span>
      <span>}</span></pre>
    <h3 class="danger">最大区间限制</h3>
    <date-ranger v-model="dateOption5.value" v-bind="dateOption5" />
    <pre>
      <span>const dateOption = {</span>
        <span>min: moment('2019-07-01', "YYYY-MM-DD"),</span>
        <span>max: moment('2019-07-31', "YYYY-MM-DD"),</span>
        <span>value: {</span>
          <span>min: moment('2019-07-01', "YYYY-MM-DD"),</span>
          <span>max: moment('2019-07-10', "YYYY-MM-DD")</span>
        <span>},</span>
        <span>limit: {</span>
          <span>max: 10</span>
        <span>}</span>
      <span>}</span></pre>
    <h3 class="danger">只读</h3>
    <date-ranger v-model="dateOption6.value" v-bind="dateOption6" />
    <pre>
      <span>const dateOption = {</span>
        <span>min: moment('2019-07-01', "YYYY-MM-DD"),</span>
        <span>max: moment('2019-07-31', "YYYY-MM-DD"),</span>
        <span>value: {</span>
          <span>min: moment('2019-07-01', "YYYY-MM-DD"),</span>
          <span>max: moment('2019-07-10', "YYYY-MM-DD")</span>
        <span>},</span>
        <span>readonly: true</span>
      <span>}</span></pre>
    <h3 class="danger">显示时间范围边界：双击日期边界可进行更改</h3>
    <date-ranger v-model="dateOption7.value" v-bind="dateOption7" />
    <pre>
      <span>const dateOption = {</span>
        <span>min: moment('2019-07-01', "YYYY-MM-DD"),</span>
        <span>max: moment('2019-07-31', "YYYY-MM-DD"),</span>
        <span>value: {</span>
          <span>min: moment('2019-07-01', "YYYY-MM-DD"),</span>
          <span>max: moment('2019-07-10', "YYYY-MM-DD")</span>
        <span>},</span>
        <span>showRangeLabel: true</span>
      <span>}</span></pre>
    <h3 class="danger">可编辑时间：双击时分秒可进行更改</h3>
    <date-ranger v-model="dateOption8.value" v-bind="dateOption8" />
    <pre>
      <span>const dateOption = {</span>
        <span>min: moment('2019-07-01', "YYYY-MM-DD"),</span>
        <span>max: moment('2019-07-31', "YYYY-MM-DD"),</span>
        <span>value: {</span>
          <span>min: moment('2019-07-01', "YYYY-MM-DD"),</span>
          <span>max: moment('2019-07-10', "YYYY-MM-DD")</span>
        <span>},</span>
        <span>showTime: true</span>
      <span>}</span></pre>
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
  font-family: "Helvetica Neue",Helvetica,"PingFang SC","Hiragino Sans GB","Microsoft YaHei","微软雅黑",Arial,sans-serif;
}
h3 {
  text-align: left;
}
.alert {
  margin: 15px 0;
}
.alert >>> p {
  font-size: 14px;
  margin: 0;
}
.el-icon-info, .el-icon-warning {
  margin-right: 5px;
}
pre {
  border-radius: 4px;
  word-wrap: break-word;
  padding: 1em;
  background: #f7f7f8;
  margin: 0 0 30px;
  direction: ltr;
  text-align: left;
  font-family: Menlo,Monaco,Consolas,Courier,monospace;
}
pre > span {
  line-height: 22px;
}
.text-center {
  text-align: center;
}
.danger {
  position: relative;
  color: #f56c6c;
  padding-left: 12px;
}
.danger:before {
  content: "";
  display: block;
  width: 5px;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
  background-color: #f56c6c;
}
</style>
