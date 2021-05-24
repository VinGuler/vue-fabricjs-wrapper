<template>
  <div id="app">
    <div class="actions">
      <button v-for="action in actions"
        :key="action"
        @click="setAction(action)">
        {{ action || 'CLEAR' }}
      </button>
    </div>
    <vue-fabricjs-wrapper
      v-if="draw"
      canvas-id="my-canvas"
      :canvas-width="480"
      :canvas-height="640"
      :object-name="current"
      @mouse:up="setAction(null)"
    />
  </div>
</template>

<script>
import VueFabricjsWrapper from '@/components/VueFabricjsWrapper'

export default {
  name: 'App',
  components: {
    VueFabricjsWrapper
  },
  data () {
    return {
      draw: false,
      actions: [
        'Circle',
        'Triangle',
        'Rect',
        'Ellipse',
        // 'Line',
        // 'Polygon',
        // 'Polyline',
        'DELETE-ALL',
        null
      ],
      current: null
    }
  },
  methods: {
    setAction(action) {
      if (action === 'DELETE-ALL') {
        return
      }
      this.current = action
    }
  },
  mounted () {
    this.draw = true
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
.canvas-container {
  margin: auto;
  
}
.actions {
  margin: 12px auto 12px auto;
  width: 640px;
  display: flex;
  justify-content: space-between;
}
button {
  cursor: pointer;
  padding: 8px;
}
</style>
