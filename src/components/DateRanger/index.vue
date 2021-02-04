<template>
  <div :class="['uni-date-ranger', skin]">
    <div class="range-picker-wrapper" :style="wrapperStyle">
      <div class="range-picker">
        <span class="not-select-process" />
        <span v-if="showRangeLabel" class="label-panel range-label" @dblclick="editMin">
          <template v-if="!edit.min">{{ minValueFmt }}</template>
          <el-date-picker
            v-else
            v-model="edit.minValue"
            v-focus
            type="date"
            size="mini"
            class="range-picker__input"
            :editable="false"
            :clearable="false"
            :picker-options="edit.minPickerOption"
            @blur="blurEditMin"
          />
        </span>
        <span v-if="showRangeLabel" class="label-panel range-label end-label" @dblclick="editMax">
          <template v-if="!edit.max">{{ maxValueFmt }}</template>
          <el-date-picker
            v-else
            v-model="edit.maxValue"
            v-focus
            type="date"
            size="mini"
            class="range-picker__input"
            :editable="false"
            :clearable="false"
            :picker-options="edit.maxPickerOption"
            @blur="blurEditMax"
          />
        </span>
        <span :class="['process', { disabled: readonly }]" :style="processStyle" />
        <span
          :class="['label-panel select-label', { 'show-time': showTime, active: isBtn === 1, disabled: readonly }]"
          :style="{ left: `${label1Left}px` }"
          @dblclick="editLabel1"
        >
          <template v-if="!edit.label1">{{ label1Fmt }}</template>
          <template v-else>
            {{ label1Fmt.split(' ')[0] }}
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
          :class="[
            'label-panel select-label select-label-2',
            { 'show-time': showTime, active: isBtn === 2, disabled: readonly }
          ]"
          :style="{ left: `${label2Left}px` }"
          @dblclick="editLabel2"
        >
          <template v-if="!edit.label2">{{ label2Fmt }}</template>
          <template v-else>
            {{ label2Fmt.split(' ')[0] }}
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
    <el-dropdown v-if="shortcut.show" class="shortcut-dropdown" @command="handleCommand">
      <el-button size="mini" type="primary" class="padding-5"
        ><span class="shortcut-desc">{{ activeShortcut }}</span
        ><i class="el-icon-arrow-up margin-left-5"
      /></el-button>
      <el-dropdown-menu slot="dropdown" class="padding-5">
        <el-dropdown-item
          v-for="(opt, idx) of shortcut.options"
          v-cloak
          :key="idx"
          :class="['dropdown-item', { active: activeShortcut === opt.name }]"
          :command="idx"
          >{{ opt.name }}</el-dropdown-item
        >
      </el-dropdown-menu>
    </el-dropdown>
  </div>
</template>

<script>
import dayjs from 'dayjs'
import { uuid, isString } from "@/utils"
import { addResizeListener, removeResizeListener } from "@/utils/resize-event"
export default {
  directives: {
    focus: {
      inserted: el => el.querySelector('input').focus()
    }
  },
  inheritAttrs: false,
  props: {
    id: {
      type: String,
      default: uuid()
    },
    skin: {
      tpye: String,
      default: 'light',
      validator: val => ['light', 'dark'].includes(val)
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
        }
      }
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
      rangeWidth: void 0,
      isBtn: void 0,
      defaultValue: this.value,
      wrapperStyle: null,
      formatYMD: Object.freeze('YYYY-MM-DD'),
      formatYMDHms: Object.freeze('YYYY-MM-DD HH:mm:ss'),
      edit: {
        enable: true,
        minValue: '',
        maxValue: '',
        min: false,
        max: false,
        label1: false,
        label2: false,
        label1Time: '',
        label2Time: '',
        minPickerOption: {
          disabledDate: time => {
            const { maxValue } = this
            const maxDate = dayjs(maxValue, this.formatYMD)
            let isAfter = dayjs(time, this.formatYMD).isAfter(maxDate)
            if (!isAfter && this.limit !== null && this.limit.min) {
              return dayjs(time, this.formatYMD)
                .add(this.limit.min, 'days')
                .isAfter(maxDate)
            }
            return isAfter
          }
        },
        maxPickerOption: {
          disabledDate: time => {
            const { minValue } = this
            const minDate = dayjs(minValue, this.formatYMD)
            let isBefore = dayjs(time, this.formatYMD).isBefore(minDate)
            if (!isBefore && this.limit !== null && this.limit.min) {
              return dayjs(time, this.formatYMD)
                .subtract(this.limit.min, 'days')
                .isBefore(minDate)
            }
            return isBefore
          }
        }
      },
      activeShortcut: this.shortcut.show && this.shortcut.options.length > 0 ? this.shortcut.options[0].name : void 0
    }
  },
  computed: {
    period() {
      return dayjs(this.maxValue).diff(this.minValue, 'days')
    },
    processStyle() {
      if (this.rangeWidth !== undefined) {
        let _left = Math.min(this.label1Left, this.label2Left) + this.selectLabelWidth / 2
        let _right = this.rangeWidth - Math.max(this.label1Left, this.label2Left) - this.selectLabelWidth / 2
        if (_left < 0) _left = 0
        if (_right < 0) _right = 0
        return {
          left: `${_left}px`,
          right: `${_right}px`
        }
      }
      return null
    },
    minValueFmt() {
      if (this.minValue === undefined || !this.minValue || !dayjs(this.minValue).isValid()) {
        return ''
      }
      return dayjs(this.minValue).format(this.formatYMD)
    },
    maxValueFmt() {
      if (this.maxValue === undefined || !this.maxValue || !dayjs(this.minValue).isValid()) {
        return ''
      }
      return dayjs(this.maxValue).format(this.formatYMD)
    },
    label1Fmt() {
      if (this.label1 === undefined || !this.label1 || !dayjs(this.label1).isValid()) {
        return ''
      }
      let fmtStr = dayjs(this.label1).format(this.formatYMD)
      if (this.showTime) {
        fmtStr += ' ' + this.edit.label1Time
      }
      return fmtStr
    },
    label2Fmt() {
      if (this.label2 === undefined || !this.label2 || !dayjs(this.label2).isValid()) {
        return ''
      }
      let fmtStr = dayjs(this.label2).format(this.formatYMD)
      if (this.showTime) {
        fmtStr += ' ' + this.edit.label2Time
      }
      return fmtStr
    }
  },
  watch: {
    value(val) {
      if (val) {
        const tempVal = {}
        if (dayjs(val.min).isValid()) {
          tempVal.min = val.min
        } else if (isString(val.min) && dayjs(val.min, this.formatYMD).isValid()) {
          tempVal.min = dayjs(val.min, this.formatYMD)
        }
        if (dayjs(val.max).isValid()) {
          tempVal.max = val.max
        } else if (isString(val.max) && dayjs(val.max, this.formatYMD).isValid()) {
          tempVal.max = dayjs(val.max, this.formatYMD)
        }
        // 更新label
        this.setValue(tempVal)
        // 更新label位置
        this.updateLabel()
        // 更新值
        this.defaultValue = tempVal
      }
    }
  },
  mounted() {
    this.initialExtraParams()
    this.$nextTick(() => {
      // 校验语法格式
      this.checkSyntax()
      // 初始化默认边界值
      if (this.defaultValue === null) {
        this.label1 = this.label2 = this.minValue
      } else {
        // 校验数据值
        if (this.limit !== null && this.limit.min && this.limit.max) {
          const differ = Math.abs(dayjs(this.label2).diff(this.label1, 'days'))
          if (differ > this.limit.max || differ < this.limit.min) {
            throw new Error('默认时间区间不在limit限制值区间内')
          }
        }
        // 如果带时间，默认给定
        if (this.showTime) {
          this.edit.label1Time = '00:00:00'
          this.edit.label2Time = '23:59:59'
        }
        // 设置默认值
        this.setValue(this.defaultValue)
      }
      // 更新label位置
      this.updateLabel()
      // 改变父类值
      this.handleValueChange()
      // 注册事件
      this.$el.addEventListener('mousedown', e => this.initMousedownEvent(e))
      this.$el.addEventListener('mousemove', e => this.initMousemoveEvent(e))
      this.$el.addEventListener('mouseup', e => this.initMouseupEvent(e))
      // document.body.addEventListener('mouseup', e => this.initMouseupEvent(e))
      // document.body.addEventListener('mouseout', e => this.initMouseupEvent(e))
      addResizeListener(this.$el, this.handleResize)
      // 初始化容器宽度
      this.initialDomSize()
    })
  },
  beforeDestroy() {
    // help GC
    this.selectLabelWidth = null
    this.deviation = null
    this.dragBtn = null
    this.dragProcess = null
    // remove events
    this.$el.removeEventListener('mousedown', e => this.initMousedownEvent(e))
    this.$el.removeEventListener('mousemove', e => this.initMousemoveEvent(e))
    this.$el.removeEventListener('mouseup', e => this.initMouseupEvent(e))
    document.body.removeEventListener('mouseup', e => this.initMouseupEvent(e))
    document.body.removeEventListener('mouseout', e => this.initMouseupEvent(e))
    if (this.$el && this.handleResize) {
      removeResizeListener(this.$el, this.handleResize)
    }
  },
  methods: {
    /**
     * 初始化额外的参数，不需要被vue管理
     */
    initialExtraParams() {
      this.selectLabelWidth = void 0
      this.deviation = void 0
      this.dragBtn = false
      this.dragProcess = false
    },

    /**
     * 校验边界min和max
     */
    checkSyntax() {
      if (isString(this.minValue) && dayjs(this.minValue, this.formatYMD).isValid()) {
        this.minValue = dayjs(this.minValue, this.formatYMD)
      } else if (!dayjs(this.minValue, this.formatYMD).isValid()) {
        throw new SyntaxError('min值必须是YYYY-MM-DD的日期字符串或dayjs对象')
      }
      if (!dayjs(this.maxValue).isValid() && isString(this.maxValue) && dayjs(this.maxValue, this.formatYMD)) {
        this.maxValue = dayjs(`${this.maxValue} 23:59:59`, this.formatYMDHms)
      } else if (!dayjs(this.maxValue, this.formatYMD).isValid()) {
        throw new SyntaxError('max值必须是YYYY-MM-DD的日期字符串或dayjs对象')
      }
      if (this.minValue === null || this.maxValue === null) {
        throw new Error('min和max属性必传')
      }
      if (this.shortcut.show && this.shortcut.options.length <= 0) {
        throw new Error('快捷时间段显示时，须设置内容')
      }
      if (
        this.value &&
        (dayjs(this.value.min, this.formatYMD).isBefore(this.minValue) ||
          dayjs(this.value.max, this.formatYMD).isAfter(this.maxValue))
      ) {
        throw new RangeError('越界：错误的时间区间')
      }
    },

    /**
     * 组装值并响应父组件值改变
     */
    handleValueChange() {
      let min, max
      const isBefore = dayjs(this.label1, this.formatYMD).isBefore(dayjs(this.label2, this.formatYMD))
      if (isBefore) {
        min = this.label1Fmt
        max = this.label2Fmt
      } else {
        min = this.label2Fmt
        max = this.label1Fmt
      }
      this.emitValue(min, max)
    },

    /**
     * 设置默认值
     * @param {min, max} 默认值
     */
    setValue({ min, max }) {
      let minValue, maxValue
      let fmt = this.formatYMD
      if (this.showTime) {
        fmt = this.formatYMDHms
      }
      if (dayjs(min).isValid()) {
        minValue = dayjs(min, fmt)
      } else if (isString(min) && dayjs(min, this.formatYMD).isValid()) {
        minValue = dayjs(min, fmt)
      }
      if (dayjs(max).isValid()) {
        maxValue = dayjs(max, fmt)
      } else if (isString(max) && dayjs(max, this.formatYMD).isValid()) {
        maxValue = dayjs(max, fmt)
      }
      let isAfter = false
      if (this.label1 && this.label2) {
        isAfter = dayjs(this.label1, this.formatYMD).isAfter(dayjs(this.label2, this.formatYMD))
      }
      if (isAfter) {
        this.label1 = maxValue
        this.label2 = minValue
      } else {
        this.label1 = minValue
        this.label2 = maxValue
      }
    },

    /**
     * 触发值改变
     * @param min 最小值
     * @param max 最大值
     */
    emitValue(min, max) {
      let fmtStr = this.formatYMD
      if (this.showTime) {
        fmtStr = this.formatYMDHms
      }
      this.$emit('input', {
        min: dayjs(min).format(fmtStr),
        max: dayjs(max).format(fmtStr)
      })
    },

    /**
     * 初始化DOM的宽度
     */
    initialDomSize() {
      this.selectLabelWidth = this.$el.querySelector('.select-label').offsetWidth
      this.rangeWidth = this.$el.offsetWidth - this.selectLabelWidth
      this.wrapperStyle = {
        'padding-left': `${this.selectLabelWidth / 2}px`,
        'padding-right': `${this.selectLabelWidth / 2}px`
      }
    },

    /**
     * 获取指定元素相对左边的位置
     * @param {HTMLElement} e dom元素
     * @returns {Number}
     */
    elementLeft(e) {
      let offset = e.offsetLeft
      if (e.offsetParent !== null) {
        offset += this.elementLeft(e.offsetParent)
      }
      return offset
    },

    /**
     * 编辑最小边界值
     */
    editMin() {
      if (this.readonly || this.edit.min) return
      this.edit.min = true
      this.edit.minValue = this.minValueFmt
    },

    /**
     * 编辑最大边界值
     */
    editMax() {
      if (this.readonly || this.edit.max) return
      this.edit.max = true
      this.edit.maxValue = this.maxValueFmt
    },

    /**
     * 编辑label1对应的时间time
     */
    editLabel1() {
      if (this.readonly || !this.showTime || this.edit.label1) return
      this.edit.label1 = true
      this.edit.label1Time = this.label1Fmt.split(' ')[1]
      if (this.edit.label2) this.edit.label2 = false
    },

    /**
     * 编辑label2对应的时间time
     */
    editLabel2() {
      if (this.readonly || !this.showTime || this.edit.label2) return
      this.edit.label2 = true
      this.edit.label2Time = this.label2Fmt.split(' ')[1]
      if (this.edit.label1) this.edit.label1 = false
    },

    /**
     * 改变label1的值
     */
    changeLabel1() {
      this.edit.label1 = false
      const time = this.edit.label1Time.split(':')
      // const
      this.label1.hour(time[0])
      this.label1.minute(time[1])
      this.label1.second(time[2])
      this.handleValueChange()
    },

    /**
     * 改变label2的值
     */
    changeLabel2() {
      this.edit.label2 = false
      const time = this.edit.label2Time.split(':')
      this.label2.hour(time[0])
      this.label2.minute(time[1])
      this.label2.second(time[2])
      this.handleValueChange()
    },

    /**
     * 改变边界最小值的回执
     */
    blurEditMin() {
      this.edit.min = false
      const tempMin = dayjs(this.edit.minValue, this.formatYMD)
      if (tempMin.isSame(this.minValue)) return
      if (this.rangeLimit > 0) {
        const tempMax = dayjs(this.edit.minValue, this.formatYMD).add(this.rangeLimit, 'days')
        if (dayjs().isBefore(tempMax)) {
          this.$message.warning(
            `日期选择区间限定max日期与min日期相差${this.rangeLimit}天, 该日期设置导致max日期已超过当天期, 请重新设置`
          )
          return
        }
        this.maxValue = tempMax
      }
      this.minValue = tempMin
      // 给定默认值
      this.label1 = this.label2 = this.minValue
      // 更新label位置
      this.updateLabel()
      // 改变父类值
      this.handleValueChange()
      // 回执
      this.$emit('change-bound', this.id, {
        min: dayjs(this.minValue).format(this.formatYMD),
        max: dayjs(this.maxValue).format(this.formatYMD)
      })
    },

    /**
     * 改变边界最大值的回执
     */
    blurEditMax() {
      this.edit.max = false
      const temp = dayjs(this.edit.maxValue, this.formatYMD)
      if (temp.isSame(this.maxValue)) return
      if (this.rangeLimit > 0) {
        this.minValue = dayjs(this.edit.maxValue, this.formatYMD).subtract(this.rangeLimit, 'days')
      }
      this.maxValue = temp
      // 给定默认值
      this.label1 = this.label2 = this.minValue
      // 更新label位置
      this.updateLabel()
      // 改变父类值
      this.handleValueChange()
      // 回执
      this.$emit('change-bound', this.id, {
        min: dayjs(this.minValue).format(this.formatYMD),
        max: dayjs(this.maxValue).format(this.formatYMD)
      })
    },

    /**
     * 记录当前时间段对应的宽度
     * @param {MouseEvent} e 鼠标对象
     * @returns {Number}
     */
    getTempWidth(e) {
      let tempWidth
      const x = e.pageX - this.elementLeft(this.$el)
      if (x - this.deviation > this.rangeWidth) {
        tempWidth = this.rangeWidth - this.selectLabelWidth / 2
      } else if (x - this.deviation < 0) {
        tempWidth = -this.selectLabelWidth / 2
      } else {
        tempWidth = x - this.selectLabelWidth / 2 - this.deviation
      }
      return tempWidth
    },

    /**
     * mousedown事件
     * @param {MouseEvent} e 鼠标对象
     */
    initMousedownEvent(e) {
      if (this.readonly) return
      const target = e.target || e.toElement
      if (target !== null) {
        // 选中单个label
        if (target.classList.contains('select-label')) {
          this.dragBtn = true
          // 记录鼠标相对于选中dom开头箭头位置的偏差
          this.deviation = e.pageX - this.elementLeft(target)
          if (target.classList.contains('select-label-2')) {
            this.isBtn = 2
          } else {
            this.isBtn = 1
          }
          return
        }
        // 选中process进度
        if (target.classList.contains('process')) {
          this.dragProcess = true
          // 记录鼠标相对于选中dom开头箭头位置的偏差
          this.deviation = e.pageX - this.elementLeft(target)
        }
      }
    },

    /**
     * mousemove事件
     * @param {MouseEvent} e 鼠标对象
     */
    initMousemoveEvent(e) {
      if (this.readonly) return
      if (e.buttons === 1) {
        // 拖动的是label
        if (this.dragBtn) {
          // 计算当前宽度值
          const tempWidth = this.getTempWidth(e)
          // 根据比例尺获取当前时间
          const current = dayjs(this.minValue).add(
            parseInt((this.period / this.rangeWidth) * (tempWidth + this.selectLabelWidth / 2)),
            'days'
          )
          if (this.isBtn === 1) {
            this.label1 = current
          } else if (this.isBtn === 2) {
            this.label2 = current
          }
          // 更新label位置
          this.updateLabel()
          // 改变父类值
          this.handleValueChange()
          return
        }
        // 拖动的是process块
        if (this.dragProcess) {
          // 计算当前宽度值
          const tempWidth = e.pageX - this.elementLeft(this.$el) - this.deviation
          const current = dayjs(this.minValue).add(
            ((this.period / this.rangeWidth) * (tempWidth - this.selectLabelWidth / 2)).toFixed(0),
            'days'
          )
          const duration = Math.abs(dayjs(this.label1).diff(this.label2, 'days'))
          // 更新当前时间
          let tmp1 = current
          let tmp2 = dayjs(current).add(duration, 'days')
          if (current.isBefore(this.minValue)) {
            tmp1 = this.minValue
            tmp2 = dayjs(this.minValue).add(duration, 'days')
          } else if (tmp2.isAfter(this.maxValue)) {
            tmp1 = dayjs(this.maxValue).subtract(duration, 'days')
            tmp2 = this.maxValue
          }
          if (dayjs(this.label2).isBefore(this.label1)) {
            this.label1 = tmp2
            this.label2 = tmp1
          } else {
            this.label1 = tmp1
            this.label2 = tmp2
          }
          // 更新label位置
          this.updateLabel()
          // 改变父类值
          this.handleValueChange()
        }
      }
    },

    /**
     * mouseup事件
     * @param {MouseEvent} e 鼠标对象
     */
    initMouseupEvent(e) {
      if ((this.dragBtn || this.dragProcess) && !this.readonly && e.buttons === 0) {
        let duration, tempLabel
        duration = dayjs(this.label1).diff(this.label2, 'days')
        if (this.limit !== null && (Math.abs(duration) > this.limit.max || Math.abs(duration) < this.limit.min)) {
          let _limit = this.limit.max
          if (Math.abs(duration) <= this.limit.min) {
            _limit = this.limit.min
          }
          // 拖拽的是第一个select-label
          switch (this.isBtn) {
            case 1:
              if (duration < 0) {
                tempLabel = dayjs(this.label1).add(_limit, 'days')
              } else {
                tempLabel = dayjs(this.label1).subtract(_limit, 'days')
              }
              if (tempLabel.isBefore(this.minValue)) {
                tempLabel = dayjs(this.label1).add(_limit, 'days')
              } else if (tempLabel.isAfter(this.maxValue)) {
                tempLabel = dayjs(this.label1).subtract(_limit, 'days')
              }
              // 防止越界
              if (!tempLabel.isBefore(this.minValue) && !tempLabel.isAfter(this.maxValue)) {
                this.label2 = tempLabel
              } else {
                this.label2 = this.minValue
                this.label1 = dayjs(this.minValue).add(_limit, 'days')
              }
              break
            case 2:
              duration = dayjs(this.label2).diff(this.label1, 'days')
              if (duration < 0) {
                tempLabel = dayjs(this.label2).add(_limit, 'days')
              } else {
                tempLabel = dayjs(this.label2).subtract(_limit, 'days')
              }
              if (tempLabel.isBefore(this.minValue)) {
                tempLabel = dayjs(this.label2).add(_limit, 'days')
              } else if (tempLabel.isAfter(this.maxValue)) {
                tempLabel = dayjs(this.label2).subtract(_limit, 'days')
              }
              // 防止越界
              if (!tempLabel.isBefore(this.minValue) && !tempLabel.isAfter(this.maxValue)) {
                this.label1 = tempLabel
              } else {
                this.label1 = this.maxValue
                this.label2 = dayjs(this.maxValue).subtract(_limit, 'days')
              }
              break
            default:
              break
          }
          // 更新label位置
          this.updateLabel()
          // 改变父类值
          this.handleValueChange()
        }
        this.$nextTick(() => {
          this.dragBtn = false
          this.dragProcess = false
          this.isBtn = void 0
          this.deviation = void 0
          this.$emit('stop')
        })
      }
    },

    /**
     * resize
     */
    handleResize() {
      // resize dom
      this.initialDomSize()
      // resize label position
      this.updateLabel()
    },

    /**
     * 更新label对应的left位置
     */
    updateLabel() {
      this.label1Left =
        (dayjs(this.label1).diff(this.minValue, 'days') / this.period) * this.rangeWidth - this.selectLabelWidth / 2
      this.label2Left =
        (dayjs(this.label2).diff(this.minValue, 'days') / this.period) * this.rangeWidth - this.selectLabelWidth / 2
    },

    /**
     * 快捷方式选择回执
     * @param {Number | String} idx 标识
     */
    handleCommand(idx) {
      const opt = this.shortcut.options[idx]
      this.activeShortcut = opt.name
      this.setValue(opt.value)
      this.$emit('stop')
    }
  }
}
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
  box-sizing: content-box;
  border-radius: 10px;
}
.label-panel {
  display: inline-block;
  padding: 0;
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
  height: 28px;
  line-height: 28px;
  width: 98px;
  padding: 0 !important;
  left: 0;
  bottom: -32px;
  position: absolute;
  /deep/ .el-date-editor {
    width: 100%;
  }
  /deep/ .el-input__prefix {
    display: none;
  }
}
.end-label {
  left: initial;
  right: 0;
}
.process {
  position: absolute;
  height: 20px;
  top: 1px;
  left: 0;
  border-radius: 10px;
  cursor: pointer;
}
.select-label {
  display: block;
  position: absolute;
  bottom: calc(100% + 6px);
  white-space: nowrap;
  width: 80px;
  height: 24px;
  line-height: 24px;
  font-size: 14px;
  float: left;
  z-index: 1;
  transition: all ease;
  cursor: col-resize;
}
.select-label.show-time {
  width: 140px;
}
.select-label:after {
  content: '';
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
  content: '';
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
.time-input {
  width: 60px !important;
  /deep/ input {
    width: 100%;
    height: 22px;
    line-height: 22px;
    padding: 0;
  }
}
.time-input {
  /deep/ .el-input__prefix {
    display: none;
  }
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
.range-picker__input {
  /deep/ input {
    height: 24px;
    line-height: 24px;
    padding: 0 10px;
  }
}
</style>
