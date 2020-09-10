<template>
  <div class="text-editor">
    <div class="controls-list">
      <color-input v-slot="{on}"
      @input="onColorInput">
        <c-btn v-on="on">Color</c-btn>
      </color-input>
      <color-input v-slot="{on}"
      @input="onBackgroundInput">
        <c-btn v-on="on">Background</c-btn>
      </color-input>
      <select
      @change="onFontSizeInput">
        <option disabled selected>Font size</option>
        <option v-for="(item, i) in fontSizeOptions" :key="i"
        :value="item+'px'">
          {{item+' px'}}
        </option>
      </select>
      <i>Select text to format</i>
    </div>
    <div class="editor-input" contenteditable
    ref="editorInput"
    @keydown.enter.prevent="onEnter"
    @input="onInput">
      
    </div>
    <div class="editor-footer">
      <c-btn
      @click="convertToJSON">Convert to JSON</c-btn>
    </div>
  </div>
</template>

<script>
import CBtn from './CBtn'
import ColorInput from './ColorInput'
import TextInput from './TextInput'

/* eslint-disable */
function execCommand(container, editFunc){;
  let selection = document.getSelection();
  if(selection.toString() && !selection.isCollapsed && selection.containsNode(container, true))
  for(let i = 0; i < selection.rangeCount; ++i){
    let range = selection.getRangeAt(i).cloneRange();
    //clamp range
    if(container.compareDocumentPosition(range.startContainer) & document.DOCUMENT_POSITION_PRECEDING){
      range.setStartBefore(container.firstChild)
    }
    if((container.compareDocumentPosition(range.endContainer) & document.DOCUMENT_POSITION_FOLLOWING) && !container.contains(range.endContainer)){
      range.setEndAfter(container.lastChild)
    }
    let startNode = range.startContainer;
    let startOffset = range.startOffset;
    let endNode = range.endContainer;
    let endOffset = range.endOffset;

    if(startNode == endNode){
      splitTextNode(startNode, startOffset, endOffset);
    }else{
      let start = splitTextNode(startNode, startOffset);
      let end = splitTextNode(endNode, 0, endOffset);
      let node = start.nextSibling;
      while(node && node != end){
        node = splitTextNode(node, 0);
        node = node?.nextSibling;
      }
    }
    function splitTextNode(node, startOffset, endOffset){
      if(node.nodeType === Node.TEXT_NODE){
        let prevText, text = node, nextText;
        if(startOffset > 0){
          text = node.splitText(startOffset);
          prevText = node;
        }
        if(endOffset){
          nextText = text.splitText(endOffset - startOffset);
        }
        let parent = node.parentElement;
        parent = (parent == container) ? null : parent;
        if(parent){
          let parentParts = [];
          if(prevText && prevText.textContent){
            let prev = parent.cloneNode()
            prev.appendChild(prevText);
            parentParts.push(prev);
          }
          if(text && text.textContent){
            let curr = parent.cloneNode()
            text = curr.appendChild(text);
            editFunc(curr);
            parentParts.push(curr);
          }
          if(nextText && nextText.textContent){
            let next = parent.cloneNode()
            next.appendChild(nextText);
            parentParts.push(next);
          }
          if(parentParts.length > 0) parent.replaceWith(...parentParts);
          return text.parentElement;
        }else if(text.textContent){
          let editNode = document.createElement('span');
          editNode.appendChild(text.cloneNode(true));
          editFunc(editNode);
          text.replaceWith(editNode);
          return editNode;
        }
        return text;
      }else if(node.nodeType === Node.ELEMENT_NODE){
        let child = node.firstChild;
        if(child){
          return splitTextNode(child);
        }
      }
      return node;
    }
  }
}
export default {
  name: "TextEditor",
  components: {
    CBtn,
    ColorInput,
    TextInput
  },
  props: {
    value: String
  },
  data: () => ({
    fontSizeOptions: [2,4,6,8,10,12,14,16,18,20,22,24,26,28,30]
  }),
  methods: {
    onEnter(e){
      document.execCommand('insertHTML', false, '<br><br>');
    },
    onInput(e){
      this.$emit('input', e.target.innerHTML);
    },
    onColorInput(value){
      execCommand(this.$refs.editorInput, elem => {
        elem.style.color = value;
      });
    },
    onBackgroundInput(value){
      execCommand(this.$refs.editorInput, elem => {
        elem.style.backgroundColor = value;
      });
    },
    onFontSizeInput(e){
      execCommand(this.$refs.editorInput, elem => {
        elem.style.fontSize = e.target.value;
      });
    },
    convertToJSON(){
      const editorInput = this.$refs.editorInput;
      let res = [];
      let last = {};
      editorInput.childNodes.forEach(node => {
        let styles = {};
        let text = '';
        if(node.nodeType === Node.TEXT_NODE){
          let el = node.parentElement;
          text = node.textContent;
          styles = {
            color: el.style.color,
            backgroundColor: el.style.backgroundColor,
            fontSize: el.style.fontSize,
          };
            
        }else if(node.nodeType === Node.ELEMENT_NODE){
          text = node.textContent;
          styles = {
            color: node.style.color,
            backgroundColor: node.style.backgroundColor,
            fontSize: node.style.fontSize,
          };
        }
        if(text){
          if(last.color == styles.color
            && last.backgroundColor == styles.backgroundColor 
            && last.fontSize == styles.fontSize
          ){
            last.text = (last.text || '') + ' '+text;
          }else{
            last = {
              text,
              ...styles
            }
            res.push(last);
          }
        }
      })
      this.$emit('convert', JSON.stringify(res, (key, val) => !val ? undefined : val, 4));
    }
  }
};
</script>

<style scoped lang="scss">
.text-editor{
  display: flex;
  flex-flow: column nowrap;
  padding: 10px;
  border: 1px solid #888;
  border-radius: 10px;
}
.editor{
  &-input{
    background-color: rgba($color: #ccc, $alpha: 0.3);
    margin: 20px 0;
    min-height: 1em;
    padding: 8px;
    text-align: left;
    border: 1px dashed #ccc;
    outline: none;
  }
  &-footer{
    display: flex;
    justify-content: flex-end;
    align-items: center;
  }
}
.controls-list{
  display: flex;
  flex-flow: row nowrap;
  >*:not(:last-child){
    margin-right: 10px;
  }
  select{
    padding: 0px 4px;
    border-radius: 4px;
  }
}
</style>
