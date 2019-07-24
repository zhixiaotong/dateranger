<template>
  <div class="uni-date-ranger" :class="skin">
    <div class="range-picker-wrapper" :style="wrapperStyle">
      <div class="range-picker">
        <span class="not-select-process" />
        <span
          v-if="showRangeLabel"
          class="label range-label"
          @dblclick="editMin"
        >
          <template v-if="!edit.min">{{ minValueFmt }}</template>
          <el-input
            v-else
            v-model="edit.minValue"
            v-focus
            el-else
            class="range-picker__input"
            type="text"
            @blur="blurEditMin"
          />
        </span>
        <span
          v-if="showRangeLabel"
          class="label range-label end-label"
          @dblclick="editMax"
        >
          <template v-if="!edit.max">{{ maxValueFmt }}</template>
          <el-input
            v-else
            v-model="edit.maxValue"
            v-focus
            class="range-picker__input"
            type="text"
            @blur="blurEditMax"
          />
        </span>
        <span
          class="process"
          :class="{ disabled: readonly }"
          :style="processStyle"
        />
        <span
          class="label select-label"
          :class="{ active: isBtn === 1, disabled: readonly }"
          :style="{ left: `${label1Left}px` }"
          @dblclick="editLabel1"
        >
          <template v-if="!edit.label1">{{ label1Fmt }}</template>
          <template v-else>
            {{ label1Fmt.split(" ")[0] }}
            <el-time-picker
              v-model="edit.label1Time"
              class="time-input"
              :clearable="false"
              value-format="HH:mm:ss"
              :default-value="label1Fmt.split(' ')[0]"
              @blur="edit.label1 = false"
              @change="changeLabel1"
            />
          </template>
        </span>
        <span
          class="label select-label select-label-2"
          :class="{ active: isBtn === 2, disabled: readonly }"
          :style="{ left: `${label2Left}px` }"
          @dblclick="editLabel2"
        >
          <template v-if="!edit.label2">{{ label2Fmt }}</template>
          <template v-else>
            {{ label2Fmt.split(" ")[0] }}
            <el-time-picker
              v-model="edit.label2Time"
              class="time-input"
              :clearable="false"
              value-format="HH:mm:ss"
              :default-value="label2Fmt.split(' ')[0]"
              @blur="edit.label2 = false"
              @change="changeLabel2"
            />
          </template>
        </span>
      </div>
    </div>
    <el-dropdown
      v-if="shortcut.show"
      class="shortcut-dropdown"
      @command="handleCommand"
    >
      <el-button size="mini" type="primary" class="padding-5"
        ><span class="shortcut-desc">{{ activeShortcut }}</span
        ><i class="el-icon-arrow-up margin-left-5"
      /></el-button>
      <el-dropdown-menu slot="dropdown" class="padding-5">
        <el-dropdown-item
          v-for="(opt, idx) of shortcut.options"
          v-cloak
          :key="idx"
          class="dropdown-item"
          :class="{ active: activeShortcut === opt.name }"
          :command="idx"
          >{{ opt.name }}</el-dropdown-item
        >
      </el-dropdown-menu>
    </el-dropdown>
  </div>
</template>

<script>
import moment from "moment";
import { uuid, isString } from "@/utils";
import { addResizeListener, removeResizeListener } from "@/utils/resize-event";
export default {
  directives: {
    focus: {
      inserted: el => el.focus()
    }
  },
  props: {
    id: {
      type: String,
      default: uuid()
    },
    value: {
      type: Object,
      default: null
    },
    /** value值限制(天) */
    limit: {
      type: Object,
      default: null
    },
    /** min值和max值区间的限制(天)：默认0, 表示没有限制 */
    rangeLimit: {
      type: Number,
      default: 0
    },
    showTime: {
      type: Boolean,
      default: false
    },
    /** 是否展示时间范围边界值 */
    showRangeLabel: {
      type: Boolean,
      default: false
    },
    /** 是否只读 */
    readonly: {
      type: Boolean,
      default: false
    },
    min: [String, Object],
    max: [String, Object],
    /** 快捷方式 */
    shortcut: {
      type: Object,
      default: () => {
        return {
          show: false,
          options: void 0
        };
      }
    },
    /** 皮肤 */
    skin: {
      type: String,
      default: "default",
      validator: val => ["default", "dark"].indexOf(val) > -1
    }
  },
  data() {
    return {
      minValue: this.min,
      maxValue: this.max,
      label1: void 0,
      label2: void 0,
      label1Left: void 0,
      label2Left: void 0,
      isBtn: void 0,
      selectLabelWidth: void 0,
      rangeWidth: void 0,
      deviation: void 0,
      dragBtn: false,
      dragProcess: false,
      defaultValue: this.value,
      wrapperStyle: null,
      formatYMD: "YYYY-MM-DD",
      formatYMDHms: "YYYY-MM-DD HH:mm:ss",
      edit: {
        enable: true,
        minValue: "",
        maxValue: "",
        min: false,
        max: false,
        label1: false,
        label2: false,
        label1Time: "",
        label2Time: ""
      },
      activeShortcut:
        this.shortcut.show && this.shortcut.options.length > 0
          ? this.shortcut.options[0].name
          : void 0
    };
  },
  computed: {
    period() {
      return moment(this.maxValue).diff(this.minValue, "days");
    },
    processStyle() {
      if (this.rangeWidth !== undefined) {
        let _left =
          Math.min(this.label1Left, this.label2Left) +
          this.selectLabelWidth / 2;
        let _right =
          this.rangeWidth -
          Math.max(this.label1Left, this.label2Left) -
          this.selectLabelWidth / 2;
        if (_left < 0) _left = 0;
        if (_right < 0) _right = 0;
        return {
          left: `${_left}px`,
          right: `${_right}px`
        };
      }
      return null;
    },
    minValueFmt() {
      if (
        this.minValue === undefined ||
        !this.minValue ||
        !moment.isMoment(this.minValue)
      )
        return "";
      return this.minValue.format(this.formatYMD);
    },
    maxValueFmt() {
      if (
        this.maxValue === undefined ||
        !this.maxValue ||
        !moment.isMoment(this.minValue)
      )
        return "";
      return this.maxValue.format(this.formatYMD);
    },
    label1Fmt() {
      if (
        this.label1 === undefined ||
        !this.label1 ||
        !moment.isMoment(this.label1)
      )
        return "";
      let fmtStr = this.label1.format(this.formatYMD);
      if (this.showTime) {
        fmtStr += " " + this.edit.label1Time;
      }
      return fmtStr;
    },
    label2Fmt() {
      if (
        this.label2 === undefined ||
        !this.label2 ||
        !moment.isMoment(this.label2)
      )
        return "";
      let fmtStr = this.label2.format(this.formatYMD);
      if (this.showTime) {
        fmtStr += " " + this.edit.label2Time;
      }
      return fmtStr;
    }
  },
  watch: {
    value(val) {
      const tempVal = {};
      if (
        !moment.isMoment(val.min) &&
        isString(val.min) &&
        moment(val.min, this.formatYMD).isValid()
      ) {
        tempVal.min = moment(val.min, this.formatYMD);
      }
      if (
        !moment.isMoment(val.max) &&
        isString(val.max) &&
        moment(val.max, this.formatYMD).isValid()
      ) {
        tempVal.max = moment(val.max, this.formatYMD);
      }
      this.defaultValue = tempVal;
    },
    label1(val) {
      this.label1Left =
        (moment(val).diff(this.minValue, "days") / this.period) *
          this.rangeWidth -
        this.selectLabelWidth / 2;
      if (this.showTime) {
        const time = this.edit.label1Time.split(":");
        val.hour(time[0]);
        val.minute(time[1]);
        val.second(time[2]);
      }
      this.defaultValue = {
        min: moment.min(val, this.label2),
        max: moment.max(val, this.label2)
      };
      this.emitValue();
    },
    label2(val) {
      this.label2Left =
        (moment(val).diff(this.minValue, "days") / this.period) *
          this.rangeWidth -
        this.selectLabelWidth / 2;
      if (this.showTime) {
        const time = this.edit.label2Time.split(":");
        val.hour(time[0]);
        val.minute(time[1]);
        val.second(time[2]);
      }
      this.defaultValue = {
        min: moment.min(this.label1, val),
        max: moment.max(this.label1, val)
      };
      this.emitValue();
    }
  },
  mounted() {
    this.$nextTick(() => {
      // 校验语法格式
      this.checkSyntax();
      // 初始化默认边界值
      if (this.defaultValue == null) {
        this.label1 = this.label2 = this.minValue;
      } else {
        if (
          !moment.isMoment(this.defaultValue.min) &&
          isString(this.defaultValue.min) &&
          moment(this.defaultValue.min, this.formatYMD).isValid()
        ) {
          this.label1 = moment(this.defaultValue.min, this.formatYMD);
        }
        if (
          !moment.isMoment(this.defaultValue.max) &&
          isString(this.defaultValue.max) &&
          moment(this.defaultValue.max, this.formatYMD).isValid()
        ) {
          this.label2 = moment(this.defaultValue.max, this.formatYMD);
        }
        // 校验数据值
        if (this.limit !== null && this.limit.min && this.limit.max) {
          const differ = Math.abs(
            moment(this.label2).diff(this.label1, "days")
          );
          if (differ > this.limit.max || differ < this.limit.min) {
            throw new Error("默认时间区间不在limit限制值区间内");
          }
        }
      }
      if (this.showTime) {
        this.edit.label1Time = "00:00:00";
        this.edit.label2Time = "23:59:59";
      }
      // 初始化容器宽度
      this.initialDomSize();
      // 注册事件
      this.$el.addEventListener("mousedown", e => this.initMousedownEvent(e));
      this.$el.addEventListener("mousemove", e => this.initMousemoveEvent(e));
      this.$el.addEventListener("mouseup", e => this.initMouseupEvent(e));
      document.body.addEventListener("mouseup", e => this.initMouseupEvent(e));
      addResizeListener(this.$el, this.handleResize);
    });
  },
  beforeDestroy() {
    this.$el.removeEventListener("mousedown", e => this.initMousedownEvent(e));
    this.$el.removeEventListener("mousemove", e => this.initMousemoveEvent(e));
    this.$el.removeEventListener("mouseup", e => this.initMouseupEvent(e));
    document.body.removeEventListener("mouseup", e => this.initMouseupEvent(e));
    if (this.$el && this.handleResize) {
      removeResizeListener(this.$el, this.handleResize);
    }
  },
  methods: {
    checkSyntax() {
      // 校验边界min和max
      if (
        isString(this.minValue) &&
        moment(this.minValue, this.formatYMD).isValid()
      ) {
        this.minValue = moment(this.minValue, this.formatYMD);
      } else {
        if (moment(this.minValue, this.formatYMD).isValid())
          throw new SyntaxError(
            "min值必须是YYYY-MM-DD的日期字符串或moment对象"
          );
      }
      if (
        !moment.isMoment(this.maxValue) &&
        isString(this.maxValue) &&
        moment(this.maxValue, this.formatYMD)
      ) {
        this.maxValue = moment(this.maxValue, this.formatYMD);
      } else {
        if (moment(this.maxValue, this.formatYMD).isValid())
          throw new SyntaxError(
            "max值必须是YYYY-MM-DD的日期字符串或moment对象"
          );
      }
      if (this.minValue == null || this.maxValue == null) {
        throw new Error("min和max属性必传");
      }
      if (moment().isBefore(this.maxValue)) {
        throw new Error("max值max不能超过当前最新时间");
      }
      if (this.shortcut.show && this.shortcut.options.length <= 0) {
        throw new Error("快捷时间段显示时，须设置内容");
      }
    },
    setValue({ min, max }) {
      if (moment.isMoment(min)) {
        this.label1 = min;
      } else if (
        !moment.isMoment(min) &&
        isString(min) &&
        moment(min, this.formatYMD).isValid()
      ) {
        this.label1 = moment(min, this.formatYMD);
      }
      if (moment.isMoment(max)) {
        this.label2 = max;
      } else if (
        !moment.isMoment(max) &&
        isString(max) &&
        moment(max, this.formatYMD).isValid()
      ) {
        this.label2 = moment(max, this.formatYMD);
      }
    },
    emitValue() {
      let fmtStr = this.formatYMD;
      if (this.showTime) {
        fmtStr = this.formatYMDHms;
      }
      this.$emit("input", {
        min: moment(this.defaultValue.min)
          .utcOffset(8)
          .format(fmtStr),
        max: moment(this.defaultValue.max)
          .utcOffset(8)
          .format(fmtStr)
      });
    },
    initialDomSize() {
      this.selectLabelWidth = this.$el.querySelector(
        ".select-label"
      ).offsetWidth;
      this.rangeWidth = this.$el.offsetWidth - this.selectLabelWidth;
      this.wrapperStyle = {
        "padding-left": `${this.selectLabelWidth / 2}px`,
        "padding-right": `${this.selectLabelWidth / 2}px`
      };
    },
    elementLeft(e) {
      let offset = e.offsetLeft;
      if (e.offsetParent !== null) {
        offset += this.elementLeft(e.offsetParent);
      }
      return offset;
    },
    editMin() {
      if (this.readonly || this.edit.min) return;
      this.edit.min = true;
      this.edit.minValue = this.minValueFmt;
    },
    editMax() {
      if (this.readonly || this.edit.max) return;
      this.edit.max = true;
      this.edit.maxValue = this.maxValueFmt;
    },
    editLabel1() {
      if (this.readonly || !this.showTime || this.edit.label1) return;
      this.edit.label1 = true;
      this.edit.label1Time = this.label1Fmt.split(" ")[1];
      if (this.edit.label2) this.edit.label2 = false;
    },
    editLabel2() {
      if (this.readonly || !this.showTime || this.edit.label2) return;
      this.edit.label2 = true;
      this.edit.label2Time = this.label2Fmt.split(" ")[1];
      if (this.edit.label1) this.edit.label1 = false;
    },
    changeLabel1() {
      this.edit.label1 = false;
      const time = this.edit.label1Time.split(":");
      this.label1.hour(time[0]);
      this.label1.minute(time[1]);
      this.label1.second(time[2]);
      this.defaultValue = {
        min: moment.min(this.label1, this.label2),
        max: moment.max(this.label1, this.label2)
      };
      this.emitValue();
    },
    changeLabel2() {
      this.edit.label2 = false;
      const time = this.edit.label2Time.split(":");
      this.label2.hour(time[0]);
      this.label2.minute(time[1]);
      this.label2.second(time[2]);
      this.defaultValue = {
        min: moment.min(this.label1, this.label2),
        max: moment.max(this.label1, this.label2)
      };
      this.emitValue();
    },
    blurEditMin() {
      this.edit.min = false;
      const tempMin = moment(this.edit.minValue, this.formatYMD);
      if (tempMin.isSame(this.minValue)) return;
      if (!tempMin.isValid()) {
        this.$message.warning("不合法的日期");
        return;
      }
      if (this.rangeLimit > 0) {
        const tempMax = moment(this.edit.minValue, this.formatYMD).add(
          this.rangeLimit,
          "days"
        );
        if (moment().isBefore(tempMax)) {
          this.$message.warning(
            `日期选择区间限定max日期与min日期相差${
              this.rangeLimit
            }天, 该日期设置导致max日期已超过当天期, 请重新设置`
          );
          return;
        }
        this.maxValue = tempMax;
      } else if (tempMin.isAfter(this.maxValue)) {
        this.$message.warning("min日期范围应小于max日期");
        return;
      }
      this.minValue = tempMin;
      // 给定默认值
      this.label1 = this.label2 = this.minValue;
      this.$emit("change-bound", this.id, {
        min: moment(this.minValue).format(this.formatYMD),
        max: moment(this.maxValue).format(this.formatYMD)
      });
    },
    blurEditMax() {
      this.edit.max = false;
      const temp = moment(this.edit.maxValue, this.formatYMD);
      if (temp.isSame(this.maxValue)) return;
      if (!temp.isValid()) {
        this.$message.warning("不合法的日期");
        return;
      }
      if (moment().isBefore(temp)) {
        this.$message.warning("max日期范围不能超过当天日期");
        return;
      }
      if (this.rangeLimit > 0) {
        this.minValue = moment(this.edit.maxValue, this.formatYMD).subtract(
          this.rangeLimit,
          "days"
        );
      } else if (temp.isBefore(this.minValue)) {
        this.$message.warning("min日期范围应小于max日期");
        return;
      }
      this.maxValue = temp;
      // 给定默认值
      this.label1 = this.label2 = this.minValue;
      this.$emit("change-bound", this.id, {
        min: moment(this.minValue).format(this.formatYMD),
        max: moment(this.maxValue).format(this.formatYMD)
      });
    },
    getTempWidth(e) {
      let tempWidth;
      const x =
        e.pageX - this.elementLeft(this.$el) - this.selectLabelWidth / 2;
      if (x + this.selectLabelWidth / 2 - this.deviation > this.rangeWidth) {
        tempWidth = this.rangeWidth - this.selectLabelWidth / 2;
      } else if (x + this.selectLabelWidth / 2 - this.deviation < 0) {
        tempWidth = -this.selectLabelWidth / 2;
      } else {
        tempWidth = x - this.deviation;
      }
      return tempWidth;
    },
    initMousedownEvent(e) {
      if (this.readonly) return;
      const target = e.target || e.toElement;
      if (target !== null) {
        // 选中单个label
        if (target.classList.contains("select-label")) {
          this.dragBtn = true;
          // 记录鼠标相对于选中dom开头箭头位置的偏差
          this.deviation = e.pageX - this.elementLeft(target);
          if (target.classList.contains("select-label-2")) {
            this.isBtn = 2;
          } else {
            this.isBtn = 1;
          }
          return;
        }
        // 选中process进度
        if (target.classList.contains("process")) {
          this.dragProcess = true;
          // 记录鼠标相对于选中dom开头箭头位置的偏差
          this.deviation = e.pageX - this.elementLeft(target);
        }
      }
    },
    initMousemoveEvent(e) {
      if (this.readonly) return;
      if (e.buttons === 1) {
        // 拖动的是label
        if (this.dragBtn) {
          // 计算当前宽度值
          const tempWidth = this.getTempWidth(e);
          // 根据比例尺获取当前时间
          const current = moment(this.minValue).add(
            parseInt(
              (this.period / this.rangeWidth) *
                (tempWidth + this.selectLabelWidth / 2)
            ),
            "days"
          );
          if (this.isBtn === 1) {
            this.label1 = current;
          } else if (this.isBtn === 2) {
            this.label2 = current;
          }
          return;
        }
        // 拖动的是process块
        if (this.dragProcess) {
          // 计算当前宽度值
          const tempWidth =
            e.pageX - this.elementLeft(this.$el) - this.deviation;
          const current = moment(this.minValue).add(
            parseInt(
              (this.period / this.rangeWidth) *
                (tempWidth - this.selectLabelWidth / 2)
            ),
            "days"
          );
          const duration = Math.abs(
            moment(this.label1).diff(this.label2, "days")
          );
          // 更新当前时间
          let tmp1 = current;
          let tmp2 = moment(current).add(duration, "days");
          if (current.isBefore(this.minValue)) {
            tmp1 = this.minValue;
            tmp2 = moment(this.minValue).add(duration, "days");
          } else if (tmp2.isAfter(this.maxValue)) {
            tmp1 = moment(this.maxValue).subtract(duration, "days");
            tmp2 = this.maxValue;
          }
          if (this.label2.isBefore(this.label1)) {
            this.label1 = tmp2;
            this.label2 = tmp1;
          } else {
            this.label1 = tmp1;
            this.label2 = tmp2;
          }
        }
      }
    },
    initMouseupEvent(e) {
      if (
        (this.dragBtn || this.dragProcess) &&
        !this.readonly &&
        e.buttons === 0
      ) {
        let duration, tempLabel;
        duration = moment(this.label1).diff(this.label2, "days");
        if (
          this.limit !== null &&
          (Math.abs(duration) > this.limit.max ||
            Math.abs(duration) < this.limit.min)
        ) {
          let _limit = this.limit.max;
          if (Math.abs(duration) <= this.limit.min) {
            _limit = this.limit.min;
          }
          // 拖拽的是第一个select-label
          if (this.isBtn === 1) {
            if (duration < 0) {
              tempLabel = moment(this.label1).add(_limit, "days");
            } else {
              tempLabel = moment(this.label1).subtract(_limit, "days");
            }
            this.label2 = tempLabel;
          } else if (this.isBtn === 2) {
            duration = moment(this.label2).diff(this.label1, "days");
            if (duration < 0) {
              tempLabel = moment(this.label2).add(_limit, "days");
            } else {
              tempLabel = moment(this.label2).subtract(_limit, "days");
            }
            this.label1 = tempLabel;
          }
        }
        this.$nextTick(() => {
          this.dragBtn = false;
          this.dragProcess = false;
          this.isBtn = void 0;
          this.deviation = void 0;
          this.$emit("stop");
        });
      }
    },
    handleResize() {
      this.initialDomSize();
      this.updateLabel();
    },
    updateLabel() {
      this.label1Left =
        (moment(this.label1).diff(this.minValue, "days") / this.period) *
          this.rangeWidth -
        this.selectLabelWidth / 2;
      this.label2Left =
        (moment(this.label2).diff(this.minValue, "days") / this.period) *
          this.rangeWidth -
        this.selectLabelWidth / 2;
    },
    handleCommand(idx) {
      const opt = this.shortcut.options[idx];
      this.activeShortcut = opt.name;
      this.setValue(opt.value);
      this.$emit("stop");
    }
  }
};
</script>

<style lang="scss" scoped>
@import "~@/styles/index.scss";
.range-picker {
  position: relative;
}
.range-picker-wrapper {
  position: relative;
  padding: 0;
  height: 84px;
  padding-top: 31px;
  user-select: none;
}
.not-select-process {
  display: block;
  height: 20px;
  border: 0;
  border-radius: 10px;
}
.label {
  display: inline-block;
  padding: 2px 4px;
  border-radius: 5px;
  cursor: pointer;
  text-align: center;
  white-space: nowrap;
  vertical-align: baseline;
  font-weight: bold;
}
.range-label {
  margin-top: 8px;
  font-size: 14px;
  height: 24px;
  line-height: 24px;
  width: 98px;
  padding: 0 !important;
  left: 0;
  bottom: -32px;
  position: absolute;
}
.end-label {
  left: initial;
  right: 0;
}
.process {
  position: absolute;
  height: 20px;
  top: 0;
  left: 0;
  border-radius: 10px;
  cursor: pointer;
}
.select-label {
  display: block;
  position: absolute;
  bottom: calc(100% + 6px);
  white-space: nowrap;
  height: 24px;
  line-height: 24px;
  font-size: 14px;
  float: left;
  z-index: 1;
  cursor: col-resize;
}
.select-label:after {
  content: "";
  position: absolute;
  top: 100%;
  left: 50%;
  width: 0;
  height: 0;
  margin-left: -4px;
  border: 6px solid transparent;
}
.select-label-2 {
  bottom: initial;
  top: calc(100% + 6px);
}
.select-label-2:after {
  content: "";
  position: absolute;
  top: -12px;
  left: 50%;
  width: 0;
  height: 0;
  margin-left: -4px;
  border: 6px solid transparent;
}
input {
  border: 0;
  font-size: inherit;
  color: inherit;
  text-align: center;
  width: 100%;
  height: 24px;
  line-height: 24px;
  outline: 0 none;
}
.disabled {
  opacity: 0.7;
  cursor: initial;
}
.time-input {
  width: 60px;
  font-weight: 600;
  height: 20px;
  line-height: 20px;
}
.time-input >>> input {
  height: 22px;
  line-height: 22px;
  padding: 0;
}
.time-input >>> .el-input__prefix {
  display: none;
}
.shortcut-dropdown {
  position: absolute;
  right: 0;
  top: 38px;
}
.shortcut-desc {
  width: 50px;
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
  display: inline-block;
}
.range-picker__input >>> input {
  width: 100px;
  height: 24px;
  line-height: 24px;
  padding: 0 10px;
}
</style>
