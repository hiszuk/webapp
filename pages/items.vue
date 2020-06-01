<template>
  <div>
    <el-row type="flex" justify="center" style="margin-top: 2rem;">
      <el-col :span="2">
        <div></div>
      </el-col>
      <el-col :span="20">
        <el-table
          ref="itemTable"
          :data="tableData"
          height="300"
          style="width: 100%"
        >
          <el-table-column
            prop="id"
            label="ID"
            min-width="40"
            header-align="center"
            align="right"
          >
          </el-table-column>
          <el-table-column
            prop="title"
            label="タイトル"
            min-width="200"
            header-align="center"
            show-overflow-tooltip
          >
          </el-table-column>
          <el-table-column
            prop="content"
            label="内容"
            min-width="400"
            header-align="center"
            show-overflow-tooltip
          >
          </el-table-column>
          <el-table-column
            prop="status"
            label="状態"
            min-width="100"
            align="center"
          >
            <template slot-scope="scope">
              <el-tag
                :type="
                  scope.row.status === 'DONE'
                    ? 'success'
                    : scope.row.status === 'PROGRESS'
                    ? 'primary'
                    : 'warning'
                "
                disable-transitions
              >
                {{ scope.row.status }}
              </el-tag>
            </template>
          </el-table-column>
        </el-table>
      </el-col>
      <el-col :span="2">
        <div></div>
      </el-col>
    </el-row>
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
          content:
            'NUXTのアプリからaxios経由でRESTAPIをコールしフロントとバックエンドを繋げる',
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
          content:
            'Javaのイメージにバックエンドのjarをレイヤー化したイメージを作成する',
          status: 'TODO'
        },
        {
          id: '6',
          title: 'フロントエンドのDockerイメージを作成する',
          content:
            'Node.jsからNUXTをインストールしWEBアプリをコピー後にbuildするイメージを作成する',
          status: 'TODO'
        },
        {
          id: '7',
          title: 'Docker-composeでパッケージ化する',
          content:
            'フロントエンド・バックエンド・DBの3層をひとつのパッケージにまとめる',
          status: 'TODO'
        }
      ]
    }
  }
}
</script>
