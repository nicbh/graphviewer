<template>
  <el-container id="graph-detail">
    <el-header class="panel-header place" height="40px">
      <div v-for="item in headerData" :key="id+item[0]" class="text">
        <b>{{item[0]}}:</b> {{item[1]}}
      </div>
    </el-header>
    <el-main class="panel-body" height="100%">
      <div v-for="row in bodyData" :key="id+'row'+row[0][0]" v-if="row.length>0" class="place">
        <div v-for="item in row" :key="id+item[0]" class="text">
          <b>{{item[0]}}:</b> {{item[1]}}
        </div>
      </div>
    </el-main>
  </el-container>
</template>

<script>
export default {
  name: 'graph-detail',
  props: {
    itemData: {
      type: Object,
      required: true
    }
  },
  data: function () {
    return {
      priority: {
        'friendly_name': 1,
        'nid': 2,
        'node_type': 11,
        'question_node': 12,
        'function': 13,
        'id': 21,
        'class': 31,
        'others': 41
      },
      priorityNum: 5
    }
  },
  computed: {
    id: function () {
      let item = this.itemData.data
      if (this.itemData.type === 'node') {
        return item.id + item.x + item.y
      } else {
        return item.source.x + item.source.y + item.target.x + item.target.y
      }
    },
    data: function () {
      let type = this.itemData.type
      let item = this.itemData.data
      let data = []
      if (type === 'line') {
        data = [
          [['friendly_name', item.friendly_name, 0]],
          [['relation', item.relation, 0]],
          [['start', item.source.friendly_name, 0], ['end', item.target.friendly_name, 1]]
        ]
      } else {
        for (let i = 0; i < this.priorityNum; i++) {
          data.push([])
        }
        for (let attr in item) {
          let prior = this.priority[attr]
          if (prior === undefined) {
            // prior = this.priority['others']
            continue
          }
          let floor = Math.floor(prior / 10)
          let dattr = item[attr]
          data[floor].push([attr, dattr, prior])
        }
      }
      for (let index in data) {
        let rdata = data[index].sort((a, b) => a[2] - b[2])
        // let spans = this.span(rdata)
        // for (let lindex in rdata) {
        //   rdata[lindex] = {
        //     data: rdata[lindex],
        //     span: spans[lindex]
        //   }
        // }
        data[index] = rdata
      }
      // console.log(data)
      return data
    },
    headerData: function () {
      return this.data[0]
    },
    bodyData: function () {
      return this.data.slice(1)
    }
  },
  methods: {
    span: function (row) {
      let len = []
      let totalLen = 0
      for (let index in row) {
        let item = row[index]
        let str = item[0] + ': ' + item[1]
        len.push(str.length)
        totalLen += str.length
      }
      for (let index in len) {
        len[index] = Math.floor(len[index] / totalLen * 24)
      }
      return len
    }
  }
}
</script>

<style scoped>
  .panel-header {
    padding: 10px 15px;
    font: 14px Small;
    color: #31708f;
    background-color: #d9edf7;
    border-bottom: 1px solid #bce8f1;
    border-top-right-radius: 3px;
    border-top-left-radius: 3px;
  }

  .panel-body {
    padding: 10px 15px;
    font: 14px Small;
    background-color: #ffffff;
    /*border-top: 1px solid inherit;*/
    border-bottom-right-radius: 3px;
    border-bottom-left-radius: 3px;
  }

  #graph-detail {
    border: 1px solid #bce8f1;
    border-radius: 5px;
    font: 14px Small;
  }
</style>
