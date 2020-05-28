<template>
  <div id="goods">
    <!-- 添加商品按钮 -->
    <el-button @click='goodAdd' class="addProduct" type='success' plain>添加商品</el-button>
    <!-- 表格 -->
    <el-table :data='productList'>
      <el-table-column label='#' type='index' :index='pagenum === 1 ? 1 : pagenum * pagesize + 1'></el-table-column>
      <el-table-column label='商品名称' prop='goods_name'></el-table-column>
      <el-table-column label='商品价格' prop='goods_price'></el-table-column>
      <el-table-column label='商品重量' prop='goods_weight'></el-table-column>
      <el-table-column label='创建时间' prop='add_time'>
        <template v-slot:default="{ row }">
          {{ row.add_time | time}}
        </template>
      </el-table-column>
      <el-table-column label='操作'>
        <template>
          <el-button type='primary' plain icon='el-icon-edit' size='mini'></el-button>
          <el-button type='danger' plain icon='el-icon-delete' size='mini'></el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <el-pagination
      background
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pagenum"
      :page-sizes="[5, 10, 15, 20]"
      :page-size="pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
  </div>
</template>

<script>
export default {
  data () {
    return {
      pagenum: 1,
      pagesize: 5,
      total: 0,
      productList: []
    }
  },
  created () {
    this.getProductList()
  },
  methods: {
    async getProductList () {
      const { meta, data } = await this.$axios.get('goods', {
        params: {
          pagenum: this.pagenum,
          pagesize: this.pagesize
        }
      })
      console.log('getProductList -> data', data)
      if (meta.status === 200) {
        this.productList = data.goods
        this.total = data.total
      } else {
        this.$message.error(meta.msg)
      }
    },
    handleSizeChange (val) {
      this.pagesize = val
      this.pagenum = 1
      this.getProductList()
    },
    handleCurrentChange (val) {
      this.pagenum = val
      this.getProductList()
    },
    goodAdd () {
      this.$router.push('/goods-add')
    }
  }
}
</script>

<style lang='scss'>
  #goods {
    .addProduct {
      margin-bottom: 10px;
    }
    .el-pagination {
      margin-top: 10px;
    }
  }
</style>
