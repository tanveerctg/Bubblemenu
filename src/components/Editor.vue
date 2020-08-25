<template>
  <div>
    <div id="editor" v-on:keyup="keyUpHandler"></div>
  	<div id="content"></div>
  </div>
</template>

<script>

import {EditorState} from "prosemirror-state"
import {EditorView} from "prosemirror-view"
import {Schema, DOMParser} from "prosemirror-model"
import {schema} from "prosemirror-schema-basic"
import {addListNodes} from "prosemirror-schema-list"
import {exampleSetup} from "prosemirror-example-setup"
import {Plugin} from "prosemirror-state"
import {toggleMark, setBlockType, wrapIn} from "prosemirror-commands"


	export default {
	  name: 'Editor',
	  props: ['msg'],
	  data:function(){
	      return{
	          view:null
	      }
	  },
	  methods:{
	  	keyUpHandler(){
	  		console.log(JSON.stringify(this.view.state.tr.doc))
	  	}
	  },
	  mounted(){
		const mySchema = new Schema({
		  nodes: addListNodes(schema.spec.nodes, "paragraph block*", "block"),
		  marks: schema.spec.marks
		})



	class FloatingToolMenu {
	  constructor(view,items) {
	  	this.items = items
	  	this.view=view
	    this.dom = document.createElement("div")
	    this.dom.className = "menubar"

	    this.items.forEach(({dom}) => this.dom.appendChild(dom))

	    view.dom.parentNode.appendChild(this.dom)

	  

	    this.update(view, null)

	    this.dom.addEventListener("mousedown", e => {
	         e.preventDefault()
	      view.focus()
	      items.forEach(({command, dom}) => {
	        if (dom.contains(e.target))
	          command(view.state, view.dispatch, view)
	      })
	    })

	  }

	  update(view, lastState){

	    this.items.forEach(({command, dom}) => {
	      let active = command(this.view.state, null, this.view)
	    
	      dom.style.display = active ? "" : "none"
	    })
	    let state = view.state
	    // Don't do anything if the document/selection didn't change
	    if (lastState && lastState.doc.eq(state.doc) &&
	        lastState.selection.eq(state.selection)) return

	    // Hide the tooltip if the selection is empty
	    if (state.selection.empty) {
	      this.dom.style.display = "none"
	      return
	    }

	    // Otherwise, reposition it and update its content
	    this.dom.style.display = ""
	    let {from, to} = state.selection
	    // These are in screen coordinates
	    let start = view.coordsAtPos(from), end = view.coordsAtPos(to)
	    // The box in which the tooltip is positioned, to use as base
	    let box = this.dom.offsetParent.getBoundingClientRect()
	    // Find a center-ish x position from the selection endpoints (when
	    // crossing lines, end may be more to the left)
	    let left = Math.max((start.left + end.left) / 2, start.left + 3)
	    this.dom.style.left = (left - box.left) + "px"
	    this.dom.style.bottom = (box.bottom - start.top) + "px"

	  
	  }

	  destroy() { 
	  	this.dom.remove()
	  }
	}




	// Helper function to create menu icons
	function icon(text, name) {
	  let span = document.createElement("span")
	  span.className = "menuicon " + name
	  span.title = name
	  span.textContent = text
	  return span
	}

	// Create an icon for a heading at the given level
	function heading(level) {
	  return {
	    command: setBlockType(mySchema.nodes.heading, {level}),
	    dom: icon("H" + level, "heading")
	  }
	}
	let allCommands=[
	  {command: toggleMark(mySchema.marks.strong), dom: icon("B", "strong")},
	  {command: toggleMark(mySchema.marks.em), dom: icon("i", "em")},
	  {command: setBlockType(mySchema.nodes.paragraph), dom: icon("p", "paragraph")},
	  heading(1), heading(2), heading(3),
	  {command: wrapIn(mySchema.nodes.blockquote), dom: icon(">", "blockquote")}
	]


	let floatingMenuPlugin = new Plugin({
	  view(editorView) { return new FloatingToolMenu(editorView,allCommands) }
	})

	let view  = new EditorView(document.querySelector("#editor"), {
	  state: EditorState.create({
	    doc: DOMParser.fromSchema(mySchema).parse(document.querySelector("#content")),
	    plugins: exampleSetup({schema: mySchema}).concat(floatingMenuPlugin)
	  })
	})

	this.view=view;

	let transaction=this.view.state.tr;
  	 	
  	transaction.insertText(JSON.parse(this.msg).greeting)
    let newState=this.view.state.apply(transaction)
  	this.view.updateState(newState)

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
