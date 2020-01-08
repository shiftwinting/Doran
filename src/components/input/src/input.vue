<template>
  <div
     @mouseenter="hovering = true"
     @mouseleave="hovering = false"
    :class="classes">
    <template v-if="type !== 'textarea'">
      <div class="dr-input-prepend" v-if="$slots.prepend">
        <slot name="prepend"></slot>
      </div>
      <input
        :value="value"
        :disabled="disabled"
        :placeholder="placeholder"
        :maxlength="maxlength"
        :type="showPassword ? (passwordVisible ? 'text' : 'password') : type"
        @input="handlerInput"
        @change="handlerChange"
        @focus="handlerFocus"
        @blur="handlerBlur"
        @keyup.enter="handlerEnter"
        @keyup="handlerKeyup"
        @keypress="handlerKeypress"
        @keydown="handlerKeydown"
        ref="input"
        class="dr-input-inner">
      <!-- 前置内容 -->
      <span v-if="prefixIcon || $slots.prefix" class="dr-input-prefix">
        <slot name="prefix">
          <i class="dr-icon-default" :class="prefixIcon"></i>
        </slot>
      </span>
      <!-- 后置内容 -->
      <span v-if="$slots.suffix || suffixIcon || showClear || showPassword || showWordLimit" class="dr-input-suffix">
        <!-- @mousedown.prevent 清除时防止input失去焦点-->
        <slot name="suffix">
          <!-- 其它icon-->
          <i
            v-if="suffixIcon"
            :class="suffixIcon"
            class="dr-icon-default">
          </i>
        </slot>
        <!-- 清除icon-->
        <i
          v-if="showClear"
          @mousedown.prevent
          @click="clearValue"
          class="dr-icon-default dr-icon-roundclose">
        </i>
        <!-- 密码icon-->
        <i
          v-if="showPassword"
          @mousedown.prevent
          @click="passwordVisible = !passwordVisible"
          :class="{
            'dr-icon-attentionfill': passwordVisible,
            'dr-icon-attention': !passwordVisible
          }"
          class="dr-icon-default">
        </i>
        <span v-if="showWordLimit" class="dr-input-count">
          {{valueLength}}/{{maxlength}}
        </span>
      </span>
      <div class="dr-input-append" v-if="$slots.append">
        <slot name="append"></slot>
      </div>
    </template>
    <textarea v-else
      :value="value"
      :disabled="disabled"
      :placeholder="placeholder"
      :type="type"
      :rows="rows"
      :maxlength="maxlength"
      :style="textareaStytles"
      @input="handlerInput"
      @change="handlerChange"
      @focus="handlerFocus"
      @blur="handlerBlur"
      @keyup.enter="handlerEnter"
      @keyup="handlerKeyup"
      @keypress="handlerKeypress"
      @keydown="handlerKeydown"
      ref="textarea"
      class="dr-input-textarea"></textarea>
    <span v-if="showWordLimit && type === 'textarea'" class="dr-input-limit">
      {{valueLength}}/{{maxlength}}
    </span>
  </div>
</template>

<script>
import { validValue } from '@/utils/validate';
import calcTextareaHeight from '@/utils/calcTextareaHeight';

const drPreFixInput = 'dr-input';

export default {
  name: 'DrInput',
  props: {
    value: [String, Number],
    type: {
      validator(val) {
        const arr = ['text', 'textarea', 'password', 'url', 'email', 'date', 'number', 'tel'];
        return validValue(val, arr);
      },
      default: 'text'
    },
    placeholder: {
      type: String,
      default: ''
    },
    suffixIcon: {
      type: String,
      default: ''
    },
    prefixIcon: {
      type: String,
      default: ''
    },
    rows: {
      type: [String, Number],
      default: 2
    },
    size: {
      validator(val) {
        const arr = ['medium', 'small'];
        return validValue(val, arr);
      }
    },
    resize: String, // css resize属性
    maxlength: [Number, String],
    disabled: Boolean,
    clearable: Boolean,
    password: Boolean,
    showWordLimit: Boolean,
    autosize: [Boolean, Object]
  },
  data() {
    return {
      hovering: false,
      passwordVisible: false,
      calcTextareaObj: {}
    };
  },
  computed: {
    classes() {
      return [
        `${drPreFixInput}-default`,
        {
          [`${drPreFixInput}-disabled`]: this.disabled,
          [`${drPreFixInput}--suffix`]: this.clearable || !!this.suffixIcon || !!this.$slots.suffix || this.showWordLimit,
          [`${drPreFixInput}--prefix`]: !!this.prefixIcon || !!this.$slots.prefix,
          [`${drPreFixInput}-${this.size}`]: !!this.size,
          [`${drPreFixInput}--prepend`]: this.$slots.prepend,
          [`${drPreFixInput}--append`]: this.$slots.append
        }
      ];
    },
    isCorrectValue() { // 判断value是不是有值
      const { value } = this;
      return value !== undefined && value !== '' && value !== null
    },
    showClear() { // 是否展示可清除icon
      return this.hovering && this.isCorrectValue && this.clearable && !this.disabled;
    },
    showPassword() { // 是否展示密码显示icon
      return this.type === 'password' && this.password && this.isCorrectValue && !this.disabled;
    },
    valueLength() {
      const val = `${this.value || ''}`;
      return val.length;
    },
    inputDom() {
      return this.$refs.input || this.$refs.textarea;
    },
    textareaStytles() {
      return Object.assign({}, this.calcTextareaObj, { resize: this.resize });
    }
  },
  mounted() {
    this.handlerTextareaHeight();
  },
  methods: {
    focus() {
      this.inputDom.focus();
    },
    blur() {
      this.inputDom.blur();
    },
    select() {
      this.inputDom.select();
    },
    handlerInput(e) {
      const targetValue = e.target.value;
      if (targetValue !== undefined) {
        this.$emit('input', targetValue);
      };
    },
    handlerChange(e) {
      const targetValue = e.target.value;
      this.$emit('change', targetValue);
    },
    handlerFocus(e) {
      this.$emit('focus', e);
    },
    handlerBlur(e) {
      this.$emit('blur', e);
    },
    handlerEnter(e) {
      this.$emit('enter', e);
    },
    handlerKeyup(e) {
      this.$emit('keyup', e);
    },
    handlerKeypress(e) {
      this.$emit('keypress', e);
    },
    handlerKeydown(e) {
      this.$emit('keydown', e);
    },
    clearValue() {
      this.$emit('input', '');
      this.$emit('change', '');
      this.$emit('clear', '');
    },
    handlerTextareaHeight() {
      if (this.$isServer || this.type !== 'textarea') {
        return;
      };
      if (!this.autosize) {
        this.calcTextareaObj = {
          minHeight: calcTextareaHeight(this.$refs.textarea).minHeight
        };
        return;
      };
      const { minRows, maxRows } = this.autosize;
      this.calcTextareaObj = calcTextareaHeight(this.$refs.textarea, minRows, maxRows);
    }
  },
  watch: {
    value() {
      this.$nextTick(this.handlerTextareaHeight);
    }
  }
};
</script>