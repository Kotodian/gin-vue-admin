<template>
  <div>
    <div class="clearflex">
      <el-button @click="relation" class="fl-right" size="small" type="primary">确 定</el-button>
    </div>
    <el-tree
      :data="menuTreeData"
      :default-checked-keys="menuTreeIds"
      :props="menuDefaultProps"
      default-expand-all
      highlight-current
      node-key="ID"
      ref="menuTree"
      show-checkbox
    ></el-tree>
  </div>
</template>
<script>
import { getBaseMenuTree, getMenuAuthority, addMenuAuthority } from '@/api/menu'

export default {
  name: 'Menus',
  props: {
    row: {
      default: function() {
        return {}
      },
      type: Object
    }
  },
  data() {
    return {
      menuTreeData: [],
      menuTreeIds: [],
      menuDefaultProps: {
        children: 'children',
        label: 'nickName'
      }
    }
  },
  methods: {
    // 关联树 确认方法
    async relation() {
      const checkArr = this.$refs.menuTree.getCheckedNodes(false, true)
      const res = await addMenuAuthority({
        menus: checkArr,
        authorityId: this.row.authorityId
      })
      if (res.success) {
        this.$message({
          type: 'success',
          message: '添加成功!'
        })
      }
    }
  },
  async created() {
    // 获取所有菜单树
    const res = await getBaseMenuTree()
    this.menuTreeData = res.data.menus
    const res1 = await getMenuAuthority({ authorityId: this.row.authorityId })
    const menus = res1.data.menus
    const arr = []
    menus.map(item => {
      // 防止直接选中父级造成全选
      if (!menus.some(same => same.parentId === item.menuId)) {
        arr.push(Number(item.menuId))
      }
    })
    this.menuTreeIds = arr
  }
}
</script>
<style lang="scss">
</style>