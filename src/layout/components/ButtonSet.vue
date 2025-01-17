<template>
  <div>
    <slot ref="default_slot" name="default" />
    <template v-if="show_button.length > 0">
      <template v-for="index in arrayRange(show_num)">
        <el-button
          :key="index"
          :ref="`btn_${index}`"
          :type="show_button[index].getAttr('type', 'text')"
          :class="show_button[index].getAttr('class', '')"
          :style="show_button[index].style"
          :size="show_button[index].getAttr('size', 'small')"
          @click="show_button[index].hasKey('click') ? show_button[index].click(scope) : () => {}"
          @mouseover.native="set_background_color(index)"
          @mouseout.native="set_background_color(index, 'out')"
        >
          {{ typeof show_button[index].text === "function" ? show_button[index].text(scope) : show_button[index].text }}
        </el-button>
      </template>
    </template>
    <template v-if="popover_num > 0">
      <el-popover trigger="hover" :width="popover_width" placement="bottom" style="margin-left: 10px" popper-class="button-set-popover">
        <template v-for="index in arrayRange(show_num, show_button.length)">
          <div :key="index" class="popover_btn" style="text-align: center">
            <el-button
              :ref="`btn_${index}`"
              :type="show_button[index].getAttr('type', 'text')"
              :class="show_button[index].hasKey('class') ? [show_button[index].class, 'popover_btn_item'] : 'popover_btn_item'"
              :style="Object.assign({width: popover_width+'px'}, show_button[index].style)"
              :size="show_button[index].getAttr('size', 'normal')"
              @click="show_button[index].hasKey('click') ? show_button[index].click(scope) : () => {}"
              @mouseover.native="set_background_color(index)"
              @mouseout.native="set_background_color(index, 'out')"
            >
              {{ typeof show_button[index].text === "function" ? show_button[index].text(scope) : show_button[index].text }}
            </el-button>
          </div>
        </template>
        <i slot="reference" class="el-icon-more" />
      </el-popover>
    </template>
  </div>
</template>

<script>
import { arrayRange, getColor, hexToRgb } from '@/utils/common_function'

export default {
  name: 'ButtonSet',
  props: {
    scope: {
      // 单元格scope参数
      type: Object,
      required: true,
      default: function() {
        return {}
      }
    },
    buttonData: {
      // 按钮配置
      type: Array,
      required: true,
      default: function() {
        return []
      }
    },
    showBtnNum: {
      // 显示的按钮数量，会根据列宽度自适应增加或减少
      type: Number,
      default: 1
    }
  },
  data() {
    return {
      minWidth: 0
    }
  },
  computed: {
    show_button: function() {
      const show_button = this.buttonData.filter((item) => {
        if (item.hasKey('v-if')) {
          if (!item['v-if'](this.scope)) {
            return false
          }
        }
        if (item.hasKey('v-permission')) {
          if (!this.$store.getters.role) {
            return false
          }
          if (Array.isArray(item['v-permission'])) {
            let res = false
            item['v-permission'].forEach((val) => {
              if (this.$store.getters.permissions.includes(val)) {
                res = true
              }
            })
            return res
          } else if (!this.$store.getters.permissions.includes(item['v-permission'])) {
            return false
          }
        }
        return true
      })
      show_button.map((item, index) => {
        const style = {}
        if (!item.hasKey('color')) {
          style.color = '#409EFF'
        } else {
          style.color = getColor(item.color)
        }
        const rgb = this.hexToRgb(style.color)
        style.background_color = `rgba(${rgb.r},${rgb.g},${rgb.b},0.10)`
        show_button[index].style = style
      })
      return show_button
    },
    show_btn_num: function() {
      let show_btn_num = this.showBtnNum
      let show_width = 44 // 单元格基本的20px padding，加上 省略号 宽度
      const font_size = 14
      const max_num =
        this.show_button.length > this.showBtnNum
          ? this.showBtnNum
          : this.show_button.length
      // 检查给定显示按钮的数量，显示出来的宽度是否会大于给定的最小宽度，超过的话，就减少显示按钮数量
      for (let i = 0; i < max_num; i++) {
        let text = this.show_button[i].text
        if (typeof this.show_button[i].text === 'function') {
          text = this.show_button[i].text(this.scope)
        }
        show_width += text.length * font_size
        if (i > 0) {
          show_width += 10 // 按钮和按钮之间有10px间距
        }
        if (show_width > this.minWidth) {
          show_btn_num--
        }
      }
      return show_btn_num
    },
    popover_num: function() {
      // 如果隐藏的数量是一个，就直接显示出来
      const reduce_num = this.show_button.length - this.show_btn_num
      if (
        reduce_num === 1 &&
        this.show_button[this.show_btn_num].text.length > 2
      ) {
        return 1
      }
      return reduce_num > 1 ? reduce_num : 0
    },
    show_num: function() {
      return this.show_button.length - this.popover_num
    },
    popover_width: function() {
      // 获取弹出框最大宽度
      const font_size = 14
      let width = 0
      if (this.popover_num > 0) {
        for (let i = this.show_num; i < this.show_button.length; i++) {
          let text = this.show_button[i].text
          if (typeof this.show_button[i].text === 'function') {
            text = this.show_button[i].text(this.scope)
          }
          const row_width = text.length * font_size
          if (row_width > width) {
            width = row_width
          }
        }
      }
      return width + 10
    }
  },
  mounted() {
    this.minWidth = this.$el.offsetWidth
  },
  methods: {
    arrayRange,
    hexToRgb,
    set_background_color(index, type = 'in') {
      if (type === 'in') {
        this.$refs[`btn_${index}`][0].$el.style['background-color'] =
          this.show_button[index].style.background_color
      } else {
        this.$refs[`btn_${index}`][0].$el.style['background-color'] = ''
      }
    }
  }
}
</script>

<style>
.button-set-popover {
  min-width: 28px;
  box-sizing: unset;
}
</style>
<style lang="scss" scoped>
.el-icon-more {
  color: #5b90ef;
  height: 28px;
  line-height: 28px;
}
</style>
