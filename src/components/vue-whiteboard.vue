<template>
  <svg
    ref="container"
    :style="{
      backgroundColor: this.backgroundColor,
    }"
  ></svg>
</template>

<script>
import { onMounted, reactive, toRefs } from 'vue'
import { curveBasis, drag, line, select } from 'd3'

const setLineAttributes = (props) => {
  const styles = Object.assign(
    {},
    {
      fill: 'none',
      stroke: props.color,
      'stroke-width': props.size,
      'stroke-linejoin': props.linejoin,
      'stroke-linecap': props.linecap,
    },
    props.lineStyles
  )
  return Object.entries(styles)
    .map(([key, value]) => `${key}: ${value};`)
    .join(' ')
}

const getCoordinates = (event) => {
  const {
    sourceEvent: { offsetX, offsetY },
  } = event
  return [offsetX, offsetY]
}

const clearSvg = (container) => container.selectAll('*').remove()

const drawLine = (container, node) => container.node().appendChild(node)

const undoAction = (state) => {
  if (!state.undoStack.length) return
  state.redoStack.push(state.undoStack.pop())
  clearSvg(state.svg)
  state.undoStack.forEach((node) => drawLine(state.svg, node))
}

const redoAction = (state) => {
  if (!state.redoStack.length) return
  state.undoStack.push(state.redoStack.pop())
  clearSvg(state.svg)
  state.undoStack.forEach((node) => drawLine(state.svg, node))
}

const nodeToSvg = (node) => {
  node.setAttribute('xlink', 'http://www.w3.org/1999/xlink')
  const serializer = new XMLSerializer()
  const svgString = serializer
    .serializeToString(node)
    .replace(/(\w+)?:?xlink=/g, 'xmlns:xlink=')
    .replace(/NS\d+:href/g, 'xlink:href')
  const svgData = 'data:image/svg+xml;base64,' + btoa(unescape(encodeURIComponent(svgString)))
  return svgData
}

const saveSvg = async (state) =>
  new Promise((resolve) => {
    const container = state.container.cloneNode(true)
    const svg = select(container)
    clearSvg(svg)
    state.undoStack.forEach((node) => drawLine(svg, node.cloneNode(true)))
    const svgData = nodeToSvg(container)

    const width = state.container.clientWidth
    const height = state.container.clientHeight

    const canvas = document.createElement('canvas')
    const context = canvas.getContext('2d')
    canvas.width = width
    canvas.height = height

    const image = new Image()

    image.onload = () => {
      context.clearRect(0, 0, width, height)
      context.drawImage(image, 0, 0, width, height)
      resolve(canvas.toDataURL('image/png'))
    }

    image.src = svgData
  })

export default {
  name: 'VueWhiteboard',
  props: {
    color: {
      type: String,
      default: '#333333',
    },
    backgroundColor: {
      type: String,
      default: '#ffffff',
    },
    lineStyles: {
      type: Object,
      default: () => {},
    },
    size: {
      type: String,
      default: '5px',
    },
    linejoin: {
      type: String,
      default: 'round',
      validator: (val) => ['miter', 'round', 'bevel', 'miter-clip', 'arcs'].includes(val),
    },
    linecap: {
      type: String,
      default: 'round',
      validator: (val) => ['butt', 'square', 'round'].includes(val),
    },
  },
  setup(props, { emit }) {
    const d3Line = line().curve(curveBasis)
    const state = reactive({
      container: null,
      svg: null,
      activeLine: null,
      undoStack: [],
      redoStack: [],
    })

    const init = () => {
      state.svg = select(state.container).call(
        drag()
          .container(state.container)
          .subject(({ x, y }) => [
            [x, y],
            [x, y],
          ])
          .on('start', (event) => {
            state.activeLine = state.svg
              .append('path')
              .datum(event.subject)
              .attr('class', 'line')
              .attr('style', setLineAttributes(props))
            state.activeLine.attr('d', d3Line)
            emit('dragstart', {
              coordinates: getCoordinates(event),
              node: state.activeLine.node(),
              options: Object.assign({}, props),
            })

            event.on('drag', (event) => {
              const coordinates = [event.x, event.y]
              state.activeLine.datum().push(coordinates)
              state.activeLine.attr('d', d3Line)
              emit('drag', {
                coordinates,
                node: state.activeLine.node(),
                options: Object.assign({}, props),
              })
            })
            event.on('end', (event) => {
              state.activeLine.attr('d', d3Line)
              const node = state.activeLine.node()
              state.undoStack.push(node)
              emit('dragend', {
                coordinates: [event.x, event.y],
                node,
                options: Object.assign({}, props),
              })
            })
          })
      )

      emit('init', state.svg)
    }

    const undo = () => {
      undoAction(state)
      emit('undo')
    }

    const redo = () => {
      redoAction(state)
      emit('redo')
    }

    const clear = () => {
      clearSvg(state.svg)
      state.undoStack = []
      state.redoStack = []
      emit('clear')
    }

    const save = async () => {
      if (!state.undoStack.length) return
      const svg = await saveSvg(state)
      emit('save')
      return svg
    }

    onMounted(init)

    return {
      ...toRefs(state),
      undo,
      redo,
      clear,
      save,
    }
  },
}
</script>
