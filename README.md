For Vue.js 2.x see https://github.com/kenberkeley/vue2-drag-and-drop-demo

***

# Vue Demo : Drag + Drop + Sort ([JS Bin](http://jsbin.com/gupebi/embed?html,js,output))
Thanks to [@bramblex](https://github.com/bramblex), the original post is [here](https://www.v2ex.com/t/300047)

`TreeNode.vue` ↓↓↓ (Practical usage: [vue-datatable-component - HeaderSettings.vue](https://github.com/kenberkeley/vue-datatable-component/blob/master/src/DataTable/HeaderSettings.vue))

```vue
<template>
  <!-- [:draggable="{Boolean}"] 空节点不能被拖动，判断依据是是否存在 node.name -->
  <!-- [@dragover.prevent] 必须设置，否则浏览器默认是禁用拖动的 -->
  <div class="tree-node" :class="{ 'empty-node': !node.name }"
    :draggable="!!node.name"
    @dragover.prevent
    @dragstart.stop="handleDragStart"
    @drop.stop="handleDrop"
    @dragenter.stop="handleDragEnter"
    @dragleave.stop="handleDragLeave"
    @dragend.prevent="handleDragEnd">
    {{ node.name }}
    <div v-if="children && children.length" class="tree-node-children">
      <tree-node v-for="child in children"
        :vm.sync="vm" :node="child" :idx="$index">
      </tree-node>
    </div>
  </div>
</template>
<script>
export default {
  name: 'tree-node', // 递归组件需指明 name
  props: {
    vm: { twoWay: true }, // 正在拖动的节点实例（TreeNode 组件通用，须双向绑定）
    node: Object, // 节点数据，形如 { name: 'xxx', children: [] }
    idx: Number // v-for 的索引，用于相邻节点的判别
  },
  computed: {
    children () { // 为每个子节点前后都生成空节点，便于实现兄弟节点间的“插入排序”
      // 举例：原本是 [N1, N2, N3]
      let { children } = this.node
      if (!children || !children.length) return []
      
      let _children = []
      children.forEach(child => _children.push({}, child))
      _children.push({})

      // 最后生成 [E1, N1, E2, N2, E3, N3, E4]（其中 N 表示节点，E 表示空节点）
      return _children
    },
    isParent () { // 拖放限制 1：判断“我”是否为被拖动节点的父节点
      return this === this.vm.$parent
    },
    isNextToMe () { // 拖放限制 2：判断“我”是否为被拖动节点的相邻节点
      return this.$parent === this.vm.$parent && Math.abs(this.idx - this.vm.idx) === 1
    },
    isMeOrMyAncestor () { // 拖放限制 3：判断被拖动节点是否为“我”自身或“我”的祖先
      var p = this
      while (p) {
        if (this.vm === p) return true
        p = p.$parent
      }
    },
    isAllowToDrop () { // 上述拖放限制条件的综合
      return !(this.isParent || this.isNextToMe || this.isMeOrMyAncestor)
    }
  },
  methods: {
    clearBgColor () { // 清理样式
      this.$el.style.backgroundColor = ''
    },
    handleDragStart () {
      this.vm = this // 设置本组件为当前正在拖动的实例，此举将同步 sync 到所有 TreeNode 实例
      this.$el.style.backgroundColor = 'silver'
    },
    handleDrop () {
      this.clearBgColor() // 此时 this 为目的地节点，vm 才是被拖动节点
      if (!this.isAllowToDrop) return

      // 无论如何都直接删除被拖动节点
      this.vm.$parent.node.children.$remove(this.vm.node)

      // 情况 1：拖入空节点，成其兄弟（使用 splice 插入节点）
      if (!this.node.name)
        return this.$parent.node.children.splice(this.idx / 2, 0, this.vm.node)

      // 情况2：拖入普通节点，成为其子
      if (!this.node.children) this.$set('node.children', []) // 须用 $set 引入双向绑定
      this.node.children.push(this.vm.node)
    },
    handleDragEnter () { // 允许拖放才会显示样式
      if (!this.isAllowToDrop) return
      this.$el.style.backgroundColor = 'yellow'
    },
    handleDragLeave () {
      this.clearBgColor()
    },
    handleDragEnd () {
      this.clearBgColor()
    }
  }
}
</script>
<style>
/* 普通节点 */
.tree-node {
  display: list-item;
  list-style: none;
  border-left: 1px dashed gray;
}
/* 空节点 */
.tree-node.empty-node {
  height: .5em;
  list-style-type: none;
}
/* 层次缩进 */
.tree-node-children {
  margin-left: 2em;
}
</style>
```
