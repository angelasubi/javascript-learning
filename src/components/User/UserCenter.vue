<template>
  <div>
    <m-header></m-header>
    <h3>用户个人中心</h3>
    <el-table :data="tableData" border stripe class="e-table">
      <el-table-column type="index" label="序号" width="80"></el-table-column>
      <el-table-column label="书名" prop="book_id.title"></el-table-column>
      <el-table-column label="作者" prop="book_id.author"></el-table-column>
      <el-table-column label="借阅时间">
        <template slot-scope="scope">
          {{ time(scope.row.booked_date) }}
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import storage from 'common/js/store'
import {mapState} from 'vuex'
import mHeader from '../Header.vue'
import formatDate from 'common/js/date'

export default {
  components: {
    mHeader
  },
  data() {
    return {
      tableData: []
    }
  },

  computed: {
    ...mapState(['logout_status'])
  },

  watch: {
    logout_status(v) {
      if (v) {
        this.$router.push({path: '/'})
        this.$store.dispatch('user_name')
      }
    }
  },

  methods: {
    getTableData() {
      this.$http.get('user/user_info', {
        params: {
          user_id: JSON.parse(storage().get('user_id'))
        }
      })
      .then(resp => {
        this.tableData = resp.data.books
      })
    },
    time(t) {
      return formatDate(t)
    }
  },

  created () {
    this.getTableData()
  }
}
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
.e-table
  width: 80%;
  margin: 20px auto
h3
  margin-left: 120px
</style>
