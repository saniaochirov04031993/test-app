<template>
  <tree-item
    v-for="(item, index) of currentItems"
    :item="item"
    :key="item.id"
    :color="getColor(item, index)"
    @update="update($event)"
  ></tree-item>
</template>

<script>
import { ref } from 'vue'
import axios from 'axios'
import TreeItem from './components/TreeItem'

const COLORS = {
  FIRST: '#BDBDBD',
  MIDDLE: '#E0E0E0',
  LAST: '#EEEEEE'
}

export default {
  name: 'App',
  components: {
    TreeItem
  },
  setup () {
    let relationships = new Set()
    const items = ref([])
    const currentItems = ref([])

    const fetchItems = () => {
      axios
        .get('https://64b4c8450efb99d862694609.mockapi.io/tree/items')
        .then(response => {
          relationships = new Set(response.data.filter(({ parent_id }) => parent_id !== null).map((item) => item.parent_id))
          items.value = response.data.map((item) => ({
            ...item,
            is_open: false,
            is_childless: !relationships.has(item.id),
            level: item.parent_id !== null ? 0 : null
          }))
          currentItems.value = items.value.filter(({ parent_id }) => parent_id === null)
        })
    }

    fetchItems()

    const getColor = (item, index) => item.parent_id === null ? COLORS.FIRST : index % 2 === 0 ? COLORS.MIDDLE : COLORS.LAST

    const update = ({ id, is_open, level }) => {
      const parentIndex = currentItems.value.findIndex((item) => item.id === id)
      currentItems.value[parentIndex].is_open = !currentItems.value[parentIndex].is_open
      if (is_open) {
        currentItems.value = currentItems.value.filter(({ parent_id }) => parent_id !== id)
      } else {
        const itemsForShow = items.value
          .filter(item => item.parent_id === id )
          .map((item) => ({ ...item, level: level + 1 }))
        currentItems.value.splice(parentIndex + 1, 0, ...itemsForShow)
      }
    }
    return { items, currentItems, getColor, update }
  }
}
</script>
