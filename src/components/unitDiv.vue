<template>
  <div
    :class="['tl', isBusiness ? 'can-res' : 'cant-res']"
    :style="{ width: width }"
    @mousedown="mousedown"
    @mouseenter="mouseenter"
    @mouseup="mouseup"
    @dragenter="setDragenterRowAndIndex"
  ></div>
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
  name: "unit-div",
  props: {
    rowIndex: Number,
    keyIndex: Number,
    width: String,
    rowData: Object,
    isBusiness: Boolean,
    isSelecting: Boolean,
    isSelectingRowIndex: Number,
  },
  setup(props, { emit }) {
    const mousedown = () => {
      if (!props.isBusiness) {
        return false;
      }
      emit("mouse-down", props.rowIndex, props.keyIndex);
    };
    const mouseenter = () => {
      if (
        !props.isSelecting ||
        props.rowIndex != props.isSelectingRowIndex ||
        !props.isBusiness
      ) {
        return false;
      }
      emit("mouse-enter", props.keyIndex);
    };
    const mouseup = () => {
      emit("mouse-up");
    };
    const setDragenterRowAndIndex = () => {
      if (!props.isBusiness) {
        return false;
      }
      emit("set-dragenter-row-and-index", props.rowIndex, props.keyIndex);
    };
    return {
      mousedown,
      mouseenter,
      mouseup,
      setDragenterRowAndIndex,
    };
  },
});
</script>

<style scoped>
</style>
