# ドラッグアンドドロップでTODOアイテムを並べ替える

## dragable インストール

プロジェクトディレクトリで以下コマンドを実行し、ドラッグアンドドロップのライブラリをプロジェクトにインストールする。
```
yarn add vuedraggable
```

## Nuxt環境設定

`plugins/vue-draggable.js`を作成
```JavaScript
import Draggable from 'vuedraggable'
import Vue from 'vue'

Vue.component('draggable', Draggable)
```

`nuxt.config.js`を編集し、pluginを使えるようにする。

```JavaScript
  /*
   ** Plugins to load before mounting the App
   */
  plugins: ['@/plugins/element-ui'],
```
ここに`vue-dgraggable`を追加する

```JavaScript
  /*
   ** Plugins to load before mounting the App
   */
  plugins: ['@/plugins/element-ui', '@/plugins/vue-draggable'],
```

## メニュー追加

`layouts/default.vue`に`board`メニューを追加する

```html
<template>
  <div>
    <el-menu :default-active="activeIndex" mode="horizontal" router>
      <el-menu-item index="/">About</el-menu-item>
      <el-menu-item index="/items">Items</el-menu-item>
    </el-menu>
    <nuxt />
  </div>
</template>
```
↓
```html
<template>
  <div>
    <el-menu :default-active="activeIndex" mode="horizontal" router>
      <el-menu-item index="/">About</el-menu-item>
      <el-menu-item index="/items">Items</el-menu-item>
      <el-menu-item index="/board">Board</el-menu-item>
    </el-menu>
    <nuxt />
  </div>
</template>
```

## ボードページ追加

`pages/board.vue`を追加する。

```html
<template>
  <div>
    <el-row type="flex" justify="center" style="margin-top: 2rem;">
      <el-col :span="1">
        <div></div>
      </el-col>
      <el-col :span="7" class="grid-content todo">
        <el-button type="success" size="mini" icon="el-icon-plus" circle style="float: right; margin:2px 5px 2px 0px;"></el-button>
        <h3 style="margin-bottom: 0.5em;text-align:center;color:#d6922c;">TODO</h3>
        <draggable v-model="itemsTodo" class="TODO" group="items" @start="drag = true" @end="onEnd">
          <div v-for="item in itemsTodo" :key="item.id">
            <el-card class="box-card">
              <div slot="header" class="clearfix">
                <span>ID: {{ item.id }}</span>
                <el-button type="danger" size="mini" icon="el-icon-delete" circle style="float: right; margin:2px;"></el-button>
                <el-button type="primary" size="mini" icon="el-icon-edit" circle style="float: right; margin:2px;"></el-button>
              </div>
              <div class="title">{{ item.title }}</div>
              <div class="text">{{ item.content }}</div>
            </el-card>
          </div>
        </draggable>
      </el-col>
      <el-col :span="7" class="grid-content progress">
        <h3 style="margin-bottom: 0.5em;text-align:center;color:#308eef;">PROGRESS</h3>
        <draggable v-model="itemsProgress" class="PROGRESS" group="items" @start="drag = true" @end="onEnd">
          <div v-for="item in itemsProgress" :key="item.id">
            <el-card class="box-card">
              <div slot="header" class="clearfix">
                <span>ID: {{ item.id }}</span>
                <el-button type="danger" size="mini" icon="el-icon-delete" circle style="float: right; margin:2px;"></el-button>
                <el-button type="primary" size="mini" icon="el-icon-edit" circle style="float: right; margin:2px;"></el-button>
              </div>
              <div class="title">{{ item.title }}</div>
              <div class="text">{{ item.content }}</div>
            </el-card>
          </div>
        </draggable>
      </el-col>
      <el-col :span="7" class="grid-content done">
        <h3 style="margin-bottom: 0.5em;text-align:center;color:#57b22a;">DONE</h3>
        <draggable v-model="itemsDone" class="DONE" group="items" @start="drag = true" @end="onEnd">
          <div v-for="item in itemsDone" :key="item.id">
            <el-card class="box-card">
              <div slot="header" class="clearfix">
                <span>ID: {{ item.id }}</span>
                <el-button type="danger" size="mini" icon="el-icon-delete" circle style="float: right; margin:2px;"></el-button>
                <el-button type="primary" size="mini" icon="el-icon-edit" circle style="float: right; margin:2px;"></el-button>
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
      rowNumber: null
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
        this.tableData[this.rowNumber] = { ...res }
      })
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

.grid-content {
  min-height: calc(100vh - 120px);
  padding: 3px;
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
```
