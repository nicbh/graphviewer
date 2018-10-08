<template>
  <g v-if="data.nodes!==undefined && data.nodes.length>0" class="kgnode" :transform="`translate(${dy}, ${dx})`"
     @mouseover="mouseOver(node)">
    <rect :x="-radius*3" :y="-radius" :width="radius*6" :height="radius*2" :rx="radius" :ry="radius"
          :style="rectStyle"></rect>
    <text :dy="textDy" :style="textStyle">{{ node.friendly_name }}</text>
  </g>
</template>

<script>
import bus from './bus.js'

export default {
  name: 'k-gnode',
  props: {
    radius: {
      type: Number,
      required: true
    },
    data: {
      type: Object,
      required: true
    },
    node: {
      type: Object,
      required: true
    }
  },
  data: function () {
    return {
      textDx: 0,
      textDy: '.31em',
      type2stroke: {
        'class': 'green',
        'entity': 'blue',
        'literal': 'black'
      },
      ques2fill: {
        1: '#eee',
        0: '#fff'
      },
      func2stroke: {
        'none': 'steelblue',
        'others': 'sienna'
      }
    }
  },
  computed: {
    dx: function () {
      return this.node.x
    },
    dy: function () {
      return this.node.y
    },
    rectStyle: function () {
      let stroke = null
      if (Object.keys(this.func2stroke).find((item) => item === this.node.function)) {
        stroke = this.func2stroke[this.node.function]
      } else {
        stroke = this.func2stroke['others']
      }
      return `fill:${this.ques2fill[this.node.question_node]};stroke:${stroke}`
    },
    textStyle: function () {
      return `stroke:${this.type2stroke[this.node.node_type]}`
    }
  },
  methods: {
    mouseOver: function (item) {
      bus.$emit('mouseover', {type: 'node', data: item})
    }
  }
}
</script>

<style scoped>
  .kgnode {
    display: inline-block;
  }

  div {
    display: inline-block;
  }

  .kgnode rect {
    /*fill: #fff;*/
    /*stroke: steelblue;*/
    stroke-width: 1.5;
  }

  .kgnode text {
    font: 13px Extra Small;
    text-anchor: middle;
    stroke-width: 0.5;
  }
</style>
