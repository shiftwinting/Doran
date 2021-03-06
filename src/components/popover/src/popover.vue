<template>
  <div class="dr-popover">
    <div class="dr-popover-reference"
      @click="handlerClick"
      v-click-outside="handlerClickOutside"
      @mouseenter="handlerMouseEnter"
      @mouseleave="handlerMouseLeave"
      @mousedown="handlerFocus"
      @mouseup="handlerBlur"
      ref="reference">
      <slot></slot>
    </div>
    <transition name="popover-opacity">
      <DrPopper
        @mouseenter.native="handlerMouseEnter"
        @mouseleave.native="handlerMouseLeave"
        :transfer="transfer"
        :placement="placement"
        :visible="isVisible || visible"
        v-show="isVisible || visible"
        :offset="`0 ${offset}px`"
        :showArrow="showArrow"
        :style="{width: `${width}px`}"
        ref="popper"
        :class="classes">
        <slot name="content">
          <div class="dr-popover-title">{{title}}</div>
          <div class="dr-popover-content">{{content}}</div>
        </slot>
      </DrPopper>
    </transition>
  </div>
</template>

<script>
import DrPopper from '@/base/popper';
import { directive as clickOutside } from 'v-click-outside-x'; // 点击dom外部时

import { validValue } from '@/utils/validate';
import emitter from '@/mixins/emitter';
import { arrowInit } from '@/mixins/base';

const popoverPreFix = 'dr-popover'

export default {
  name: 'DrPopover',
  directives: {
    clickOutside
  },
  mixins: [emitter, arrowInit],
  props: {
    placement: {
      validator (value) {
        const arr = ['top', 'bottom', 'top-start', 'bottom-start', 'top-end', 'bottom-end', 'left', 'left-start', 'left-end', 'right', 'right-start', 'right-end'];
        return validValue(value, arr);
      },
      default: 'top-start'
    },
    trigger: {
      validator (value) {
        return validValue(value, ['hover', 'click', 'focus', 'custom']);
      },
      default: 'hover'
    },
    transfer: {
      type: Boolean,
      default() {
        return !!this.$DORAN.transfer
      }
    },
    title: {
      type: String,
      default: 'title'
    },
    offset: {
      type: Number,
      default: 12
    },
    content: {
      type: String,
      default: 'content'
    },
    visible: Boolean,
    showArrow: {
      type: Boolean,
      default: true
    },
    popoverClass: String,
    width: Number
  },
  components: {
    DrPopper
  },
  data() {
    return {
      isVisible: this.visible,
      timeOut: ''
    }
  },
  computed: {
    classes() {
      return [
        `${popoverPreFix}-popper`,
        this.popoverClass
      ]
    }
  },
  methods: {
    handlerClick() {
      if (this.trigger === 'click') {
        this.isVisible = !this.isVisible;
      }
    },
    handlerClickOutside() {
      if (this.trigger === 'click') {
        this.isVisible = false;
      }
    },
    handlerFocus() {
      if (this.trigger === 'focus') {
        this.isVisible = true;
      }
    },
    handlerBlur() {
      if (this.trigger === 'focus') {
        this.isVisible = false;
      }
    },
    handlerMouseEnter() {
      if (this.trigger === 'hover') {
        // 延迟出现，防止鼠标快速hover时的显示bug
        // 移入或者移出的时候互相清除对方内部的计时器，执行自己内部的计时器
        this.timeOut && clearTimeout(this.timeOut);
        this.timeOut = setTimeout(() => {
          this.isVisible = true;
        }, 251);
      }
    },
    handlerMouseLeave() {
      if (this.trigger === 'hover') {
        this.timeOut && clearTimeout(this.timeOut);
        this.timeOut = setTimeout(() => {
          this.isVisible = false;
        }, 150);
      }
    }
  },
  watch: {
    isVisible(newValue) {
      this.broadcast('DrPopper', newValue ? 'onUpdatePopper' : 'onDestoryPopper');
      if (newValue) {
        this.arrowInit(this.$refs.popper.$refs.arrow);
      }
    },
    visible(newValue) {
      if (this.trigger !== 'custom') {
        return;
      }
      this.broadcast('DrPopper', newValue ? 'onUpdatePopper' : 'onDestoryPopper');
      if (newValue) {
        this.arrowInit(this.$refs.popper.$refs.arrow);
      }
    }
  },
  beforeDestroy() {
    this.$refs.popper.removeChild(); // 销毁时移除popper的dom元素
  }
}
</script>
