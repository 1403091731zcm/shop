<template>
  <div>
    <el-switch
      v-model="draggable"
      active-text="开启拖拽"
      inactive-text="关闭拖拽">
    </el-switch>
    <el-button v-if="draggable" @click="batchSave">批量保存</el-button>
    <el-button type="danger" @click="batchDelete" >批量删除</el-button>
    <el-tree :data="menus" :props="defaultProps"
             :expand-on-click-node="false" show-checkbox node-key="catId"
             :default-expanded-keys="expandedKey" draggable
             :allow-drop="allowDrop"
             @node-drop="handleDrop"
             :draggable="draggable"
             ref="menuTree">
    <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)">
            Append
          </el-button>
          <el-button
            type="text"
            size="mini"
            @click="() => edit(data)">
            Edit
          </el-button>
          <el-button
            v-if="node.childNodes.length === 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)">
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog :title="title" :visible.sync="dialogVisible" width="30%" :close-on-click-modal="false">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData()">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
export default {
  name: 'category',
  data () {
    return {
      updateNodes: [],
      maxLevel: 0,
      title: '',
      dialogType: '', // edit, add
      category: {name: '', parentCid: 0, catLevel: 0, showStatus: 1, sort: 0, catId: null, icon: '', productUnit: ''},
      menus: [],
      expandedKey: [],
      dialogVisible: false,
      defaultProps: {
        children: 'children',
        label: 'name'
      },
      draggable: false,
      pCid: []
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
    batchDelete () {
      let catIds = []
      let checkedNodes = this.$refs.menuTree.getCheckedNodes()
      console.log('被选中的元素', checkedNodes)
      for (let i = 0; i < checkedNodes.length; i++) {
        catIds.push(checkedNodes[i].catId)
      }
      this.$confirm(`是否批量删除【${catIds}】菜单`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(catIds, false)
        }).then(({data}) => {
          this.$message({
            type: 'success',
            message: '批量删除成功'
          })
          // 刷新出新的菜单
          this.getMenus()
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    batchSave () {
      this.$http({
        url: this.$http.adornUrl('/product/category/update/sort'),
        method: 'post',
        data: this.$http.adornData(this.updateNodes, false)
      }).then(({data}) => {
        this.$message({
          type: 'success',
          message: '菜单顺序修改成功'
        })
        // 刷新出新的菜单
        this.getMenus()
        // 设置需要默认展开的菜单
        this.expandedKey = this.pCid
        this.updateNodes = []
        this.maxLevel = 0
      })
    },
    append (data) {
      console.log('append', data)
      this.dialogType = 'add'
      this.dialogVisible = true
      this.title = '添加分类'
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
      this.category.catId = null
      this.category.icon = ''
      this.category.productUnit = ''
      this.category.name = ''
      this.category.sort = 0
      this.category.showStatus = 1
    },
    // 添加三级分类
    addCategory () {
      console.log('提交的三级分类数据', this.category)
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({data}) => {
        this.$message({
          type: 'success',
          message: '保存成功'
        })
        // 关闭对话框
        this.dialogVisible = false
        // 刷新出新的菜单
        this.getMenus()
        // 设置需要默认展开的菜单
        this.expandedKey = [this.category.parentCid]
      })
    },

    remove (node, data) {
      var ids = [data.catId]
      this.$confirm(`是否删除【${data.name}】菜单`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$message({
          type: 'success',
          message: '删除成功!'
        })
        // 刷新出新的菜单
        this.getMenus()
        // 设置需要默认展开的菜单
        this.expandedKey = [node.parent.data.catId]
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
      this.$http({
        url: this.$http.adornUrl('/product/category/delete'),
        method: 'post',
        data: this.$http.adornData(ids, false)
      }).then(({data}) => {
        console.log('删除成功...')
        this.getMenus()
      })
      console.log('remove', node, data)
    },

    edit (data) {
      console.log('要修改的分类', data)
      this.dialogVisible = true
      this.dialogType = 'edit'
      this.title = '修改分类'

      // 发送请求获取当前节点最新的数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get'
      }).then(({data}) => {
        console.log('要回写的数据', data.data)
        this.category.name = data.data.name
        this.category.catId = data.data.catId
        this.category.icon = data.data.icon
        this.category.productUnit = data.data.productUnit
        this.category.parentCid = data.data.parentCid
        this.category.sort = data.data.sort
        this.category.showStatus = data.data.showStatus
        this.category.catLevel = data.data.catLevel
      })
    },
    // 修改三级分类数据
    editCategory () {
      var {catId, name, icon, productUnit} = this.category
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData({catId, name, icon, productUnit}, false)
      }).then(({data}) => {
        this.$message({
          type: 'success',
          message: '菜单修改成功'
        })
        // 关闭对话框
        this.dialogVisible = false
        // 刷新出新的菜单
        this.getMenus()
        // 设置需要默认展开的菜单
        this.expandedKey = [this.category.parentCid]
      })
    },

    submitData () {
      if (this.dialogType === 'add') {
        this.addCategory()
      }
      if (this.dialogType === 'edit') {
        this.editCategory()
      }
    },

    allowDrop (draggingNode, dropNode, type) {
      console.log('allowDrop:', draggingNode, dropNode, type)
      // 被托动的当前节点，以及所在的父节点的总层数不能大于3
      // 1，被托动的当前节点的总层数
      this.countNodeLevel(draggingNode)
      // 2. 被托动的当前节点的总层数 + 以及所在的父节点的总层数 不大于3
      let deep = Math.abs(this.maxLevel - draggingNode.level + 1)
      console.log('深度', deep)
      if (type === 'inner') {
        return (deep + dropNode.level) <= 3
      } else {
        return (deep + dropNode.parent.level) <= 3
      }
    },

    countNodeLevel (node) {
      // 找到所有子节点，求出最大深度
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxLevel) {
            this.maxLevel = node.childNodes[i].level
          }
          this.countNodeLevel(node.childNodes[i])
        }
      }
    },

    handleDrop (draggingNode, dropNode, type, ev) {
      // 1, 当前节点最新的父节点id
      console.log('handleDrop:', draggingNode, dropNode, type)
      let pCid = 0
      let siblings = null
      if (type === 'before' || type === 'after') {
        pCid = dropNode.parent.data.catId === undefined ? 0 : dropNode.parent.data.catId
        siblings = dropNode.parent.childNodes
      } else {
        pCid = dropNode.data.catId
        siblings = dropNode.childNodes
      }
      this.pCid.push(pCid)
      // 2，当前拖拽节点的最新顺序
      for (let i = 0; i < siblings.length; i++) {
        if (siblings[i].data.catId === draggingNode.data.catId) {
          // 如果遍历的是当前正在拖拽的节点
          let catLevel = draggingNode.level
          if (siblings[i].level !== draggingNode.level) {
            // 当前节点的层级发生变化
            catLevel = siblings[i].level
            // 还要修改当前节点的子节点的层级
            this.updateChildNode(siblings[i])
          }
          this.updateNodes.push({catId: siblings[i].data.catId, sort: i, parentCid: pCid, catLevel: catLevel})
        } else {
          this.updateNodes.push({catId: siblings[i].data.catId, sort: i})
        }
      }
      // 3. 当前拖拽节点的最新层级
      console.log('updateNodes:', this.updateNodes)
    },
    updateChildNode (node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          var currentNode = node.childNodes[i].data
          this.updateNodes.push({catId: currentNode.catId, catLevel: node.childNodes[i].level})
          this.updateChildNode(node.childNodes[i])
        }
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
