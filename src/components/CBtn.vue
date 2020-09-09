<template functional>
  <button
  v-bind="data.attrs"
  v-on="listeners">
      <slot/>
  </button>
</template>

<script>
export default {
  name: "CButton",
  props: {
    
  }
};
</script>

<style scoped lang="scss">
button{
    border: 1px solid #444;
    border-radius: 4px;;
    padding: 4px 8px;
    outline: none;
}
</style>
