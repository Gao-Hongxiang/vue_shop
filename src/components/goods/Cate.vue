<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="toAddCate">添加商品</el-button>
        </el-col>
      </el-row>
      <tree-table
        class="treeTable"
        border
        :expand-type="false"
        :selection-type="false"
        :show-row-hover="false"
        :show-index="true"
        index-text="#"
        :data="cateList"
        :columns="columns"
      >
        <template v-slot:isok="data">
          <i
            class="el-icon-success"
            v-if="data.row.cat_deleted === false"
            style="color: lightgreen"
          ></i>
          <i class="el-icon-error" v-else style="color: red"></i>
        </template>
        <template v-slot:order="data">
          <el-tag type="primary" v-if="data.row.cat_level === 0">一级</el-tag>
          <el-tag type="success" v-else-if="data.row.cat_level===1">二级</el-tag>
          <el-tag type="warning" v-else>三级</el-tag>
        </template>
        <template v-slot:opt>
          <el-button type="primary" icon="el-icon-edit">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete">删除</el-button>
        </template>
      </tree-table>
      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>
    <!-- 添加商品对话框 -->
    <el-dialog
      title="提示"
      :visible.sync="addCateDialogVisible"
      width="30%"
      @close="addCateDialogCiosed"
    >
      <el-form :model="AddCateForm" :rules="AddCateFormRules" ref="AddCateForm" label-width="100px">
        <el-form-item label="商品名称" prop="cat_name">
          <el-input v-model="AddCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <el-cascader
            change-on-select
            expand-trigger="hover"
            clearable
            v-model="selectKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChange"
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible='false'">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      parentCateList: [],
      selectKeys: [],
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      AddCateForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      AddCateFormRules: {
        cat_name: { required: true, message: '请输入商品名称', trigger: 'blur' }
      },
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      cateList: [],
      total: 0,
      columns: [
        {
          label: '商品名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          type: 'template',
          template: 'isok'
        },
        {
          label: '排序',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'opt'
        }
      ],
      addCateDialogVisible: false
    }
  },
  created() {
    this.geCateList()
  },
  mounted() {},

  methods: {
    async geCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败！')
      }
      // console.log(res.data)
      this.cateList = res.data.result
      this.total = res.data.total
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.geCateList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.geCateList()
    },
    toAddCate() {
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: {
          type: 2
        }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取分类失败')
      }
      // console.log(res.data)
      this.parentCateList = res.data
    },
    parentCateChange() {
      console.log(this.selectKeys)
      if (this.selectKeys.length > 0) {
        this.AddCateForm.cat_pid = this.selectKeys[this.selectKeys.length - 1]
        this.AddCateForm.cat_level = this.selectKeys.length
      } else {
        this.AddCateForm.cat_pid = 0
        this.AddCateForm.cat_level = 0
      }
    },
    addCate() {
      // console.log(this.AddCateForm)
      this.$refs.AddCateForm.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(
          'categories',
          this.AddCateForm
        )
        if (res.meta.status !== 201) {
          return this.$message.error('添加商品分类失败！')
        }
        this.$message.success('添加分类成功！')
        this.geCateList()
        this.addCateDialogVisible = false
      })
    },
    addCateDialogCiosed() {
      this.$refs.AddCateForm.resetFields()
      this.selectKeys = []
      this.AddCateForm.cat_pid = 0
      this.AddCateForm.cat_level = 0
    }
  }
}
</script>

<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}
.el-cascader {
  width: 100%;
}
</style>
