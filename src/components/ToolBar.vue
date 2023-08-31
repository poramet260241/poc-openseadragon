<template>
  <div class="tool-bar">
    <div class="tools">
      <button :class="{ active: selectedTool === 'rect' }" @click="setSelectedTool('rect')">
        Rect
      </button>
      <button :class="{ active: selectedTool === 'polygon' }" @click="setSelectedTool('polygon')">
        Polygon
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ToolBar',
  props: {
    annotations: Object
  },
  data() {
    return {
      selectedTool: ''
    }
  },
  methods: {
    setSelectedTool(id) {
      this.selectedTool = id
      this.annotations.setDrawingTool(id)
      this.annotations.setDrawingEnabled(true)
    },
    reset() {
      this.selectedTool = ''
      this.annotations.setDrawingEnabled(false)
    }
  }
}
</script>

<style scoped>
.tools {
  display: flex;
  gap: 0.5rem;
}
.tools > button {
  padding: 0.25rem 1rem;
  background-color: white;
  border: 1px solid;
  border-radius: 0.25rem;
}

.tools > .active {
  background-color: blue;
  color: white;
}
</style>