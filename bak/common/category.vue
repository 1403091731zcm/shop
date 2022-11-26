<template>
  <el-tree :data="menus" :props="defaultProps"
           node-key="catId"
           ref="menuTree"
           @node-click="nodeClick">
  </el-tree>
</template>

<script>
export default {
  name: 'category',
  data () {
    return {
      menus: [],
      expandedKey: [],
      dialogVisible: false,
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  methods: {
    getMenus () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({data}) => {
        console.log('成功获取菜单....', data.data)
        this.menus = data.data
      })
    },
    nodeClick (data, node, component) {
      console.log('子组件被点击了', data, node, component)
      // 向父组件发送事件
      this.$emit('tree-node-click', data, node, component)
    }
  },
  created () {
    this.getMenus()
  }
}
</script>

<style scoped>

</style>
