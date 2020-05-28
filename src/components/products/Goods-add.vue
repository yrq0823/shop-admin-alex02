<template>
  <div id="goods-add">
    <!-- 步骤条 -->
    <el-steps :active="active" finish-status="success">
      <el-step title="步骤 1" description="基本信息"></el-step>
      <el-step title="步骤 2" description="商品图片"></el-step>
      <el-step title="步骤 3" description="商品内容"></el-step>
    </el-steps>
    <!-- tabs -->
    <el-tabs :tab-position="tabPosition" v-model="activeName" @tab-click="handleClick">
      <el-tab-pane label="基本信息" name='0'>
        <el-form ref="form" :model="goodsInfo" label-width="80px" :rules='rules'>
          <el-form-item label="商品名称" prop='goods_name'>
            <el-input v-model='goodsInfo.goods_name' placeholder='请输入商品名称'></el-input>
          </el-form-item>
           <el-form-item label="商品价格" prop='goods_price'>
            <el-input v-model='goodsInfo.goods_price' placeholder='请输入商品价格'></el-input>
          </el-form-item>
           <el-form-item label="商品重量" prop='goods_weight'>
            <el-input v-model='goodsInfo.goods_weight' placeholder='请输入商品重量'></el-input>
          </el-form-item>
           <el-form-item label="商品数量" prop='goods_number'>
            <el-input v-model='goodsInfo.goods_number' placeholder='请输入商品数量'></el-input>
          </el-form-item>
          <el-form-item label="商品分类">
             <el-cascader :options="options" :props='props' v-model='goodsInfo.goods_cat' clearable></el-cascader>
          </el-form-item>
          <el-button @click='next' type='primary'>下一步</el-button>
        </el-form>
      </el-tab-pane>
      <el-tab-pane label="商品图片" name='1'>
        <el-upload
          :on-preview="handlePictureCardPreview"
          :on-remove="handleRemove"
          :on-success='handleSuccess'
          name='file'
          multiple
          :headers='header'
          action="http://localhost:8888/api/private/v1/upload"
          list-type="picture-card"
          >
          <i slot="default" class="el-icon-plus"></i>
        </el-upload>
        <el-dialog :visible.sync="dialogVisible">
          <img width="100%" :src="dialogImageUrl" alt="">
        </el-dialog>
        <el-button @click='next' type='primary'>下一步</el-button>
      </el-tab-pane>
      <el-tab-pane label="商品内容" name='2'>
        <quill-editor v-model="goodsInfo.goods_introduce"></quill-editor>
        <el-button type='primary' @click='addGoods'>添加</el-button>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
export default {
  data () {
    return {
      dialogImageUrl: '',
      dialogVisible: false,
      activeName: '',
      tabPosition: 'left',
      active: 0,
      goodsInfo: {
        goods_name: '',
        goods_cat: [],
        goods_price: '',
        goods_number: '',
        goods_weight: '',
        goods_introduce: '',
        pics: []
      },
      options: [],
      props: {
        label: 'cat_name',
        children: 'children',
        value: 'cat_id'
      },
      header: {
        Authorization: localStorage.getItem('token')
      },
      rules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: ['blur', 'change'] }
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: ['blur', 'change'] }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: ['blur', 'change'] }
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: ['blur', 'change'] }
        ]
      }
    }
  },
  created () {
    this.getGoodsList()
  },
  methods: {
    handleClick (tab, event) {
      console.log(tab, event)
      this.active = +tab.index
    },
    async getGoodsList () {
      const { meta, data } = await this.$axios.get('categories', {
        params: {
          type: 3
        }
      })
      if (meta.status === 200) {
        this.options = data
      } else {
        this.$message.error(meta.msg)
      }
    },
    next () {
      if (this.active++ > 2) {
        this.active = 0
      }
      this.activeName = '' + this.active
    },
    handleSuccess (res) {
      console.log(res)
      const { meta, data } = res
      if (meta.status === 200) {
        this.goodsInfo.pics.push({
          pic: data.tmp_path
        })
      } else {
        this.$message.error(meta.msg)
      }
    },
    handleRemove (file, fileList) {
      const imgPath = file.response.data.tmp_path
      this.goodsInfo.pics = this.goodsInfo.pics.filter(item => item.pic !== imgPath)
    },
    handlePictureCardPreview (file) {
      this.dialogImageUrl = file.url
      this.dialogVisible = true
    },
    async addGoods () {
      try {
        await this.$refs.form.validate()
        const { meta } = await this.$axios.post('goods', {
          ...this.goodsInfo,
          goods_cat: this.goodsInfo.goods_cat.join(',')
        })
        if (meta.status === 201) {
          this.$message.success(meta.msg)
          this.$router.push('/goods')
        } else {
          this.$message.error(meta.msg)
        }
      } catch (e) {
        console.log(e)
      }
    }
  }
}
</script>

<style lang='scss' scoped>
  .quill-editor {
    background-color: #fff;
    ::v-deep .ql-editor {
      min-height: 300px;
    }
  }
</style>
