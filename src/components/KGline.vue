<template>
  <g v-if="data.lines!==undefined && data.lines.length>0" class="kgline" @mouseover="mouseOver(line)"
     :transform="`translate(${(source.y+target.y)/2}, ${(source.x+target.x)/2})`">
    <path :d="`M${-dy/2},${-dx/2}L${dy/2},${dx/2}`" :style="lineStyle"></path>
    <text :dy="textDy">{{ line.friendly_name }}</text>
  </g>
</template>

<script>
import bus from './bus'

export default {
  name: 'k-gline',
  props: {
    data: {
      type: Object,
      required: true
    },
    line: {
      type: Object,
      required: true
    }
  },
  data: function () {
    return {
      textDx: 0,
      textDy: '.31em'
    }
  },
  computed: {
    source: function () {
      return this.line.source
    },
    target: function () {
      return this.line.target
    },
    dx: function () {
      return this.target.x - this.source.x
    },
    dy: function () {
      return this.target.y - this.source.y
    },
    d: function () {
      console.log('compute d')
      const parent = this.line.source
      const node = this.line.target
      return 'M' + parent.y + ',' + parent.x +
        // 'C' + (parent.y + parent.y) / 2 + ',' + parent.x +
        // ' ' + (parent.y + node.y) / 2 + ',' + node.x +
        'L' + node.y + ',' + node.x
    },
    lineStyle: function () {
      return {
        stroke: '#666666'
      }
    }
  },
  methods: {
    mouseOver: function (item) {
      bus.$emit('mouseover', {type: 'line', data: item})
    }
  }
}
</script>

<style scoped>
  path {
    fill: none;
    stroke-width: 2
  }

  .kgline text {
    font: 13px Extra Small;
    text-anchor: middle;
    stroke: black;
    stroke-width: 0.5;
  }
</style>
