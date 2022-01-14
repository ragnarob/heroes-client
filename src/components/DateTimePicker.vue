<template>
  <div :class="`customTextInput ${classes}`" :style="wrapperStyle">
    <p v-if="title" class="titleText">
      {{ title }}
    </p>

    <div class="positionRelative innerInputWrapper">
      <input type="datetime-local" :value="value" class="paddedInput" @change="e => $emit('change', e.target.value)"/>
    </div>
  </div>
</template>

<script>
import SearchIcon from 'vue-material-design-icons/Magnify.vue'
import CrossIcon from 'vue-material-design-icons/Close.vue'
import TagsIcon from 'vue-material-design-icons/TagMultiple.vue'

export default {
  components: {
    SearchIcon, CrossIcon, TagsIcon,
  },

  props: {
    value: {
      type: String,
      required: false,
    },
    title: {
      type: String,
      required: false,
    },
    wrapperStyle: {
      type: String,
      required: false,
    },
    classes: {
      type: String,
      required: false,
    },
  },

  data () {
    return {
      localValue: this.value,
    }
  },

  watch: {
    value (newValue) {
      this.localValue = newValue
    },
  },

  methods: {
    clear () {
      this.localValue = ''
      this.$emit('change', '')
      this.$emit('clear')
    },
  },
}
</script>

<style lang="scss" scoped>
@import "../scss/colors.scss";

$paddingBig: 0.5rem;
$paddingSmall: 0.4rem;

.titleText {
  font-size: 0.85rem;
  margin-left: $paddingBig;
  margin-right: $paddingBig;
  text-align: left;
  @media (max-width: 900px) {
    margin-left: $paddingSmall;
    margin-right: $paddingSmall;
  }
}

.innerInputWrapper {
  margin-top: -4px;
}

.paddedInput {
  text-align: left;
  box-sizing: border-box;
  font-family: 'Montserrat', sans-serif;
  padding: 9px $paddingBig;
  font-size: 0.9rem;
  font-weight: 400;
  display: block;
  text-align: center;
  background: transparent;
  outline: none;
  color: #333;
  width: 100%;
  border-width: 0;
  border-style: hidden;
  border-image: linear-gradient(to right, $theme1, $theme2) 1; 
  border-bottom-width: 2px;
  border-bottom-style: solid;
  @media (max-width: 900px) {
    padding: 9px $paddingSmall;
  }
}

.dark {
  .paddedInput {
    color: $darkThemeColor;
  }
}

</style>