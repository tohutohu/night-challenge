<template>
  <el-container>
    <el-main>
      <div>
        <h1>Tyeptal Custom Response Setting Service</h1>
      </div>
      <el-table :data="dicts">
        <el-table-column prop="id" label="ID" width="60px"></el-table-column>
        <el-table-column label="Request">
          <template slot-scope="scope">
            <template v-if="editing.id !== scope.row.id">{{
              scope.row.request
            }}</template>
            <template v-else>
              <el-input v-model="editing.request"></el-input>
            </template>
          </template>
        </el-table-column>
        <el-table-column label="Response">
          <template slot-scope="scope">
            <template v-if="editing.id !== scope.row.id">{{
              scope.row.response
            }}</template>
            <template v-else>
              <el-input v-model="editing.response"></el-input>
            </template>
          </template>
        </el-table-column>
        <el-table-column label="Operation">
          <template slot-scope="scope">
            <template v-if="editing.id !== scope.row.id">
              <el-button size="small" @click="editData(scope.row)"
                >Edit</el-button
              >
              <el-button
                size="small"
                type="danger"
                @click="confirmDelete(scope.row)"
                >Delete</el-button
              >
            </template>
            <template v-else>
              <template v-if="scope.row.id === 'new'">
                <el-button size="small" @click="createData(scope.row)"
                  >Create</el-button
                >
              </template>
              <template v-else>
                <el-button size="small" @click="updateData(scope.row)"
                  >Update</el-button
                >
              </template>
              <el-button type="danger" size="small" @click="cancelEdit"
                >Cancel</el-button
              >
            </template>
          </template>
        </el-table-column>
      </el-table>
      <div style="margin-top: 1.2em; padding-left: calc(100% - 6em);">
        <el-button
          type="primary"
          :disabled="(dicts[dicts.length - 1] || {}).id === 'new'"
          @click="newData"
          >New</el-button
        >
      </div>
    </el-main>

    <el-dialog title="Confirm" :visible.sync="dialogVisible" width="30%">
      <span>削除してもよろしいですか？</span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">Cancel</el-button>
        <el-button type="danger" @click="deleteData">Delete</el-button>
      </span>
    </el-dialog>
  </el-container>
</template>

<script>
export default {
  data() {
    return {
      dialogVisible: false,
      deletingId: -1
    }
  },
  async asyncData({ $axios }) {
    const dicts = await $axios.$get('/dicts')
    dicts.sort((a, b) => Number(a.id) - Number(b.id))
    return {
      dicts,
      editing: {
        id: -1,
        request: '',
        response: ''
      }
    }
  },
  methods: {
    newData() {
      this.editing = {
        id: 'new',
        request: '',
        response: ''
      }
      this.dicts.push({
        id: 'new',
        request: '',
        response: ''
      })
    },
    editData(data) {
      this.editing = { ...data }
    },
    cancelEdit() {
      this.editing = {
        id: -1,
        request: '',
        response: ''
      }
      this.dicts = this.dicts.filter((data) => data.id !== 'new')
    },
    confirmDelete(data) {
      this.deletingId = data.id
      this.dialogVisible = true
    },
    async deleteData() {
      await this.$axios.$delete(`/dicts/${this.deletingId}`)
      this.dialogVisible = false
      await this.fetchData()
      this.$message({
        message: 'カスタムレスポンスを削除しました',
        type: 'success'
      })
    },
    async updateData() {
      if (this.editing.response === '' || this.editing.request === '') {
        this.$message({
          message: 'リクエストまたはレスポンスが空です',
          type: 'warning'
        })
        return
      }

      await this.$axios.$put(`/dicts/${this.editing.id}`, this.editing)
      await this.fetchData()
      this.$message({
        message: 'カスタムレスポンスを更新しました',
        type: 'success'
      })

      this.editing = {}
    },
    async createData() {
      if (this.editing.response === '' || this.editing.request === '') {
        this.$message({
          message: 'リクエストまたはレスポンスが空です',
          type: 'warning'
        })
        return
      }

      await this.$axios.$post(`/dicts`, this.editing)
      await this.fetchData()
      this.$message({
        message: 'カスタムレスポンスを作成しました',
        type: 'success'
      })

      this.editing = {}
    },
    async fetchData() {
      const dicts = await this.$axios.$get('/dicts')
      this.dicts = dicts
      this.dicts.sort((a, b) => Number(a.id) - Number(b.id))
    }
  }
}
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>
