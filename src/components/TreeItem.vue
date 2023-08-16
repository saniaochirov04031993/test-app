<template>
  <div class="tree-item" >
    <div :style="treeItemStyle">
      <div class="tree-item__content" :style="treeItemContentStyle" @click="$emit('update', { ...item })">
        <img v-if="item.is_childless" alt="has no child" width="15" height="15" src="../assets/dash.svg">
        <img v-else-if="item.is_open" alt="opened" width="15" height="15" src="../assets/chevron-down.svg">
        <img v-else alt="closed" width="15" height="15" src="../assets/chevron-right.svg">
        <div class="tree-item__content__title">{{ title }}</div>        
      </div>
    </div>
  </div>
</template>

<script>
import { computed } from 'vue'

const COLORS = {
  FIRST: '#BDBDBD',
  MIDDLE: '#E0E0E0',
  LAST: '#EEEEEE'
}

export default {
  name: 'TreeItem',
  props: ['item', 'index'],
  setup (props) {
    const treeItemContentStyle = computed(() => ({
        transform: `translate(${props.item.level * 15}px)`
    }))
    const treeItemStyle = computed(() => ({
      'background-color': props.item.parent_id === null ? COLORS.FIRST : props.index % 2 === 0 ? COLORS.MIDDLE : COLORS.LAST
    }))
    return { treeItemContentStyle, treeItemStyle, title: props.item.title }
  }
}
</script>

<style scoped>
.tree-item {
  width: 400px;
  .tree-item__content {
    display: flex;
    align-items: center;
    padding: 0.5rem;
    .tree-item__content__title {
      margin-left: 0.5rem;
      font-size: large;
    }
  }
}
</style>
