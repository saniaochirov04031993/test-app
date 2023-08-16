<template>
  <tree-item
    v-for="(item, index) of activeItems"
    :item="item"
    :key="item.id"
    :index="index"
    @update="update($event)"
  ></tree-item>
</template>

<script>
import { ref } from 'vue'
import axios from 'axios'
import TreeItem from './components/TreeItem'

export default {
  name: 'App',
  components: {
    TreeItem
  },
  setup () {
    const items = ref([])
    const activeItems = ref([])
    const treeSimple = new Map()
    const fetchItems = () => {
      axios
        .get('https://64b4c8450efb99d862694609.mockapi.io/tree/items')
        .then(response => {
          const data = response.data.map(({ id, parent_id }) => [parent_id, id])
          data.forEach((item) => {
            if (treeSimple.has(item[0])) {
              treeSimple.set(item[0], [...treeSimple.get(item[0]), item[1]])
            } else {
              treeSimple.set(item[0], [item[1]])
            }
          })
          items.value = response.data.map((item) => ({
            ...item,
            is_open: false,
            is_childless: !treeSimple.has(item.id),
            level: item.parent_id !== null ? 0 : null
          }))
          activeItems.value = items.value.filter(({ parent_id }) => parent_id === null)
        })
    }

    fetchItems()

    const findItem = (id, list) => list.find((item) => item.id === id)

    const findIndex = (id, list) => list.findIndex((item) => item.id === id)

    const updateIsOpen = (id) => {
      const currentItem = findItem(id, activeItems.value)
      const item = findItem(id, items.value)
      currentItem.is_open = !currentItem.is_open
      item.is_open = currentItem.is_open
    }

    const hide = (id) => {
      const childrenIds = treeSimple.get(id) || []
      activeItems.value = activeItems.value.filter(({ id }) => !childrenIds.includes(id))
      childrenIds.forEach(itemId => hide(itemId))
    }

    const open = (parentIndex, parentId, parentLevel) => {
      const itemsForShow = items.value
        .filter(item => item.parent_id === parentId )
        .map((item) => ({ ...item, level: parentLevel + 1 }))
      activeItems.value.splice(parentIndex + 1, 0, ...itemsForShow)    
      itemsForShow.forEach(child => {
        if (child.is_open) {
          const childIndex = findIndex(child.id, activeItems.value)
          open(childIndex, child.id, child.level)
        }
      })
    }

    const update = ({ id, is_open, level }) => {
      updateIsOpen(id)
      is_open ? hide(id) : open(findIndex(id, activeItems.value), id, level)
    }

    return { items, activeItems, update }
  }
}
</script>
