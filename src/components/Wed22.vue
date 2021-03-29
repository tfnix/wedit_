<template>
  <div class="wed22">

    <div id="title">
      <!-- <h2 >Wed22 Editor</h2>
      <span>for learning purposes</span> -->
    </div>
    
    <div id="toolbar-container">
      <div id="toolbar">
        <span class="emoji" @click="insertTextAtCaret('😍')">😍</span>
        <span class="emoji" @click="insertTextAtCaret('😎')">😎</span>
        <span class="emoji" @click="insertTextAtCaret('🙂')">🙂</span>
        <span v-if="!isVisible" class="arrow-emoji" @click="toogleToolbar()">🙂</span>
      </div>
    </div>
    <div id="editor" contenteditable="true"></div>
    <em style="margin-top: 20px">List of Tags (WIP)</em>


  </div>
</template>

<script>
export default {
  data() {
    return {
      rangeSelection : null,
      savedSel : null,
      keyTimer : null ,
      isVisible: false,
    }
  },
  mounted() {
    const editor = this.getEditor()
    editor.addEventListener('keydown', this.handleKeydown)
    editor.addEventListener('mouseleave', this.handleMouseLeave)
    editor.addEventListener('dragstart', function(event){
      event.preventDefault();
      return false;
    });
  },
  methods: {


    getEditor() {
      return document.getElementById('editor')
    },       
    clearEmpty() {
      var wrapper = this.getEditor();
      var ps = wrapper.querySelectorAll('.tag');
      for (var p = 0; p < ps.length; p++) {
        if (ps[p].innerText == '') {
          ps[p].parentNode.removeChild(ps[p]);
        }
      }    
    }, 
    toogleToolbar() {     
      var el = document.getElementById("toolbar")
      if(!this.isVisible) {        
        el.style.transform = "translateX(0px)";        
      } else {        
        el.style.transform = "translateX(-94px)";
      }
      this.isVisible = !this.isVisible
    },
    randomColors() {
      var colors = [
                'rgb(22, 96, 138)',
                'rgb(22, 138, 52)',
                'rgb(163, 48, 48)' ]
      return colors[Math.floor(Math.random() * colors.length)]
    },
  
    insertTextAtCaret(text) {
      
      // TODO: NEED TO FIX 
      // var sel, range;
      // this.restoreSelection2(this.rangeSelection);
      // if (window.getSelection) {
      //     sel = window.getSelection();        
      //     if (sel.getRangeAt && sel.rangeCount) {
      //         //  console.log(sel.anchorNode.contentEditable)
      //         // if (!sel.anchorNode.contentEditable) {
      //         //   console.log('error')
      //         //   return
      //         //   //TODO: FIX THIS for other browser brands
      //         // }
      //         range = sel.getRangeAt(0);
      //         //range.deleteContents();
              
      //         range.insertNode( document.createTextNode(text) );
      //         range.collapse(false)
      //     }
      // } else if (document.selection && document.selection.createRange) {
      //     document.selection.createRange().text = text;
      // }
      
      this.toogleToolbar();
    },      
    
    // Handling Events... 

    handleMouseLeave(){
      this.saveSelection2();      
    }, 
    handleKeydown(e) {
      var self=this;      
      if (e.keyCode == 39 ) {  
       var node = window.getSelection().anchorNode
       if (node.parentNode.getAttribute('data') == 'tag') {         
         var position = window.getSelection().getRangeAt(0).startOffset;        
         if (position == node.nodeValue.length) {
           var space = document.createTextNode("\xA0");
           node.parentNode.parentNode.insertBefore (space, node.parentNode.nextSibling)
         }
       }
      }
      if (e.keyCode == 46) {  this.clearEmpty() } 
      if (e.keyCode == 8 ) {  this.clearEmpty() } 
      
      clearTimeout(this.keyTimer);      

      if (e.keyCode == 32){                
        this.keyTimer = setTimeout( function() {           
          self.getEditor().focus();          
          self.reFormat()            
          self.keyTimer = null;
        }, 10)
      }

      if (e.keyCode === 9) { 
          e.preventDefault();           
          var sel = window.getSelection();
          var range = sel.getRangeAt(0);
          var tabNode = document.createTextNode("\u00a0\u00a0\u00a0\u00a0");
          range.insertNode(tabNode);
          range.setStartAfter(tabNode);
          range.setEndAfter(tabNode); 
          sel.removeAllRanges();
          sel.addRange(range);
      }      

    },
    reFormat() {       
      var self = this;
      const doTreeWalk = (element) => {
        var nodeList = element.childNodes;        
        if (nodeList != null) {
          if (nodeList.length) {                  
            for (var i = 0; i < nodeList.length; i++) {              
              if (nodeList[i].nodeType == 3) { 
                var string = nodeList[i].nodeValue;       
                const regex = /(^#|#)([a-z0-9]+)/gi
                const found = string.match(regex);                 
                var attr = nodeList[i].parentNode.getAttribute('data')        
                if (found != null && attr != 'tag') {
                    self.savedSel = self.saveSelection(self.getEditor())                    
                    var node; 
                    if (nodeList[i].parentNode.contentEditable) {
                       node = nodeList[i];
                     } else {
                       node = nodeList[i].parentNode
                     }                  
                    // create frag
                    const tpl = document.createElement('template');
                    const buff = '<span class="tag" data="tag">'+ found[0] +'</span>'
                    tpl.innerHTML = string.replace(regex,buff );
                    // TODO: Need fix multiple matches
                    const fragment = tpl.content;
                    node.replaceWith(fragment)                                     
                    self.restoreSelection(self.getEditor(), self.savedSel)
                    return;
                }                
              }
              else {
                doTreeWalk(nodeList[i]);
              }
            }
          }
        }
      }
      
      const editor = this.getEditor()
      doTreeWalk(editor);     
      editor.focus()
      
    },

    // TimDown @ SO (add here)
    saveSelection(containerEl) {
      var range = window.getSelection().getRangeAt(0);
      var preSelectionRange = range.cloneRange();
      preSelectionRange.selectNodeContents(containerEl);
      preSelectionRange.setEnd(range.startContainer, range.startOffset);
      var start = preSelectionRange.toString().length;
      return {
          start: start,
          end: start + range.toString().length
      };

    },
    restoreSelection(containerEl, savedSel) {
      var charIndex = 0, range = document.createRange();
      range.setStart(containerEl, 0);
      range.collapse(true);
      var nodeStack = [containerEl], node, foundStart = false, stop = false;
      
      while (!stop && (node = nodeStack.pop())) {
        if (node.nodeType == 3) {
          var nextCharIndex = charIndex + node.length;
          if (!foundStart && savedSel.start >= charIndex && savedSel.start <= nextCharIndex) {
              range.setStart(node, savedSel.start - charIndex);
              foundStart = true;
          }
          if (foundStart && savedSel.end >= charIndex && savedSel.end <= nextCharIndex) {
              range.setEnd(node, savedSel.end - charIndex );
              stop = true;
          }
          charIndex = nextCharIndex;
        } else {
          var i = node.childNodes.length;
          while (i--) {
              nodeStack.push(node.childNodes[i]);
          }
        }
      }
      var sel = window.getSelection();
      sel.removeAllRanges();
      sel.addRange(range);     
    },       
    saveSelection2() {
      if (window.getSelection) {              
        var sel = window.getSelection();
        if (sel.getRangeAt && sel.rangeCount) {
          this.rangeSelection = sel.getRangeAt(0);
        }
      } else if (document.selection && document.selection.createRange) {
          this.rangeSelection = document.selection.createRange();
      }

    },
    restoreSelection2(range) {
      if (range) {
        if (window.getSelection) {
            var sel = window.getSelection();
            sel.removeAllRanges();
            sel.addRange(range);
        } else if (document.selection && range.select) {
            range.select();
        }
      }      

    },         

  }
  

}
</script>
<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Source+Sans+Pro:wght@200&display=swap');

* {  font-family: 'Source Sans Pro', sans-serif; }
.wed22 {
  display: flex;
  margin: 0 auto;
  width: 70%;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}
#title {
  display: flex;
  flex-direction: column;
  justify-self: flex-start;
  align-self: flex-start;
  
  text-align: left;
  h2 {
    padding: 0;
    margin: 0;    
  }
  span {
    color: darkred; 
    font-size: 1em; 
    text-decoration: underline      
  }
}
#toolbar-container {
  width: 130px;
  align-self: flex-start;
  overflow-x: hidden;
}
#toolbar {
  transform: translateX(-95px);
  justify-self: flex-start;
  align-self: flex-start;
  border-radius: 4px;
  opacity: 1;
  transition: .10s all;
  display: flex;
  border-radius: 5px;
  background-color: white;
  span {
    font-size: 20px;
    width: 56px;
    height: 36px;
    color: white;
    &:hover {      
      background-color: rgb(230, 230, 230);
      cursor: pointer
    }
  }  
}
.emoji {  
   margin: 10px 0;
   font-size: 50px;
   justify-self: flex-start;
   align-self: flex-start;
   display:flex;
   justify-content: center;
   align-items: center;
}
.arrow-emoji {     
   margin: 10px 0;
   font-size: 10px;
   color: black;   
   display:flex;
   justify-content: center;
   align-items: center;
}


.tag {
  color: white;
  cursor: pointer;
  font-size: .8em;
  background-color: rgb(22, 96, 138);
  border-radius: 5px;
  padding: 0 15px;
  margin: 0;
}
#editor { 
  display: block;  
  white-space: pre-wrap;
  caret-color: red;
  font-size: 2.2em;
  font-weight: bold;
  margin: 0 auto;  
  width: 100%;
  height: 300px;  
  color: rgb(53, 53, 53);
  border-radius: 10px;
  outline: none;
  text-align: left;  
  overflow-y: auto;
  border: 2px solid rgb(188, 194, 173);  
}

</style>
