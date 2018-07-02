<template>
<div style="width: 60%;">
    <diagram ref='diag' :model-data='diagramData' @model-changed='modelChanged' @changed-selection='changedSelection' style='border: solid 1px black; width:100%; height:200px'></diagram>
    <button @click='addNode'>Add Child to Gamma</button>
    <button @click='modifyStuff'>Modify view model data without undo</button>
    <br/>Current Node:
    <input v-model.lazy='currentNodeText' :disabled='currentNode === null'/>
    <br/>The saved GoJS Model:
    <!--<textarea style='width:100%;height:250px'>{{ savedModelText }}</textarea>-->
    <textarea style='width:100%;height:200px' v-model="savedModelText"></textarea>
</div>
</template>

<script>
import diagram from '../components/GoDiagram'

export default {
    name: '',
    components: {
        diagram
    },
    data () {
        return {
            diagramData: { // passed to <diagram> as its modelData
                nodeDataArray: [
                    { key: 1, text: 'Alpha', color: 'lightblue' },
                    { key: 2, text: 'Beta', color: 'orange' },
                    { key: 3, text: 'Gamma', color: 'lightgreen' },
                    { key: 4, text: 'Delta', color: 'pink' }
                ],
                linkDataArray: [
                    { from: 1, to: 2 },
                    { from: 1, to: 3 },
                    { from: 3, to: 4 }
                ]
            },
            currentNode: null,
            savedModelText: '',
            counter: 1, // used by addNode
            counter2: 4 // used by modifyStuff
        }
    },
    mounted () {

    },
    computed: {
        currentNodeText: {
            get: function () {
                let node = this.currentNode
                if (node instanceof window.go.Node) {
                    return node.data.text
                } else {
                    return ''
                }
            },
            set: function (val) {
                let node = this.currentNode
                if (node instanceof window.go.Node) {
                    let model = this.model()
                    model.startTransaction()
                    model.setDataProperty(node.data, 'text', val)
                    model.commitTransaction('edited text')
                }
            }
        }
    },
    methods: {
        // get access to the GoJS Model of the GoJS Diagram
        model: function () {
            return this.$refs.diag.model()
        },
        // tell the GoJS Diagram to update based on the arbitrarily modified model data
        updateDiagramFromData: function () {
            this.$refs.diag.updateDiagramFromData()
        },
        // this event listener is declared on the <diagram>
        modelChanged: function (e) {
            if (e.isTransactionFinished) { // show the model data in the page's TextArea
                this.savedModelText = e.model.toJson()
            }
        },
        changedSelection: function (e) {
            let node = e.diagram.selection.first()
            if (node instanceof window.go.Node) {
                this.currentNode = node
                this.currentNodeText = node.data.text
            } else {
                this.currentNode = null
                this.currentNodeText = ''
            }
        },
        // Here we modify the GoJS Diagram's Model using its methods,
        // which can be much more efficient than modifying some memory and asking
        // the GoJS Diagram to find differences and update accordingly.
        // Undo and Redo will work as expected.
        addNode: function () {
            let model = this.model()
            model.startTransaction()
            model.setDataProperty(model.findNodeDataForKey(4), 'color', 'purple')
            let data = { text: 'NEW ' + this.counter++, color: 'yellow' }
            model.addNodeData(data)
            model.addLinkData({ from: 3, to: model.getKeyForNodeData(data) })
            model.commitTransaction('added Node and Link')
            // also manipulate the Diagram by changing its Diagram.selection collection
            let diagram = this.$refs.diag.diagram
            diagram.select(diagram.findNodeForData(data))
        },
        // Here we modify VUE's view model directly, and
        // then ask the GoJS Diagram to update everything from the data.
        // This is less efficient than calling the appropriate GoJS Model methods.
        // NOTE: Undo will not be able to restore all of the state properly!!
        modifyStuff: function () {
            let data = this.diagramData
            data.nodeDataArray[0].color = 'red'
            // Note here that because we do not have the GoJS Model,
            // we cannot find out what values would be unique keys, for reference by the link data.
            data.nodeDataArray.push({ key: ++this.counter2, text: this.counter2.toString(), color: 'orange' })
            data.linkDataArray.push({ from: 2, to: this.counter2 })
            this.updateDiagramFromData()
        }
    }
}
</script>

<style>

</style>
