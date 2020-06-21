<template>
  <div>
    <el-row type="flex" justify="center" style="margin-top: 2rem;">
      <el-col :span="2">
        <div></div>
      </el-col>
      <el-col :span="20">
        <!-- 検索入力 -->
        <el-row>
          <el-col :span="20">
            <el-input v-model="search" size="small" placeholder="検索文字列を入力">
              <template slot="prepend">内容で絞り込む</template>
            </el-input>
          </el-col>
          <el-col :span="4" style="text-align: center">
            <el-button type="success" icon="el-icon-circle-plus-outline" size="small" @click="handleNew()">新規登録</el-button>
          </el-col>
        </el-row>
        <!-- 表部分 -->
        <el-table ref="itemTable" :data="tableData.filter((data) => !search || data.content.toLowerCase().includes(search.toLowerCase()))" height="400" style="width: 100%">
          <el-table-column prop="id" label="ID" min-width="40" header-align="center" align="right"> </el-table-column>
          <el-table-column prop="title" label="タイトル" min-width="200" header-align="center" show-overflow-tooltip> </el-table-column>
          <el-table-column prop="content" label="内容" min-width="300" header-align="center" show-overflow-tooltip> </el-table-column>
          <el-table-column prop="status" label="状態" min-width="100" align="center">
            <template slot-scope="scope">
              <el-tag :type="scope.row.status === 'DONE' ? 'success' : scope.row.status === 'PROGRESS' ? 'primary' : 'warning'" disable-transitions>
                {{ scope.row.status }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column label="操作" min-width="120" header-align="center">
            <template slot-scope="scope">
              <el-button size="small" circle @click="handleEdit(scope.$index, scope.row)"><i class="el-icon-edit"></i> </el-button>
              <el-button size="small" type="danger" circle @click="handleDelete(scope.$index, scope.row)"><i class="el-icon-delete"></i> </el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-col>
      <el-col :span="2">
        <div></div>
      </el-col>
    </el-row>
    <!-- 編集入力 Dialog -->
    <div>
      <el-dialog :title="dialogTitle" :visible.sync="dialogFormVisible">
        <el-form :model="form">
          <el-form-item label="ID" :label-width="formLabelWidth">
            <el-input v-model="form.id" disabled></el-input>
          </el-form-item>
          <el-form-item label="タイトル" :label-width="formLabelWidth">
            <el-input v-model="form.title"></el-input>
          </el-form-item>
          <el-form-item label="内容" :label-width="formLabelWidth">
            <el-input v-model="form.content" type="textarea"></el-input>
          </el-form-item>
          <el-form-item label="状態" :label-width="formLabelWidth">
            <el-select v-model="form.status" placeholder="選択してください">
              <el-option label="TODO" value="TODO"></el-option>
              <el-option label="PROGRESS" value="PROGRESS"></el-option>
              <el-option label="DONE" value="DONE"></el-option>
            </el-select>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button type="info" plain @click="dialogFormVisible = false">
            <i class="el-icon-circle-close" style="font-size: 120%"></i>
            <span>中止</span>
          </el-button>
          <el-button type="primary" @click="doExecute()">
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
      tableData: [
        {
          id: '1',
          title: 'WEBApplication作成',
          content: 'NUXT.js+ElementUIでフロントエンドのWEBアプリを作成する',
          status: 'DONE'
        },
        {
          id: '2',
          title: 'RESTfulAPI作成',
          content: 'SpringBootを用いてRESTfulAPIを構築する',
          status: 'DONE'
        },
        {
          id: '3',
          title: 'フロントエンドとバックエンドを結合',
          content: 'NUXTのアプリからaxios経由でRESTAPIをコールしフロントとバックエンドを繋げる',
          status: 'PROGRESS'
        },
        {
          id: '4',
          title: 'MySQLのDockerイメージを作成する',
          content: 'DockerでRESTfulAPIでアクセスするMySQLのイメージを作成する',
          status: 'PROGRESS'
        },
        {
          id: '5',
          title: 'バックエンドのDockerイメージを作成する',
          content: 'Javaのイメージにバックエンドのjarをレイヤー化したイメージを作成する',
          status: 'TODO'
        },
        {
          id: '6',
          title: 'フロントエンドのDockerイメージを作成する',
          content: 'Node.jsからNUXTをインストールしWEBアプリをコピー後にbuildするイメージを作成する',
          status: 'TODO'
        },
        {
          id: '7',
          title: 'Docker-composeでパッケージ化する',
          content: 'フロントエンド・バックエンド・DBの3層をひとつのパッケージにまとめる',
          status: 'TODO'
        }
      ],
      item: {
        id: null,
        title: '',
        content: '',
        status: ''
      },
      search: '',
      formLabelWidth: '150px',
      dialogFormVisible: false,
      isUpdate: true,
      rowNumber: '',
      form: {
        id: null,
        title: '',
        content: '',
        status: ''
      },
      nextId: 8
    }
  },
  computed: {
    dialogTitle() {
      return this.isUpdate ? 'ToDo 編集' : 'ToDo 新規登録'
    }
  },
  methods: {
    handleEdit(index, row) {
      this.isUpdate = true
      this.rowNumber = index
      this.fetchKey(row.id)
      this.form = { ...this.item }
      this.dialogFormVisible = true
    },
    handleDelete(index, row) {
      this.rowNumber = index
      this.confirmDelete()
    },
    handleNew() {
      this.isUpdate = false
      this.form.id = null
      this.form.title = ''
      this.form.content = ''
      this.form.status = 'TODO'
      this.dialogFormVisible = true
      // 以下はREST APIと接続するまでのダミー処理
      // idはREST APIでは自動採番の予定
      this.form.id = this.nextId
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
    fetchKey(id) {
      // ToDo: REST APIのkey(row.id)検索し結果をitemにセットする
      // Dummy select
      const items = this.tableData.filter((data) => data.id === id)
      this.item = { ...items[0] }
    },
    updateItem(param) {
      // ToDo: REST APIのアップデートを起動する
      // Dummy update
      this.tableData[this.rowNumber] = { ...param }

      // 更新後の情報を取得しtableDataにセットする
      this.fetchKey(param.id)
      this.tableData[this.rowNumber] = { ...this.item }

      const target = `${param.id}:${param.title}`
      this.$message({
        type: 'success',
        message: `${target} : 更新が成功しました。`,
        showClose: true,
        duration: 5000
      })
    },
    deleteItem(id) {
      // ToDo: REST API削除処理呼び出し
      // Dummy delete
      this.tableData = this.tableData.filter((data) => data.id !== id)
    },
    registerItem(param) {
      // ToDo: REST API登録処理呼び出し
      // Dummiy insert
      const item = { ...param }
      this.tableData.push(item)
      this.nextId++

      const target = `${param.id}:${param.title}`
      this.$message({
        type: 'success',
        message: `${target} : 登録が成功しました。`,
        showClose: true,
        duration: 5000
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
          this.$message({
            type: 'success',
            message: `${target} : 削除が成功しました。`,
            showClose: true,
            duration: 5000
          })
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
