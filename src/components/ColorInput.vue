<template>
  <div class="color-picker">
    <input ref="colorInput" type="color"
    @input="onInput"
    @change="onChange" />
    <slot class="activator" :on="{click: open}"></slot>
  </div>
</template>

<script>

export default {
  name: "ColorPicker",
  props: {
    value: String
  },
  methods: {
    open(){
      this.$refs.colorInput.click();
    },
    onInput(e){
      this.$emit('input', e.target.value);
    },
    onChange(e){
      this.$emit('change', e.target.value);
    }
  }
};
</script>

<style scoped lang="scss">
.color-picker{
  position: relative;
  overflow: hidden;
}
.activator{
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 2;
}
input[type=color]{
  visibility: hidden;
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1;
}
</style>
