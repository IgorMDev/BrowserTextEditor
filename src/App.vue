<template>
  <div id="app">
    <div class="editor-wrapper mx-auto">
      <h3>Text editor</h3>
      <text-editor
      @convert="onConverted" />
    </div>

    <div style="padding: 20px 0; text-align: left">
      <pre>
        {{text}}
      </pre>
    </div>
  </div>
</template>

<script>
import TextEditor from './components/TextEditor'

export default {
  name: "App",
  components: {
    TextEditor
  },
  data: () => ({
    text: ''
  }),
  methods: {
    onConverted(data){
      this.text = data;
    }
  }
};
</script>

<style lang="scss">
*, *:before, *:after {
  box-sizing: border-box;
}
.mx-auto{
  margin-left: auto;
  margin-right: auto;
}
.border{
  border: 1px solid #333;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.editor-wrapper{
  width: 60%;
}
</style>
