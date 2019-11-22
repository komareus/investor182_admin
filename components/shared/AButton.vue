<template>
  <button
    class="a-button"
    :class="{ 'a-button--outline': outline, 'a-button--disabled': disabled }"
    @click="onClick"
  >
    <slot></slot>
  </button>
</template>

<script>
export default {
  name: 'AButton',
  props: {
    outline: {
      type: Boolean,
      default: false
    },
    disabled: {
      type: Boolean,
      default: false
    }
  },
  methods: {
    onClick() {
      if (!this.disabled) {
        this.$emit('click')
      }
    }
  }
}
</script>

<style scoped lang="scss">
  .a-button{
    padding: 6px 8px;
    border: none;
    border-radius: 6px;
    background-color: $c--primary;
    min-width: 80px;
    width: 100%;
    height: 100%;
    color: white;
    text-transform: uppercase;
    cursor: pointer;
    transition: all .2s ease-in-out;
    &:hover{
      background-color: $c--secondary;
    }
    &--outline{
      border: 1px solid $c--primary;
      color: $c--primary;
      background-color: transparent;
      &:hover{
        border: 1px solid $c--secondary;
        color: $c--secondary;
        background-color: transparent;
      }
    }
    &--disabled{
      border: 1px solid #b0b0b0;
      background-color: $c--general-border;
      color: #b0b0b0;
      cursor: default;
      &:hover{
        border: 1px solid #b0b0b0;
        background-color: $c--general-border;
        color: #b0b0b0;
      }
    }
  }
</style>
