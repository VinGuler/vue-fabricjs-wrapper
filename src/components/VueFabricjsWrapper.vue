<script>
  import { fabric } from 'fabric'

  const toCamelCase = (str) => {
    if (!str || typeof str !== 'string') return str
    const chars = str.split('').map((char) => char.toLowerCase())
    chars[0] = chars[0].toUpperCase()
    return chars.join('')
  }

  export default {
    name: 'VueFabricjsWrapper',
    props: {
      canvasId: String,
      canvasWidth: {
        type: Number,
        default: 360
      },
      canvasHeight: {
        type: Number,
        default: 640
      },
      objectName: String,
      fill: {
        type: String,
        default: '#FF6622'
      }
    },
    data() {
      return {
        canvas: null,
        canvasAttributes: {
          id: 'fabric-canvas',
          width: 360,
          height: 640
        },
        canvasStyle: {
          border: '1px solid black'
        },
        originalPointer: { x: 0, y: 0 },
        current: null
      }
    },
    methods: {
      // UTILITY FUNCTIONS
      // Making sure the object is inside the canvas
      handleObjectPositionInCanvas({ x, y, width = 0, height = 0, scaleX = 1, scaleY = 1, angle = 0 }) {
        // TODO: Handle rotated Objcets
        if (angle) {
          return { x, y }
        }
        width = width * scaleX
        height = height * scaleY
        if (x <= 0) {
          x = 1
        }
        if (y <= 0) {
          y = 1
        }
        if (x + width >= this.canvas.width) {
          x = this.canvas.width - width - 1
        }
        if (y + height >= this.canvas.height) {
          y = this.canvas.height - height - 1
        }
        return { x, y }
      },
      // DRAW OBJECTS FUNCTIONS
      createObject(event) {
        // Creating the object inside the canvas
        const { x, y } = this.handleObjectPositionInCanvas(event.pointer)
        // Default options used for all the shapes
        const options = {
          left: x,
          top: y,
          fill: this.fill
        }
        const shape = this.objectName
        // Adding extra information based on the shape
        switch (shape) {
          case 'Circle':
            options.radius = 4
            break
          case 'Triangle':
            options.width = 8
            options.height = 8
            break
          case 'Rect':
            options.width = 8
            options.height = 8
            break
          case 'Ellipse':
            options.rx = 4
            options.ty = 4
            break
          case 'Line':
            console.log('options', options)
            options.x1 = options.left
            options.y1 = options.top
            options.strokeWidth = 8
            options.stroke = 'black'
            delete options.left
            delete options.top
            delete options.fill
            break
        }
        this.current = new fabric[shape](options)
        this.canvas.add(this.current)
        this.canvas.setActiveObject(this.current)
      },
      // OBJECT EVENTS FUNCTIONS
      onObjectMoving() {
        const active = this.canvas.getActiveObject()
        let { x, y } = this.handleObjectPositionInCanvas({ x: active.left, y: active.top, ...active })
        active.set({ top: y, left: x })
        this.canvas.renderAll()
      },
      onObjectScaling() {
        // TODO: Handle object scaling in a negative direction (scaling to the top or to the left)
        const active = this.canvas.getActiveObject()
        let { left: x, top: y, width, height, scaleX, scaleY } = active
        const newWidth = width * scaleX
        const newHeight = height * scaleY
        if (x + newWidth >= this.canvas.width) {
          const maxWidth = this.canvas.width - x - 1
          scaleX = maxWidth / width
        }
        if (y + newHeight >= this.canvas.height) {
          const maxHeight = this.canvas.height - y - 1
          scaleY = maxHeight / height
        }
        active.set({ scaleX, scaleY })
        this.canvas.renderAll()
      },
      // MOUSE EVENTS FUNCTIONS
      onMouseDown(event) {
        if (!this.objectName || this.canvas.getActiveObject() || this.objectName === 'Paint') {
          return
        }
        this.canvas.selection = false
        this.originalPointer.x = event.pointer.x
        this.originalPointer.y = event.pointer.y
        this.createObject(event)
      },
      onMouseMove(event) {
        if (!this.objectName || !this.current || this.objectName === 'Paint') {
          return
        }
        // This handles scalling the created shape
        let { x, y } = this.handleObjectPositionInCanvas(event.pointer)
        const { x: originalX, y: originalY } = this.originalPointer
        let change

        if (this.objectName === 'Rect') {
          change = {
            width: Math.abs(x - originalX),
            height: Math.abs(y - originalY)
          }
        } else if (this.objectName === 'Circle') {
          const radius = Math.abs(x - originalX) / 2
          change = {
            radius
          }
        } else if (this.objectName === 'Triangle') {
          change = {
            width: Math.abs(x - originalX),
            height: Math.abs(y - originalY)
          }
        } else if (this.objectName === 'Ellipse') {
          change = {
            rx: Math.abs(x - originalX) / 2,
            ry: Math.abs(y - originalY) / 2
          }
        } else if (this.objectName === 'Line') {
          change = {
            stroke: this.fill,
            x1: originalX,
            y1: originalY,
            x2: Math.abs(x),
            y2: Math.abs(y)
          }
        }
        this.current.set({ ...change })
        this.canvas.renderAll()
      },
      onMouseUp() {
        if (this.objectName === 'Paint') {
          return
        }
        this.current = null
        this.canvas.renderAll()

        this.canvas.selection = true
      }
    },
    watch: {
      objectName(value) {
        this.canvas.isDrawingMode = false
        if (value) {
          if (value === 'Paint') {
            this.canvas.isDrawingMode = true
            this.canvas.isDrawingMode = true
            this.canvas.freeDrawingBrush.width = 4
            this.canvas.freeDrawingBrush.color = this.fill
          } else if (value === 'DELETE-ALL') {
            this.canvas.clear()
          }
        }
      }
    },
    created() {
      if (this.canvasId) {
        this.canvasAttributes.id = this.canvasId
      }
      if (this.canvasWidth) {
        this.canvasAttributes.width = this.canvasWidth
      }
      if (this.canvasHeight) {
        this.canvasAttributes.height = this.canvasHeight
      }
    },
    render(createElement) {
      const attrs = this.canvasAttributes
      const style = this.canvasStyle
      return createElement('canvas', { attrs, style, ref: attrs.id })
    },
    mounted() {
      this.canvas = new fabric.Canvas(this.canvasAttributes.id)
      this.canvas.selection = true
      // this.canvas.renderOnAddRemove = true
      const events = [
        // MOUSE EVENTS
        'mouse:down',
        'mouse:up',
        'mouse:move',
        'mouse:dblclick',
        'mouse:over',
        'mouse:out',
        // SELECTION EVENTS
        'selection:created',
        'selection:updated',
        'before:selection:cleared',
        'selection:cleared',
        // OBJECT EVENTS
        'object:added',
        'object:removed',
        'object:modified',
        'object:rotating',
        'object:scaling',
        'object:moving'
      ]
      for (let i = 0; i < events.length; i++) {
        const eventName = events[i]
        const eventFunctionName =
          'on' +
          eventName
            .split(':')
            .map((part) => toCamelCase(part))
            .join('')
        if (this[eventFunctionName]) {
          this.canvas.on(eventName, (event) => {
            this[eventFunctionName](event)
            this.$emit(eventName, event)
          })
        } else {
          this.canvas.on(eventName, (event) => {
            this.$emit(eventName, event)
          })
        }
      }
    }
  }
</script>
