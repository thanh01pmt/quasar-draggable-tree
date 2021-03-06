<template>
  <div style="border-radius: 3px;"
       :class="hasChildren?'q-tree__node--link':'q-tree__node--link q-treeview-node--leaf'"
  >
    <div style="padding-top: 6px;" class="row q-treeview-node__root" @click="open = !open">
      <q-icon size="sm" v-if="hasChildren" name="arrow_right"
              :class="open?'text-grey-8 q-tree__arrow--rotate':'text-grey-8'"/>
      <slot name="left" v-bind="{ item: value, open }"/>
      <slot v-if="hasDefaultSlot" name="body" v-bind="{ item: value, open }"/>
      <div v-if="!hasDefaultSlot" class="q-tree__node-header-content q-pa-xs">
        {{value.label}}
      </div>
    </div>
    <div
      v-if="open && value.children.length > 0"
      class="q-tree__children"
      style="padding-top: 6px"
    >
      <draggable
        :value="value.children"
        ghost-class="ghost"
        @input="updateValue"
        :group="group"
        v-bind="dragOptions"
        @start="drag = true"
        @end="drag = false"
      >
          <q-draggable-tree-node
            v-for="item,index in value.children"
            :key="index"
            :value="item"
            @input="updateChildValue"
            :group="group"
            :rowKey="rowKey"
          >
            <template v-slot:left="{ item, open }">
              <slot name="left" v-bind="{ item, open }"></slot>
            </template>
            <template v-if="hasDefaultSlot" v-slot:body="{ item, open }">
              <slot name="body" v-bind="{ item, open }"></slot>
            </template>
            <span v-if="!hasDefaultSlot">{{item.label}}</span>
          </q-draggable-tree-node>
      </draggable>
    </div>
    <div v-else class="q-tree__children">
      <draggable
        :value="value.children"
        ghost-class="ghost"
        @input="updateValue"
        :group="group"
        class="dragArea"
        v-bind="dragOptions"
        @start="drag = true"
        @end="drag = false"
      ></draggable>
    </div>
  </div>
</template>

<script>
    import Draggable from "vuedraggable";

    export default {
        name: "QDraggableTreeNode",
        components: {
            Draggable
        },
        props: {
            value: {
                type: Object,
                default: () => ({
                    id: 0,
                    name: "",
                    children: []
                })
            },
            root: {
                type: Boolean,
                default: () => false
            },
            group: {
                type: String,
                default: null
            },
            rowKey: {
                type: String,
                default: 'label'
            },
        },
        data() {
            return {
                open: true,
                drag: false,
                localValue: Object.assign({}, this.value)
            };
        },
        computed: {
            hasChildren() {
                return this.value.children != null && this.value.children.length > 0;
            },
            isDark() {
                return "";
            },
            hasDefaultSlot() {
                return this.$scopedSlots.hasOwnProperty("body");
            },
            dragOptions() {
                return {
                    animation: 200,
                    group: "description",
                    emptyInsertThreshold: 10,
                    disabled: false,
                    ghostClass: "ghost"
                };
            }
        },
        watch: {
            value(value) {
                this.localValue = Object.assign({}, value);
            }
        },
        methods: {
            updateValue(value) {
                if (value.constructor === Array) {
                    this.localValue.children = [...value];
                    this.$emit("input", this.localValue);
                    this.open=true;
                }
            },
            updateChildValue(value) {
                const index = this.localValue.children.findIndex(c => c[this.rowKey] === value[this.rowKey]);
                this.$set(this.localValue.children, index, value);
                this.$emit("input", this.localValue);
            }
        }
    };

</script>

<style scoped>

  .ghost {
    opacity: 0.5;
    background: lightgray;
  }

  .list-group {
    min-height: 20px;
  }

  .list-group-item {
    cursor: move;
  }

  .list-group-item i {
    cursor: pointer;
  }

  .dragArea {
    min-height: 11px;
  }
</style>
