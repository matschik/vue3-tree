# Getting Started

## Installation

Install the `vue3-tree` package:

```sh
npm i vue3-tree
```

Or

```sh
yarn add vue3-tree
```

## Quick Start

```vue
<template>
  <Tree
    :nodes="data"
  />
</template>

<script>
import { ref } from 'vue'
import Tree from 'vue3-tree'

export default {
  components: {
    Tree,
  },
  setup() {
    const data = ref([{
      id: '1',
      label: 'a',
      nodes: [{
        id: '4',
        label: 'aa',
      }, {
        id: '5',
        label: 'ab',
      }],
    }, {
      id: '2',
      label: 'b',
      nodes: [{
        id: '6',
        label: 'ba',
        nodes: [{
          id: '11',
          label: 'aaaa',
          nodes: [{
            id: '15',
            label: 'aaaa',
          },{
            id: '16',
            label: 'bbbb',
          }],
        },{
          id: '12',
          label: 'bbbb',
        }],
      },{
        id: '7',
        label: 'bb',
        nodes: [{
          id: '13',
          label: 'cccc',
        },{
          id: '14',
          label: 'dddd',
        }],
      }],
    }, {
      id: '3',
      label: 'c',
    }])

    return {
      data,
    }
  },
}
</script>
```