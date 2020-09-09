<template>
  <div class="controlsList">
    <color-input v-slot="{on}"
    @input="onColorInput">
      <c-btn v-on="on">Color</c-btn>
    </color-input>
    <color-input v-slot="{on}"
    @input="onBackgroundInput">
      <c-btn v-on="on">Background</c-btn>
    </color-input>
    <span>
      <text-input type="number"/>
      <c-btn></c-btn>
    </span>
  </div>
</template>

<script>
import CBtn from './CBtn'
import ColorInput from './ColorInput'
import TextInput from './TextInput'

export default {
  name: "EditorControls",
  components: {
    CBtn,
    ColorInput,
    TextInput
  },
  methods: {
    onColorInput(value){
      this.execCommand('foreColor', value);
    },
    onBackgroundInput(value){
      this.execCommand('backColor', value);
    },
    execCommand(command, value){
      document.execCommand(command, false, value)
    }
  }
};
</script>

<style scoped lang="scss">
.controlsList{
  display: flex;
  flex-flow: row nowrap;
  >*:not(:last-child){
    margin-right: 5px;
  }
}
</style>
