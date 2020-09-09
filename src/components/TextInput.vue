<template functional>
  <input
  v-bind="data.attrs"
  v-on="listeners"/>
</template>

<script>
export default {
  name: "TextInput",
  props: {
    
  }
};
</script>

<style scoped lang="scss">
input{
    border: 1px solid rgb(87, 87, 87);
    border-radius: 4px;
    padding: 4px 8px;
}
</style>
