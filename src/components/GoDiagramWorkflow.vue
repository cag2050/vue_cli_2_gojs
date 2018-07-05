<template>
    <div style='width:100%; white-space:nowrap;'>
        <span style='border: 1px solid gray;display: inline-block; vertical-align: top; width:120px;'>
            <div ref='myPaletteDiv' style='height: 210px;'>1111</div>
        </span>
        <span style='border: 1px solid gray;display: inline-block; vertical-align: top; width:40%;'>
            <div ref='myDiagramDiv' style='height: 210px'></div>
        </span>
    </div>
</template>

<script>
let go = window.go
let $ = go.GraphObject.make

export default {
    name: '',
    props: ['modelData'],
    data () {
        return {
            diagram: null
        }
    },
    mounted () {
        let self = this
        let myDiagram =
            $(go.Diagram, this.$refs.myDiagramDiv,
                {
                    initialContentAlignment: go.Spot.Center,
                    // layout: $(go.TreeLayout, {angle: 90, arrangement: go.TreeLayout.ArrangementHorizontal}),
                    'undoManager.isEnabled': true,
                    // Model ChangedEvents get passed up to component users
                    'ModelChanged': function (e) {
                        self.$emit('model-changed', e)
                    },
                    'ChangedSelection': function (e) {
                        self.$emit('changed-selection', e)
                    },
                    'Modified': function (e) {
                        self.$emit('modified', e)
                    },
                    'TextEdited': function (e) {
                        self.$emit('text-edited', e)
                    },
                    allowDrop: true
                })
        myDiagram.nodeTemplateMap.add('',
            $(go.Node, 'Spot', this.nodeStyle(),
                $(go.Panel, 'Auto',
                    $(go.Shape, 'Rectangle',
                        {fill: '#00A9C9', stroke: null},
                        new go.Binding('figure', 'figure')),
                    $(go.TextBlock, 'command',
                        {
                            font: 'bold 11pt Helvetica, Arial, sans-serif',
                            stroke: 'whitesmoke',
                            margin: 8,
                            maxSize: new go.Size(160, NaN),
                            wrap: go.TextBlock.WrapFit,
                            editable: true
                        },
                        new go.Binding('text').makeTwoWay())
                ),
                // four named ports, one on each side:
                this.makePort('T', go.Spot.Top, false, true),
                this.makePort('L', go.Spot.Left, true, true),
                this.makePort('R', go.Spot.Right, true, true),
                this.makePort('B', go.Spot.Bottom, true, false)
            ))
        myDiagram.linkTemplate =
            $(go.Link,
                $(go.TextBlock, // this is a Link label
                    new go.Binding('text', 'text')),
                // the whole link panel
                {
                    routing: go.Link.AvoidsNodes,
                    curve: go.Link.JumpOver,
                    corner: 5,
                    toShortLength: 4,
                    relinkableFrom: true,
                    relinkableTo: true,
                    reshapable: true,
                    resegmentable: true,
                    // mouse-overs subtly highlight links:
                    mouseEnter: function (e, link) {
                        link.findObject('HIGHLIGHT').stroke = 'rgba(30,144,255,0.2)'
                    },
                    mouseLeave: function (e, link) {
                        link.findObject('HIGHLIGHT').stroke = 'transparent'
                    }
                },
                new go.Binding('points').makeTwoWay(),
                $(go.Shape, // the highlight shape, normally transparent
                    {isPanelMain: true, strokeWidth: 8, stroke: 'transparent', name: 'HIGHLIGHT'}),
                $(go.Shape, // the link path shape
                    {isPanelMain: true, stroke: 'gray', strokeWidth: 2}),
                $(go.Shape, // the arrowhead
                    {toArrow: 'standard', stroke: null, fill: 'gray'}),
                $(go.Panel, 'Auto', // the link label, normally not visible
                    {visible: true, name: 'LABEL', segmentIndex: 2, segmentFraction: 0.5},
                    new go.Binding('visible', 'visible').makeTwoWay(),
                    $(go.Shape, // the label shape
                        {fill: $(go.Brush, 'Radial', { 0: 'rgb(240, 240, 240)', 0.3: 'rgb(240, 240, 240)', 1: 'rgba(240, 240, 240, 0)' }), stroke: null}),
                    $(go.TextBlock, 'Yes', // the label
                        {
                            textAlign: 'center',
                            font: '10pt helvetica, arial, sans-serif',
                            stroke: '#333333',
                            editable: false
                        },
                        new go.Binding('text').makeTwoWay())
                )
            )
        let myPalette =
            $(go.Palette, this.$refs.myPaletteDiv, // must name or refer to the DIV HTML element
                {
                    'animationManager.duration': 800, // slightly longer than default (600ms) animation
                    nodeTemplateMap: myDiagram.nodeTemplateMap, // share the templates used by myDiagram
                    // nodeTemplate: myDiagram.nodeTemplate, // share the templates used by myDiagram
                    model: new go.GraphLinksModel([ // specify the contents of the Palette
                        {category: 'Start', text: 'Start'},
                        {category: 'Command', text: 'Command'},
                        {category: 'End', text: 'End'}
                    ])
                })
        console.log(myPalette)
        this.diagram = myDiagram
        this.updateModel(this.modelData)
    },
    watch: {
        modelData: function (val) {
            console.log('watch')
            console.log(val)
            this.updateModel(val)
        }
    },
    computed: {},
    methods: {
        makePort (name, spot, output, input) {
            return $(go.Shape, 'Circle',
                {
                    fill: 'transparent',
                    stroke: null, // this is changed to 'white' in the showPorts function
                    desiredSize: new go.Size(8, 8),
                    alignment: spot,
                    alignmentFocus: spot, // align the port on the main Shape
                    portId: name, // declare this object to be a 'port'
                    fromSpot: spot,
                    toSpot: spot, // declare where links may connect at this port
                    fromLinkable: output,
                    toLinkable: input, // declare whether the user may draw links to/from here
                    cursor: 'pointer' // show a different cursor to indicate potential link point
                })
        },
        nodeStyle () {
            return [
                new go.Binding('location', 'loc', go.Point.parse).makeTwoWay(go.Point.stringify),
                {
                    locationSpot: go.Spot.Center,
                    mouseEnter: (e, obj) => {
                        this.showPorts(obj.part, true)
                    },
                    mouseLeave: (e, obj) => {
                        this.showPorts(obj.part, false)
                    }
                }
            ]
        },
        showPorts (node, show) {
            let diagram = node.diagram
            if (!diagram || diagram.isReadOnly || !diagram.allowLink) return
            node.ports.each(function (port) {
                port.stroke = (show ? 'white' : null)
            })
        },
        model: function () {
            return this.diagram.model
        },
        updateModel: function (val) {
            // No GoJS transaction permitted when replacing Diagram.model.
            if (val instanceof go.Model) {
                this.diagram.model = val
            } else {
                let m = new go.GraphLinksModel()
                if (val) {
                    for (let p in val) {
                        m[p] = val[p]
                    }
                }
                this.diagram.model = m
            }
        },
        updateDiagramFromData: function () {
            this.diagram.startTransaction()
            // This is very general but very inefficient.
            // It would be better to modify the diagramData data by calling
            // Model.setDataProperty or Model.addNodeData, et al.
            this.diagram.updateAllRelationshipsFromData()
            this.diagram.updateAllTargetBindings()
            this.diagram.commitTransaction('updated')
        }
    }
}
</script>

<style>

</style>
