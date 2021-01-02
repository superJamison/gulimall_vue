<template>
  <div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      @node-click="handleNodeClick"
      :expand-on-click-node="false"
      :default-expanded-keys="expandedList"
      show-checkbox
      node-key="catId"
    >
    <span class="custom-tree-node" slot-scope="{ node, data }">
      <span>{{ node.label }}</span>
      <span>
        <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)">
          Append
        </el-button>
        <el-button v-if="node.childNodes.length == 0" type="text" size="mini" @click="() => remove(node, data)">
          Delete
        </el-button>
        <el-button type="text" @click="() => edit(data)">
          edit
        </el-button>
      </span>
    </span>
    </el-tree>

    <el-dialog :title="title" :visible.sync="dialogFormVisible">
      <el-form :model="category">
        <el-form-item label="分类名称:">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标:">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位:">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitForm">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      title: '',
      dialogFormVisible: false,
      menus: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      },
      dialogType: '',
      category: {
        name: '',
        icon: '',
        productUnit: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0
      },
      expandedList: []
    }
  },
  methods: {
    handleNodeClick (data) {
      console.log(data)
    },
    edit(data){
      console.log(data);
      this.dialogType = 'edit'
    },
    getMenus () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then((result) => {
        this.menus = result.data.data
      })
    },
    append (data) {
      console.log('append:', data)
      this.dialogType = 'add'
      this.dialogFormVisible = true
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
    },
    addCategory(){
      console.log("category---->",this.category);
      this.$http({
          url: this.$http.adornUrl('/product/category/save'),
          method: 'post',
          data: this.$http.adornData(this.category, false)
      })
      .then(({data}) => {
        this.dialogFormVisible = false
        this.$message.success('分类添加'+data.msg)
        this.getMenus()
        this.expandedList = [this.category.parentCid]
        this.category = {}
      })
      .catch((result) => {
        this.$message.error('分类添加失败！')
      })
    },
    remove (node, data) {
      console.log("node:", node)
      this.$confirm(`是否删除【${node.data.name}】菜单`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        let ids = [data.catId]
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          this.$message.success('分类删除'+data.msg)
          this.getMenus()
          this.expandedList = [node.parent.data.catId]
        })
      }).catch(() => {

      })
    },
    submitForm() {
      if (this.dialogType === 'add'){
        this.addCategory()
      }else if (this.dialogType === 'edit'){

      }
    }
  },
  created () {
    this.getMenus()
  }
}
</script>

<style scoped>
</style>
