<template>
  <label class="customRadio">
    <input type="radio"
           @click="$emit('onChange', value)"/>
    <span style="color: white;" :class="{selectedRadioSpan: currentValue === value}">
    </span>
    {{label}}
  </label>
</template>

<script>

export default {
  props: {
    label: String,
    value: String,
    currentValue: String,
  },

  data () {
    return {
    }
  },
}
</script>

<style lang="scss">
@import "../scss/colors.scss";

.customRadio {
  display: flex;
  flex-direction: row;
  align-items: center;

  &:hover {
    cursor: pointer;
  }

  &>span {
    width: 0.8rem;
    height: 0.8rem;
    border: 1px solid #aaa;
    margin-right: 0.25rem;
    display: inline-block;
    border-radius: 80px;
    transition: all linear 0.1s;
    &:after {
      content: "";
      position: absolute;
      top: -1px;
      left: 6px;
      border-bottom: 2px solid #fff;
      border-right: 2px solid #fff;
      height: 9px;
      width: 4px;
      transform: rotate(45deg);
      visibility: hidden;
    }
  }
  
  input {
    display: none;
  }
  .selectedRadioSpan {
    border-color: $seasonColor2;
    background: $seasonColor2;
  }
}
</style>