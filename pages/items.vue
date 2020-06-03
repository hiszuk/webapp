<template>
  <div>
    <el-row type="flex" justify="center" style="margin-top: 2rem;">
      <el-col :span="2">
        <div></div>
      </el-col>
      <el-col :span="20">
        <!-- 検索入力 -->
        <el-input v-model="search" size="small" placeholder="検索文字列を入力">
          <template slot="prepend">内容を検索する</template>
        </el-input>
        <!-- 表部分 -->
        <el-table ref="itemTable" :data="tableData.filter((data) => !search || data.content.toLowerCase().includes(search.toLowerCase()))" height="300" style="width: 100%">
          <el-table-column prop="id" label="ID" min-width="40" header-align="center" align="right"> </el-table-column>
          <el-table-column prop="title" label="タイトル" min-width="200" header-align="center" show-overflow-tooltip> </el-table-column>
          <el-table-column prop="content" label="内容" min-width="400" header-align="center" show-overflow-tooltip> </el-table-column>
          <el-table-column prop="status" label="状態" min-width="100" align="center">
            <template slot-scope="scope">
              <el-tag :type="scope.row.status === 'DONE' ? 'success' : scope.row.status === 'PROGRESS' ? 'primary' : 'warning'" disable-transitions>
                {{ scope.row.status }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column label="操作" min-width="120" header-align="center">
            <template slot-scope="scope">
              <el-button size="mini" round @click="handleEdit(scope.$index, scope.row)"><i class="el-icon-edit"></i> </el-button>
              <el-button size="mini" type="danger" round @click="handleDelete(scope.$index, scope.row)"><i class="el-icon-delete"></i> </el-button>
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
      <el-dialog title="Todo 編集" :visible.sync="dialogFormVisible">
        <el-form :model="form">
          <el-form-item label="ID" :label-width="formLabelWidth">
            <el-input v-model="form.id" :readonly="disableID"></el-input>
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
          <el-button type="primary" @click="doConfirm()">
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
      search: '',
      formLabelWidth: '150px',
      dialogFormVisible: false,
      disableID: true,
      form: {
        id: null,
        title: '',
        content: '',
        status: 'TODO'
      }
    }
  },
  methods: {
    handleEdit(index, row) {
      this.disableID = true
      this.form = row
      this.dialogFormVisible = true
    },
    handleDelete(index, row) {
      console.log(index, row)
    },
    doConfirm() {
      this.dialogFormVisible = false
      const target = `${this.form.id}:${this.form.title}`
      // ToDo: REST APIのアップデートを起動する
      this.$message({
        type: 'success',
        message: `${target} : 更新が成功しました。`,
        showClose: true,
        duration: 5000
      })
    }
  }
}
</script>
