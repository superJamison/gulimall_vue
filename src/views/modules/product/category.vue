<template>
  <div>
    <el-switch
      v-model="draggable"
      active-text="开启菜单拖拽"
      style="margin: 0 20px;">
    </el-switch>
    <el-button type="danger" @click="batchDelete">批量删除</el-button>
    <el-tree
      :data="menus"
      :props="defaultProps"
      @node-click="handleNodeClick"
      :expand-on-click-node="false"
      :default-expanded-keys="expandedList"
      :show-checkbox="true"
      node-key="catId"
      :highlight-current="true"
      ref="menusTree"
      :draggable="draggable"
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
      draggable: false,
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
    },
    batchDelete(){
      let catIds = []
      let catNames = []
      let checkMenus = this.$refs.menusTree.getCheckedNodes()
      console.log("checkMenus---->",checkMenus)
      if(checkMenus.length === 0){
        this.$message.error('您还未选择菜单，请选择您要删除的菜单！')
        return
      }
      for (let i = 0; i < checkMenus.length; i++){
        catIds.push(checkMenus[i].catId)
        catNames.push(checkMenus[i].name)
      }
      this.$confirm(`是否批量删除【${catNames}】菜单`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(catIds, false)
        }).then(({data}) => {
          this.$message.success('批量删除'+data.msg)
          this.getMenus()
        })
      }).catch(() => {
        this.$message.error('已取消批量删除！')
      })
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
      this.dialogType = 'add'
      this.dialogFormVisible = true
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
    },
    edit(data){
      console.log("edit-->",data);
      this.dialogType = 'edit'
      this.dialogFormVisible = true
      this.category.catId = data.catId
      this.category.name = data.name
      this.category.icon = data.icon
      this.category.productUnit = data.productUnit
      this.category.parentCid = data.parentCid
    },
    addCategory(){
      this.$http({
          url: this.$http.adornUrl('/product/category/save'),
          method: 'post',
          data: this.$http.adornData(this.category, false)
      })
      .then(({data}) => {
        this.dialogFormVisible = false
        this.$message.success('分类添加'+data.msg)
        this.getMenus()
        this.expandedList.push(this.category.parentCid)
        this.category = {
          name: '',
          icon: '',
          productUnit: '',
          parentCid: 0,
          catLevel: 0,
          showStatus: 1,
          sort: 0
        }
      })
      .catch((result) => {
        this.$message.error('分类添加failed！')
      })
    },
    editCategory(){
      let {catId, name, icon, productUnit} = this.category
      this.$http({
          url: this.$http.adornUrl('/product/category/update'),
          method: 'post',
          data: this.$http.adornData({catId, name, icon, productUnit}, false)
      })
      .then(({data}) => {
        this.dialogFormVisible = false
        this.$message.success('分类修改'+data.msg)
        this.getMenus()
        this.expandedList.push(this.category.parentCid)
        this.category = {
          name: '',
          icon: '',
          productUnit: '',
          parentCid: 0,
          catLevel: 0,
          showStatus: 1,
          sort: 0
        }
      })
      .catch((result) => {
          console.log(result)
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
        this.$message.error('已取消分类删除！')
      })
    },
    submitForm() {
      if (this.dialogType === 'add'){
        this.addCategory()
      }else if (this.dialogType === 'edit'){
        this.editCategory()
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
