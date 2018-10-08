<template>
  <el-container direction="vertical">
    <el-main class="full-height">
      <svg id="svgGraph" style="width:100%" class="full-height">
        <g id="svgGroup" class="svgGroup">
          <kgline v-for="line in data.lines" :key="line.source.x+line.source.y+line.target.x+line.target.y"
                  :line="line" :data="data"></kgline>
          <kgnode v-for="node in data.nodes" :key="node.id+node.x+node.y" :radius="radius" :data="data"
                  :node="node"></kgnode>
        </g>
      </svg>
    </el-main>
    <el-footer v-if="selectedData!==null" style="height:auto">
      <gdetail :itemData="selectedData"></gdetail>
    </el-footer>
  </el-container>
</template>

<script>
import * as d3 from 'd3'
import * as d3b from 'd3-bboxCollide'
import bus from './bus'
import kgnode from './KGnode'
import kgline from './KGline'
import gdetail from './graph-detail'

export default {
  name: 'knowledge-graph',
  components: {
    kgnode, kgline, gdetail
  },
  data: function () {
    return {
      radius: 25,
      zoomMax: 3,
      zoomMin: 0.1,
      svgLeft: 0,
      svgTop: 0,
      refreshTrigger: 1,
      alphaDecay: 0.1,
      strength: 1.5,
      selectedData: null
    }
  },
  props: {
    graphData: {
      type: Object,
      required: true
    },
    graphSize: {
      type: Object,
      required: true
    }
  },
  computed: {
    gwidth: function () {
      let width = this.graphSize.width - 80
      // console.log('width', width)
      return width
    },
    gheight: function () {
      let height = 400
      // console.log('height', height)
      return height
    },
    tickTimes: function () {
      return Math.log(0.001) / Math.log(1 - this.alphaDecay)
    },
    svgGroup: function () {
      return d3.select('#svgGraph')
        .call(d3.zoom()
          .scaleExtent([this.zoomMin, this.zoomMax])
          .on('zoom', () => {
            // console.log('transform')
            this.svgGroup.attr('transform', d3.event.transform)
          }))
        .on('dblclick.zoom', null)
        .select('#svgGroup')
    },
    simulation: function () {
      let simulation = d3.forceSimulation()
        .alphaDecay(this.alphaDecay)
        .force('charge',
          d3.forceManyBody())
        .force('collide',
          d3b.bboxCollide((d, i) => {
            return [[-this.radius * 2, -this.radius * 5], [this.radius * 2, this.radius * 5]]
          }))
        // d3.forceCollide(this.radius * 3))
        .force('center', d3.forceCenter(this.gheight / 2, this.gwidth / 2))
        .nodes(this.graphData.nodes)
        .force('link', d3.forceLink(this.graphData.lines).strength(this.strength))
        .on('tick', this.ticked)
      // console.log('sim')
      this.resetSelectedData()
      return simulation
    },
    data: function () {
      if (this.refreshTrigger !== null && this.graphData !== null && this.simulation !== null) {
        // console.log(JSON.stringify(this.graphData))
        // console.log('data')
        for (let i = 0; i < this.tickTimes; i++) {
          this.simulation.tick()
        }
        return this.graphData
      }
      return {nodes: null, lines: null}
    }
  },
  methods: {
    ticked: function () {
      // console.log('ticked', JSON.stringify(this.data.nodes))
      this.refreshTrigger = -this.refreshTrigger
      // console.log('tick')
      // this.graphData.nodes.splice(0, 0)
      // this.graphData.lines.splice(0, 0)
    },
    resetSelectedData: function () {
      this.selectedData = null
    }
  },
  created () {
    bus.$on('mouseover', item => {
      this.selectedData = item
      console.log(item.type, item.data)
    })
  },
  mounted () {
    if (this.svgGroup !== null) {
      console.log('svgGroup')
    }
  }
}
</script>

<style scoped>
  .full-height {
    height: 100%;
  }

  #svgGraph {
    border: 1px solid #cccccc;
    border-radius: 5px;
    /*font: 14px Small;*/
  }
</style>
