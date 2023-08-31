<template>
  <div class="app">
    <!-- <div class="">{{ selectedAnnotation }}</div> -->
    <ToolBar v-if="anno" ref="toolbar" :annotations="anno"></ToolBar>
    <div id="openseadragon" style="width: 80vw; height: 80vh"></div>
  </div>
</template>

<script>
import ToolBar from './components/ToolBar.vue'
import OpenSeadragon from 'openseadragon'
import Annotorious from '@recogito/annotorious-openseadragon'
import BetterPolygon from '@recogito/annotorious-better-polygon'

import '@recogito/annotorious-openseadragon/dist/annotorious.min.css'

export default {
  name: 'App',
  components: {
    ToolBar
  },
  data() {
    return {
      viewer: null,
      anno: null,
      selectedAnnotation: null
    }
  },
  mounted() {
    // this.init()
    const viewer = OpenSeadragon({
      id: 'openseadragon',
      showNavigationControl: false,
      tileSources: {
        type: 'image',
        url: `https://images.unsplash.com/photo-1683695948101-fd31df8697b5?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=2071&q=80`,
        fitBounds: true
      }
    })

    let options = {
      // disableEditor: true // the default editor is disabled to implement a custom behaviour
    }
    let anno = Annotorious(viewer, options)

    // let MyImportantFormatter = function (annotation) {
    //   const isImportant = annotation.bodies.find((b) => {
    //     return b.purpose === 'tagging' && b.value.toLowerCase() === 'important'
    //   })

    //   if (isImportant) {
    //     return 'important'
    //   }
    // }

    let MyLabelFormatter = function (annotation) {
      // Find the label body (if any)
      const label = annotation.bodies.find((b) => b.purpose === 'identifying')

      if (label) {
        // Return an HTML label, wrapped in an SVG foreignObject
        const foreignObject = document.createElementNS(
          'http://www.w3.org/2000/svg',
          'foreignObject'
        )
        foreignObject.innerHTML = `<label xmlns="http://www.w3.org/1999/xhtml" >${label.value}</label>`

        return {
          element: foreignObject
        }
      }
    }

    let MyClassFormatter = function (annotation) {
      const label = annotation.bodies.find((b) => b.purpose === 'identifying')

      if (label) {
        return label.value
      }
    }

    let Step3Formatter = function () {
      return 'step-3'
    }

    anno.formatters = [MyClassFormatter, MyLabelFormatter, Step3Formatter]

    // init plugins
    BetterPolygon(anno)

    anno.loadAnnotations(
      'https://raw.githubusercontent.com/annotorious/annotorious/main/public/annotations.w3c.json'
    )

    this.viewer = viewer
    this.anno = anno
    anno.setDrawingTool('rect')
    // anno.setDrawingEnabled(true)

    let me = this
    anno.on('clickAnnotation', function (annotation, element) {
      console.log('clickAnnotation', annotation, element)
      me.selectedAnnotation = annotation
      me.$refs['toolbar'].reset()
    })

    anno.on('createAnnotation', function (annotation, overrideId) {
      console.log('createAnnotation', annotation, overrideId)
      me.$refs['toolbar'].reset()
    })

    anno.on('createSelection', async function (selection) {
      selection.body = [
        {
          type: 'TextualBody',
          purpose: 'tagging',
          value: 'important'
        },
        {
          type: 'TextualBody',
          purpose: 'identifying',
          value: 'Lucy'
        }
      ]
      // Make sure to wait before saving!
      await anno.updateSelected(selection)
      anno.saveSelected()

      me.$refs['toolbar'].reset()

      console.log('createSelection', selection)

      // Or: anno.updateSelected(selection, true);
    })
  },

  methods: {}
}
</script>

<style>
:deep(.openseadragon-container :focus) {
  outline: none;
}

.a9s-annotation foreignObject {
  overflow: visible;
  width: 1px;
  height: 1px;
}

.a9s-annotation foreignObject label {
  display: inline-block;
  padding: 3px 8px;
}

.a9s-annotation.Lucy .a9s-inner {
  stroke: #00a4cb;
  fill: #00a4cb22;
}

.a9s-annotation.Emmet .a9s-inner {
  stroke: #ffa500;
  fill: #ffa50022;
}

.a9s-annotation.Batman .a9s-inner {
  stroke: #bb1dbb;
  fill: #bb1dbb22;
}

.a9s-annotation.step-3 label {
  transform: translateY(-100%) translateY(-1px) translateX(-1px);

  font-family: Arial, Helvetica, sans-serif;
  font-size: 14px;
  color: #fff;

  border-width: 2px 2px 0 2px;
  border-top-left-radius: 6px;
  border-top-right-radius: 6px;
}

.a9s-annotation.step-3.Lucy label {
  border-color: #00a4cb;
  background-color: #00a4cb;
}

.a9s-annotation.step-3.Emmet label {
  border-color: #ffa500;
  background-color: #ffa500;
}

.a9s-annotation.step-3.Batman label {
  border-color: #bb1dbb;
  background-color: #bb1dbb;
}

.a9s-annotation label {
  transform: translateY(-100%) translateX(-1px);
}

.a9s-annotation.important .a9s-inner {
  stroke-width: 2px;
  stroke: blue;
  fill: rgba(0, 0, 255, 0.1);
  -webkit-filter: drop-shadow(0 0 10px rgba(0, 0, 255, 0.8));
  filter: drop-shadow(0 0 10px rgba(0, 0, 255, 0.8));
}

.a9s-annotation.important .a9s-inner:hover {
  stroke-width: 6px;
  stroke: blue;
  fill: rgba(0, 0, 255, 0.3);
}
</style>