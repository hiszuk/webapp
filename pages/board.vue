<template>
  <div>
    <el-row type="flex" justify="center" style="margin-top: 2rem;">
      <el-col :span="1">
        <div></div>
      </el-col>
      <el-col :span="7" class="grid-content todo">
        <el-button type="success" size="mini" icon="el-icon-plus" circle style="float: right; margin:2px 5px 2px 0px;" @click="handleNew()"></el-button>
        <h3 style="margin-bottom: 0.5em;text-align:center;color:#d6922c;">TODO</h3>
        <draggable v-model="itemsTodo" class="TODO" group="items" :options="{ animation: 500, handle: '.handle' }" @start="drag = true" @end="onEnd">
          <div v-for="item in itemsTodo" :key="item.id">
            <el-card class="box-card">
              <div slot="header" class="clearfix handle">
                <span>ID: {{ item.id }}</span>
                <el-button type="danger" size="mini" icon="el-icon-delete" circle style="float: right; margin:2px;" @click="handleDelete(item.id)"></el-button>
                <el-button type="primary" size="mini" icon="el-icon-edit" circle style="float: right; margin:2px;" @click="handleEdit(item.id)"></el-button>
              </div>
              <div class="title">{{ item.title }}</div>
              <div class="text">{{ item.content }}</div>
            </el-card>
          </div>
        </draggable>
      </el-col>
      <el-col :span="7" class="grid-content progress">
        <h3 style="margin-bottom: 0.5em;text-align:center;color:#308eef;">PROGRESS</h3>
        <draggable v-model="itemsProgress" class="PROGRESS" group="items" :options="{ animation: 500, handle: '.handle' }" @start="drag = true" @end="onEnd">
          <div v-for="item in itemsProgress" :key="item.id">
            <el-card class="box-card">
              <div slot="header" class="clearfix handle">
                <span>ID: {{ item.id }}</span>
                <el-button type="danger" size="mini" icon="el-icon-delete" circle style="float: right; margin:2px;" @click="handleDelete(item.id)"></el-button>
                <el-button type="primary" size="mini" icon="el-icon-edit" circle style="float: right; margin:2px;" @click="handleEdit(item.id)"></el-button>
              </div>
              <div class="title">{{ item.title }}</div>
              <div class="text">{{ item.content }}</div>
            </el-card>
          </div>
        </draggable>
      </el-col>
      <el-col :span="7" class="grid-content done">
        <h3 style="margin-bottom: 0.5em;text-align:center;color:#57b22a;">DONE</h3>
        <draggable v-model="itemsDone" class="DONE" group="items" :options="{ animation: 500, handle: '.handle' }" @start="drag = true" @end="onEnd">
          <div v-for="item in itemsDone" :key="item.id">
            <el-card class="box-card">
              <div slot="header" class="clearfix handle">
                <span>ID: {{ item.id }}</span>
                <el-button type="danger" size="mini" icon="el-icon-delete" circle style="float: right; margin:2px;" @click="handleDelete(item.id)"></el-button>
                <el-button type="primary" size="mini" icon="el-icon-edit" circle style="float: right; margin:2px;" @click="handleEdit(item.id)"></el-button>
              </div>
              <div class="title">{{ item.title }}</div>
              <div class="text">{{ item.content }}</div>
            </el-card>
          </div>
        </draggable>
      </el-col>
      <el-col :span="1">
        <div></div>
      </el-col>
    </el-row>
    <!-- Dialog -->
    <div>
      <el-dialog :title="dialogTitle" :visible.sync="dialogFormVisible">
        <el-form :model="form" size="mini">
          <!-- <el-form-item label="ID" :label-width="formLabelWidth">
            <el-input v-model="form.id" disabled></el-input>
          </el-form-item> -->
          <el-form-item label="タイトル" :label-width="formLabelWidth">
            <el-input v-model="form.title"></el-input>
          </el-form-item>
          <el-form-item label="内容" :label-width="formLabelWidth">
            <el-input v-model="form.content" type="textarea"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button type="info" size="mini" plain @click="dialogFormVisible = false">
            <i class="el-icon-circle-close" style="font-size: 120%"></i>
            <span>中止</span>
          </el-button>
          <el-button type="primary" size="mini" @click="doExecute()">
            <i class="el-icon-circle-check" style="font-size: 120%"></i>
            <span>確定</span>
          </el-button>
        </span>
      </el-dialog>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      tableData: [],
      itemsTodo: [],
      itemsProgress: [],
      itemsDone: [],
      formLabelWidth: '80px',
      dialogFormVisible: false,
      isUpdate: true,
      rowNumber: null,
      form: {
        id: null,
        title: '',
        content: '',
        status: ''
      }
    }
  },
  computed: {
    dialogTitle() {
      return this.isUpdate ? 'ToDo 編集' : 'ToDo 新規登録'
    }
  },
  mounted() {
    this.fetchAll()
  },
  methods: {
    classifyItems() {
      this.itemsTodo = [...this.tableData.filter((data) => data.status === 'TODO')]
      this.itemsProgress = [...this.tableData.filter((data) => data.status === 'PROGRESS')]
      this.itemsDone = [...this.tableData.filter((data) => data.status === 'DONE')]
    },
    onEnd(event) {
      const id = event.item._underlying_vm_.id
      const from = event.from.className
      const to = event.to.className
      if (from !== to) {
        // console.log(`${id} move to ${to}`)
        const items = this.tableData.filter((d) => d.id === id)
        const item = items[0]
        this.rowNumber = this.tableData.indexOf(item)
        item.status = to
        this.updateItem(item)
        this.classifyItems()
      }
    },
    async fetchAll() {
      await this.$axios.$get('/api/items/').then((res) => {
        this.tableData = [...res]
        this.classifyItems()
      })
    },
    async fetchKey(id) {
      await this.$axios.$get(`/api/items/${id}`).then((res) => {
        this.form = { ...res }
        this.tableData[this.rowNumber] = { ...res }
        this.classifyItems()
      })
    },
    handleEdit(id) {
      this.isUpdate = true
      const rows = this.tableData.filter((data) => data.id === id)
      const row = rows[0]
      this.rowNumber = this.tableData.indexOf(row)
      this.fetchKey(row.id).then((res) => {
        this.dialogFormVisible = true
      })
    },
    handleDelete(id) {
      const rows = this.tableData.filter((data) => data.id === id)
      const row = rows[0]
      this.rowNumber = this.tableData.indexOf(row)
      this.confirmDelete()
    },
    handleNew() {
      this.isUpdate = false
      this.form.id = null
      this.form.title = ''
      this.form.content = ''
      this.form.status = 'TODO'
      this.dialogFormVisible = true
    },
    doExecute() {
      this.dialogFormVisible = false
      // 編集モード(isUpdate=true)の場合は更新処理
      if (this.isUpdate) {
        this.updateItem(this.form)
      } else {
        this.registerItem(this.form)
      }
    },
    updateItem(param) {
      const target = `${param.id}:${param.title}`
      this.$axios({
        url: `/api/items/${param.id}`,
        method: 'PUT',
        data: param
      })
        .then((res) => {
          this.fetchKey(param.id)
        })
        .catch((err) => {
          this.$message({
            type: 'danger',
            message: `${target} : 更新に失敗しました。: ${err}`,
            showClose: true,
            duration: 0
          })
        })
    },
    deleteItem(id) {
      const target = `${id}`
      this.$axios
        .$delete(`/api/items/${id}`)
        .then((res) => {
          this.tableData = this.tableData.filter((data) => data.id !== id)
          this.$message({
            type: 'success',
            message: `${target} : 削除が成功しました。`,
            showClose: true,
            duration: 5000
          })
        })
        .catch((err) => {
          this.$message({
            type: 'danger',
            message: `${target} : 削除に失敗しました。: ${err}`,
            showClose: true,
            duration: 0
          })
        })
    },
    registerItem(param) {
      this.rowNumber = this.tableData.length
      this.$axios({
        url: `/api/items/`,
        method: 'POST',
        data: param
      })
        .then((res) => {
          this.fetchKey(res.data).then((response) => {
            const target = `${res.data}:${param.title}`
            this.$message({
              type: 'success',
              message: `${target} : 登録が成功しました。`,
              showClose: true,
              duration: 5000
            })
          })
        })
        .catch((err) => {
          const target = `${param.title}`
          this.$message({
            type: 'danger',
            message: `${target} : 登録に失敗しました。: ${err}`,
            showClose: true,
            duration: 0
          })
        })
    },
    confirmDelete() {
      const row = this.tableData[this.rowNumber]
      const target = `${row.id}:${row.title}`
      const msg = `${target} を削除します。削除後は元に戻せませんが、実行してよろしいですか?`
      this.$confirm(msg, 'Warning', {
        confirmButtonText: '削除する',
        cancelButtonText: 'やめる',
        type: 'warning'
      })
        .then(() => {
          this.deleteItem(row.id)
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '削除を中止しました。'
          })
        })
    }
  }
}
</script>

<style>
.el-card__header {
  padding: 5px 10px 0px 20px;
}
.el-card__body {
  padding: 5px 10px 20px 20px;
}
.box-card .title {
  font-size: 0.8em;
  background-color: beige;
}
.box-card .text {
  font-size: 0.7em;
}
.handle {
  cursor: grab;
}
.handle:active {
  cursor: grabbing;
}

.grid-content {
  height: calc(100vh - 120px);
  padding: 3px;
  overflow-y: auto;
}
.todo {
  background-color: #eedabb;
}
.progress {
  background-color: #b8d4f0;
}
.done {
  background-color: #c9ecb8;
}

.clearfix:before,
.clearfix:after {
  display: block;
  content: '';
}
.clearfix:after {
  clear: both;
}
.clearfix {
  margin: 0px;
  padding: 0px;
}
.box-card {
  margin: 0 auto;
  width: 90%;
}
</style>
