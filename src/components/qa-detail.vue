<template>
  <el-container id="qa-detail">
    <el-header class="panel-header place" height="42px">
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
      <el-dialog title="Fill Attribute" :visible.sync="dialogVisible" width="30%">
        <el-form :model="form" ref="form" :rules="rules">
          <el-form-item label="KEY" label-width="70px" prop="key">
            <el-input v-model="form.key"></el-input>
          </el-form-item>
          <el-form-item label="VALUE" label-width="70px" prop="value">
            <el-input v-model="form.value" ref="focus"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="cancel()">CANCEL</el-button>
          <el-button type="primary" @click="submitForm()">DONE</el-button>
        </span>
      </el-dialog>
      <div style="text-align:right;">
        <el-button icon="el-icon-plus" size="mini" @click="showDialog()">ADD</el-button>
      </div>
    </el-main>
  </el-container>
</template>

<script>
import bus from './bus'

export default {
  name: 'qa-detail',
  props: {
    rdata: {
      type: Object,
      required: true
    }
  },
  data: function () {
    return {
      hiddenAttr: ['graph_query', 'sparql_query', 'display_answer'],
      priority: {
        'qid': 1,
        'commonness': 2,
        'num_node': 11,
        'num_edge': 12,
        'function': 13,
        'question': 21,
        'answer': 31,
        'others': 41
      },
      focusState: false,
      priorityNum: 5,
      dialogVisible: false,
      form: {
        key: 'label',
        value: ''
      },
      rules: {
        key: [{required: true, message: 'No Empty', trigger: 'blur'}],
        value: [{required: true, message: 'No Empty', trigger: 'blur'}]
      }
    }
  },
  computed: {
    id: function () {
      return this.rdata.qid
    },
    data: function () {
      let data = []
      for (let i = 0; i < this.priorityNum; i++) {
        data.push([])
      }
      let rdata = this.rdata
      for (let attr in rdata) {
        if (this.hiddenAttr.find((str) => str === attr) === undefined) {
          let prior = this.priority[attr]
          if (prior === undefined) {
            prior = this.priority['others']
          }
          let floor = Math.floor(prior / 10)
          let dattr = rdata[attr]
          if (attr === 'commonness') {
            dattr = dattr.toFixed(3)
          }
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
    },
    showDialog: function () {
      this.dialogVisible = true
      this.$nextTick(() => {
        this.$refs['focus'].focus()
      })
    },
    submitForm: function () {
      console.log(this.$refs)
      this.$refs['form'].validate((valid) => {
        if (valid) {
          // console.log('submit', this.form)
          bus.$emit('add', this.form)
          this.$refs['form'].resetFields()
          this.dialogVisible = false
        } else {
          // console.log('error submit!!')
          return false
        }
      })
    },
    cancel: function () {
      this.$refs['form'].resetFields()
      this.dialogVisible = false
    }
  }
}
</script>

<style scoped>
  .panel-header {
    padding: 10px 15px;
    font: 16px Medium;
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

  #qa-detail {
    border: 1px solid #bce8f1;
    border-radius: 5px;
    font: 14px Small;
  }
</style>
