<template>
  <div class="text-editor">
    <div class="controls-list"
    @mousedown.stop
    @mouseup.stop
    @click.stop
    @focus.stop>
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
    lastRanges: [],
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
      this.execCommand(elem => {
        elem.style.color = value;
      });
    },
    onBackgroundInput(value){
      this.execCommand(elem => {
        elem.style.backgroundColor = value;
      });
    },
    onFontSizeInput(e){
      this.execCommand(elem => {
        elem.style.fontSize = e.target.value;
      });
    },
    execCommand(editFunc){
      const editorInput = this.$refs.editorInput;
      let selection = document.getSelection();
      if(selection.toString() && !selection.isCollapsed){
        this.lastRanges = [];
        for(let i = 0; i < selection.rangeCount; ++i){
          this.lastRanges.push(selection.getRangeAt(i));
        }
      }
      for(let range of this.lastRanges){
        //clamp range
        if(editorInput.compareDocumentPosition(range.startContainer) & document.DOCUMENT_POSITION_PRECEDING){
          range.setStartBefore(editorInput.firstChild)
          console.log('preceding')
        }
        if(editorInput.compareDocumentPosition(range.endContainer) & (document.DOCUMENT_POSITION_FOLLOWING ^ ~document.DOCUMENT_POSITION_CONTAINED_BY)){
          range.setEndAfter(editorInput.lastChild)
          console.log('follow')
        }
        let startNode = range.startContainer;
        let startOffset = range.startOffset;
        let endNode = range.endContainer;
        let endOffset = range.endOffset;

        let node = startNode;
        let offset = startOffset;
        let isEnd = false;
        while(!isEnd){
          if(node == endNode){
            isEnd = true;
          }
          console.log('sibling', node);
          if(node.nodeType === Node.TEXT_NODE){
            let prevText, text, nextText;
            if(offset > 0){
              text = node.splitText(offset);
              prevText = node;
            }
            if(isEnd){
              if(offset > 0){
                nextText = text.splitText(endOffset - offset);
              }else{
                nextText = node.splitText(endOffset);
                text = node;
              }
            }
            let parent = node.parentElement;
            parent = (parent === editorInput) ? null : parent;
            if(parent){
              let parentParts = [];
              if(prevText){
                let prev = parent.cloneNode()
                prev.textContent = prevText.textContent;
                parentParts.push(prev);
              }
              if(text){
                let curr = parent.cloneNode()
                curr.textContent = text.textContent;
                editFunc(curr)
                parentParts.push(curr);
              }
              if(nextText){
                let next = parent.cloneNode()
                next.textContent = nextText.textContent;
                parentParts.push(next);
              }
              if(parentParts.length > 0) parent.replaceWith(...parentParts);
            }else{
              let editNode = document.createElement('span');
              editNode.textContent = text.textContent;
              editFunc(editNode);
              text.replaceWith(editNode);
            }
          }else if(node.nodeType === Node.ELEMENT_NODE){
            editFunc(node);
          }
          node = node.nextSibling;
          offset = 0;
        }

        // let rangeL = document.createRange();
        // rangeL.setStartBefore(editorInput.firstChild);
        // rangeL.setEnd(startNode, startOffset);
        // let fragmentL = rangeL.cloneContents();

        // let rangeR = document.createRange();
        // rangeR.setStart(endNode, endOffset);
        // rangeR.setEndAfter(editorInput.lastChild);
        // let fragmentR = rangeR.cloneContents();
        
        // let editFragment = range.extractContents();
        // console.log(editFragment)
        // editFragment.childNodes.forEach(node => {
        //   if(node.nodeType === Node.TEXT_NODE){
        //     if(node.textContent){
        //       let editNode = document.createElement('span');
        //       editNode.innerText = node.textContent;
        //       editFragment.replaceChild(editNode, node);
        //       let parent = editNode.parentElement;
        //       if(parent) editNode.style = parent.style;
        //       editFunc(editNode);
        //     }
        //   }else if(node.nodeType === Node.ELEMENT_NODE){
        //     if(node.nodeName.toLowerCase() === 'span'){
        //       editFunc(node);
        //     }
        //   }
        // })

        // editorInput.innerHTML = '';
        // console.log(fragmentL, editFragment, fragmentR);
        // appendFragment(fragmentL, editorInput);
        // appendFragment(editFragment, editorInput);
        // appendFragment(fragmentR, editorInput);
        // editorInput.appendChild(fragmentL);
        // editorInput.appendChild(editFragment);
        // editorInput.appendChild(fragmentR);
        // let editRange = document.createRange();
        // editRange.setStart(editorInput.firstChild, 0);
        // editRange.setEndAfter(editorInput.lastChild);
        // //editRange.selectNodeContents(editorInput);
        // console.log('edit range  ', editRange.cloneContents())
        // range = editRange;
        
        console.log(editorInput);

        
        
        function appendFragment(frag, parent){
          frag.childNodes.forEach(node => {
            if(node.textContent)
              parent.appendChild(node);
          })
        }
      }
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
            last.text = (last.text || '') + text;
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
