<template>
  <el-container class="full-height">
    <el-aside width="50%">
      <div class="place">
        <el-upload action="null" :auto-upload="false" :on-change="fileChange" :file-list="fileList">
          <el-button slot="trigger" icon="el-icon-plus" size="mini">LOAD</el-button>
        </el-upload>
        <div>
          <el-switch v-model="indent" active-text="readable" inactive-text="compact"></el-switch>
          <el-button icon="el-icon-download" type="primary" size="mini" @click="download">DOWN</el-button>
        </div>
      </div>
      <el-table ref="singleTable" :data="data" class="full-height" :max-height="availHeight"
                highlight-current-row @current-change="handleCurrentChange">
        <el-table-column type="index" sortable :width="indexWidth"></el-table-column>
        <el-table-column property="question" label="Question"></el-table-column>
        <el-table-column property="display_answer" label="Answer" width="120"></el-table-column>
        <el-table-column property="num_node" sortable label="Nd" width="63"></el-table-column>
        <el-table-column property="num_edge" sortable label="Eg" width="62"></el-table-column>
        <el-table-column property="function" sortable label="Func" width="77"></el-table-column>
      </el-table>
    </el-aside>
    <el-container v-if="selectedData!==null" class="right-part full-height">
      <el-header class="qa-header" ref="header" height="auto">
        <qa-detail :rdata="selectedData"></qa-detail>
      </el-header>
      <el-main class="full-height">
        <KG class="full-height" :graph-data="selectedData.graph_query" :graph-size="kgSize"></KG>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
import KG from './components/KnowledgeGraph'
import qaDetail from './components/qa-detail'
import bus from './components/bus'
import * as d3 from 'd3'

export default {
  name: 'app',
  components: {
    KG, qaDetail
  },
  data: function () {
    return {
      data: null,
      selectedData: null,
      selectedIndex: null,
      fileList: [],
      indent: false
    }
  },
  computed: {
    availHeight: function () {
      return document.documentElement.clientHeight
    },
    indexWidth: function () {
      if (this.data === null) {
        return 50
      }
      console.log(this.data.length)
      return 20 + `${this.data.length}`.length * 10
    },
    kgSize: function () {
      var height = this.availHeight
      return {
        width: document.documentElement.clientWidth / 2,
        height: height
      }
    },
    loadData: {
      get: function () {
        let data = JSON.parse(JSON.stringify(this.data))
        for (let itemIndex in data) {
          // console.log(item)
          let item = data[itemIndex]
          let query = item.graph_query
          for (let nodeIndex in query.nodes) {
            let node = query.nodes[nodeIndex]
            delete node.index
            delete node.x
            delete node.y
            delete node.vx
            delete node.vy
          }
          query.edges = query.lines
          delete query.lines
          for (let lineIndex in query.edges) {
            let line = query.edges[lineIndex]
            line.start = line.source.nid
            line.end = line.target.nid
            delete line.source
            delete line.target
            delete line.index
          }
          delete item.display_answer
        }
        return data
      },
      set: function (data) {
        for (let itemIndex in data) {
          // console.log(item)
          let item = data[itemIndex]
          let query = item.graph_query
          query.lines = query.edges
          delete query.edges
          for (let lineIndex in query.lines) {
            let line = query.lines[lineIndex]
            line.source = line.start
            line.target = line.end
            delete line.start
            delete line.end
          }
          item.display_answer = this.display(item.answer)
        }
        this.data = data
      }
    }
  },
  methods: {
    handleCurrentChange: function (val) {
      // console.log(val)
      if (val === null && this.selectedIndex !== null) {
        this.$refs.singleTable.setCurrentRow(this.data[this.selectedIndex])
        return
      }
      this.selectedData = val
      let index = this.data.findIndex((item) => item === this.selectedData)
      this.selectedIndex = index
    },
    display: function (answer) {
      if (answer.length === 1) {
        return answer[0]
      } else {
        return `[${answer[0]}, ${answer[1]},...]`
      }
    },
    fileChange: function (file, fileList) {
      this.fileList = fileList.slice(-1)
      console.log(this.fileList)
      let reader = new FileReader()
      reader.onload = (e) => {
        // console.log(e.target.result)
        let data = JSON.parse(e.target.result)
        this.loadData = data
      }
      reader.readAsText(this.fileList[0].raw)
    },
    download: function () {
      let eleLink = document.createElement('a')
      let filename = 'data.json'
      if (this.fileList.length === 1) {
        filename = this.fileList[0].name
      }
      eleLink.download = filename
      eleLink.style.display = 'none'
      let data = (this.indent) ? JSON.stringify(this.loadData, null, 4) : JSON.stringify(this.loadData)
      eleLink.href = URL.createObjectURL(new Blob([data]))
      document.body.appendChild(eleLink)
      eleLink.click()
      document.body.removeChild(eleLink)
    }
  },
  created () {
    bus.$on('add', (item) => {
      console.log('add', item)
      let ndata = {}
      ndata[item.key] = item.value
      ndata = Object.assign({}, this.selectedData, ndata)
      this.data.splice(this.selectedIndex, 1, ndata)
    })
  },
  mounted () {
    d3.json(`/static/data.json?${Date.now()}`).then((data) => {
      // data = data.slice(0, 100)
      // data = data.filter((item, index) => {
      //   return item.function !== 'none'
      // })
      this.loadData = data
    })
  }
}
</script>

<style>
  html, body, .full-height {
    height: 100%;
  }

  .place {
    width: 100%;
    display: flex;
    justify-content: space-between;
  }

  .place > div {
    display: inline-block;
  }

</style>
