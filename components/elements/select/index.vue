<template>
  <div class="dropdown">
    <div
      :tabindex="tabindex"
      @blur="toggle(false)"
      @click="toggle(true)"
      class="dropdown-container"
    >
      <div class="dropdown--arrow">
        <img v-if="(!show && !isClear) || (selected === '' && !show)" src="./images/arrow-d.svg">
        <img v-if="(isClear && !show) && selected !== ''" src="./images/close.svg">
        <img v-if="show" src="./images/arrow-u.svg">
      </div>
      <div v-if="!selected.length" class="dropdown--placeholder">
        <span class="dropdown--title">{{title}}</span>
      </div>
      <div v-else class="dropdown-options">
        <span class="dropdown--option">
          <span>{{selected}}</span>
        </span>
      </div>
    </div>
    <div v-show="show" class="dropdown-selector">
      <ul>
        <li v-for="option in options">
          <a
            @focus.stop="selectOption(option)"
            @click.stop="selectOption(option)"
            href="#"
          >{{option.title}}</a>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    isClear: {
      type: Boolean,
      default: false
    },
    value: {
      default: ""
    },
    options: {
      default: []
    },
    title: {
      default: ""
    },
    tabindex: {
      default: "1"
    },
    select: {
      default: ""
    }
  },
  data() {
    return {
      selected: this.select,
      show: false
    };
  },
  mounted() {
    if (this.value) {
      this.selected = this.value;
    }
  },
  methods: {
    toggle(show) {

      this.show = show
      if (this.show && this.isClear && this.selected !== '') {
        this.selected = '';
        this.$emit("input", null);
      }
    },
    selectOption(option, noToggle) {
      this.selected = option.title;
      this.$emit("input", option.value || option.title);

    }
  }
};
</script>

<style scoped lang="stylus">
.dropdown
  position relative
  ul, li
    margin 0
    padding 0
  .dropdown-selector, .dropdown-container
    border solid 1px #e7e8ea
    border-radius 10px
    background-color #ffffff
  .dropdown-container
    position relative
    display flex
    align-items center
    margin-bottom 8px
    min-height 46px
    cursor pointer
    .dropdown--placeholder
      padding 16px
      line-height 0
      span
        color #bfbfbf
        font-size 12px
    .dropdown-options
      .dropdown--option
        display inline-block
        margin-right 16px
        padding 3px
        span
          display inline-block
          overflow hidden
          max-width 131px
          text-overflow ellipsis
          white-space nowrap
    .dropdown--arrow
      cursor pointer
      img
        position absolute
        top calc(50% - 5px)
        left 16px
        width 16px
        height 10px
  .dropdown-selector
    position absolute
    top 47px
    right 10px
    left 10px
    z-index 1
    box-sizing border-box
    border-top none
    border-top-left-radius 0
    border-top-right-radius 0
    direction ltr
    ul
      li
        a
          display block
          overflow hidden
          padding 5px 15px
          font-size 12px
          font-family Roboto, sans-serif, yekan
          transition all ease 0.6s
          &:hover
            background #f7f8fb
        &:first-child
          a
            padding-top 10px
        &:last-child
          a
            padding-bottom 10px
</style>