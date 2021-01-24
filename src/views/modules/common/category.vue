<template>
  <el-tree
    :data="menus"
    ref="menusTree"
    :props="defaultProps"
    :check-on-click-node="true"
    @node-click="nodeClick"
    node-key="catId">
  </el-tree>
</template>

<script>
    export default {
      name: "category",
      data() {
        return {
          menus: [],
          defaultProps: {
            children: 'children',
            label: 'name'
          },
        }
      },
      methods: {
        getMenus () {
          this.$http({
            url: this.$http.adornUrl('/product/category/list/tree'),
            method: 'get'
          }).then((result) => {
            this.menus = result.data.data
          })
        },
        // tree点击
        nodeClick(data, node, el){
          if (node.data.children.length === 0){
            this.$emit("tree-node-click", data, node, el)
          }
        }
      },
      created() {
        this.getMenus()
      }
    }
</script>

<style scoped>

</style>
