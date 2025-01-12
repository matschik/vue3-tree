<template>
  <div class="tree">
    <ul :style="{'gap': gap + 'px'}" class="tree-list">
      <tree-row
        v-for="node in state.data"
        :ref="'tree-row-' + node.id"
        :key="node.id"
        :node="node"
        :use-checkbox="useCheckbox"
        :use-icon="useIcon"
        :use-row-delete="useRowDelete"
        :show-child-count="showChildCount"
        :indent-size="indentSize"
        :gap="gap"
        :row-hover-background="rowHoverBackground"
        :set-node="setNode"
        :get-node="getNode"
        :update-node="updateNode"
        :expandable="expandable"
        @delete-row="onDeleteRow"
        @node-expanded="onNodeExpanded"
        @checkbox-toggle="onCheckboxToggle"
      >
        <template #checkbox="{ id, node: slotNode, checked, indeterminate }">
          <slot
            name="checkbox"
            :node="slotNode"
            :checked="checked"
            :indeterminate="indeterminate"
            :toggleCheckbox="() => toggleCheckbox(id)"
          />
        </template>
        <template v-if="useIcon" #iconActive>
          <slot name="iconActive" />
        </template>
        <template v-if="useIcon" #iconInactive>
          <slot name="iconInactive" />
        </template>
        <template v-if="useRowDelete" #deleteIcon>
          <slot name="deleteIcon" />
        </template>
        <template v-if="showChildCount" #childCount="{ count, checkedCount, childs }">
          <slot
            name="childCount"
            :count="count"
            :checkedCount="checkedCount"
            :childs="childs"
          />
        </template>
      </tree-row>
    </ul>
  </div>
</template>

<script>
import { watch, reactive, onMounted } from 'vue';
import TreeRow from './TreeRow.vue';
import initData from '../composables/initData';
import useSearch from '../composables/useSearch';
import {
  setNodeById,
  getNodeById,
  updateNodeById,
  updateNodes,
  removeNodeById,
  expandNodeWithChilds,
} from '../utils';

export default {
  name: 'Tree',
  components: {
    TreeRow,
  },
  props: {
    nodes: {
      type: Array,
      required: true,
    },
    props: {
      type: Object,
      default() {
        return {
          nodes: 'nodes',
          label: 'label',
        };
      },
    },
    indentSize: {
      type: Number,
      default: 24,
    },
    gap: {
      type: Number,
      default: 10,
    },
    searchText: {
      type: String,
      default: '',
    },
    expandRowByDefault: {
      type: Boolean,
      default: false,
    },
    expandAllRowsOnSearch: {
      type: Boolean,
      default: true,
    },
    useCheckbox: {
      type: Boolean,
      default: false,
    },
    useIcon:{
      type: Boolean,
      default: true,
    },
    useRowDelete:{
      type: Boolean,
      default: false,
    },
    showChildCount: {
      type: Boolean,
      default: false,
    },
    rowHoverBackground: {
      type: String,
      default: '#e0e0e0',
    },
    expandable: {
      type: Boolean,
      default: true,
    },
  },
  emits: ['nodeExpanded', 'checkboxToggle', 'update'],
  setup(props, { emit }) {
    const { search } = useSearch();
    const state = reactive({ data: initData(props.nodes) });

    const setNode = (id, node) => {
      state.data.value = setNodeById(state.data, id, node);
    };

    const getNode = id => {
      return getNodeById(state.data, id);
    };

    const updateNode = (id, data) => {
      state.data = updateNodes(updateNodeById(state.data, id, data));
    };

    const toggleCheckbox = id => {
      const { checked } = getNode(id);
      updateNode(id, { checked: !checked });
    };

    watch(()=> state.data, ()=>{
      emit('update', state.data);
    });

    const handleSearch = () => {
      let newData = state.data;
      if (props.searchText !== '') {
        newData = search(props.nodes, props.searchText);
        if (props.expandAllRowsOnSearch) {
          newData.forEach(expandNodeWithChilds);
        }
      } else {
        newData = props.nodes;
      }
      state.data = updateNodes(newData);
    };

    watch(() => props.searchText, handleSearch);

    const onNodeExpanded = (node, state) => {
      emit('nodeExpanded', node, state);
    };

    const onCheckboxToggle = context => {
      emit('checkboxToggle', context);
    };

    const onUpdate = () => {
      emit('update', state.data);
    };

    onMounted(()=> {
      if (props.searchText) {
        handleSearch();
      }
    });

    const onDeleteRow = node => {
      removeNodeById(state.data, node.id);
      state.data = updateNodes(removeNodeById(state.data, node.id));
    };

    return {
      state,
      setNode,
      getNode,
      updateNode,
      onNodeExpanded,
      onCheckboxToggle,
      onUpdate,
      toggleCheckbox,
      onDeleteRow,
    };
  },
};
</script>

<style lang="scss" scoped>
.tree {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
}

.tree-list {
  margin: 0;
  padding: 0;
  overflow: hidden;

  .tree-row{
    padding-left: 0px !important;
  }
}
</style>
