<template>
  <div
    v-if="state.isShow"
    :class="['sc-bar', state.isMe ? 'isMe' : 'notMe']"
    :style="state.styleObject"
    :draggable="'true'"
    @dragstart="moveStart"
    @dragend="moveEnd"
    @dragover="editting"
    @mouseup="mouseup"
    @mousemove="mousemove"
    @click="$emit('click-event')"
  >
    <span style="float: right; padding: 5px" @click="deleteEvent">✖</span
    ><span class="head">
      <span class="startTime time">{{ startText }}</span
      >～<span class="endTime time">{{ endText }}</span>
    </span>
    <span class="text">{{ contentText }}</span>
    <div
      class="resizable-e"
      :draggable="'true'"
      @dragstart="editStart"
      @dragend="editEnd"
    ></div>
  </div>
</template>

<script>
import { defineComponent, reactive, watch } from "vue";

export default defineComponent({
  name: "reserved-div",
  props: {
    rowIndex: Number,
    keyNo: Number,
    unitWidth: Number,
    unitHeight: Number,
    titleDivWidth: Number,
    borderWidth: Number,
    startText: String,
    endText: String,
    contentText: String,
    minDate: String,
    maxDate: String,
    unit: Number,
    clearSwitch: Boolean,
    dragenterRowIndex: Number,
    dragenterKeyIndex: Number,
    isSelecting: Boolean,
    isSelectingRowIndex: Number,
    isSelectingIndex: Number,
  },
  setup(props, { emit }) {
    const state = reactive({
      styleObject: {
        Opacity: 1,
        left: "0px",
        width: "0px",
        height: "130px",
      },
      isShow: false,
      mouseXStarted: null,
      startLineNo: null,
      endLineNo: null,
      isMe: false,
      isEdit: false,
      isMove: false,
    });

    /**
     * Set the Block left pixel
     *
     * @returns void
     */
    const setLeftPosition = () => {
      let leftDiff = getMinutesDiff(
        new Date(props.minDate),
        new Date(props.startText)
      );
      let shiftCnt = parseInt(leftDiff / props.unit);
      state.startLineNo = shiftCnt;
      let shiftLeft = props.unitWidth * shiftCnt + shiftCnt * props.borderWidth;
      state.styleObject.left = shiftLeft + "px";
    };
    /**
     * Set the Block width pixel
     *
     * @returns void
     */
    const setWidth = () => {
      let rightDiff = getMinutesDiff(
        new Date(props.startText),
        new Date(props.endText)
      );
      let widthCnt = parseInt(rightDiff / props.unit);
      state.endLineNo = state.startLineNo + widthCnt;
      let width = props.unitWidth * widthCnt + widthCnt * props.borderWidth;
      state.styleObject.width = width + "px";
    };
    /**
     * Set the Block to edit status
     *
     * @param Object e Event
     *
     * @returns void
     */
    const editStart = (e) => {
      if (props.isSelecting) {
        e.preventDefault();
        return;
      }
      state.isEdit = true;
      state.mouseXStarted = e.clientX;
      state.styleObject.Opacity = 0.5;
    };
    /**
     * Adjust time event
     *
     * @param Obejct e Event
     *
     * @returns void
     */
    const editting = (e) => {
      if (props.isSelecting) {
        e.preventDefault();
        return;
      }
      if (state.isEdit) {
        if (e) {
          let width = e.x + e.layerX;
          if (state.mouseXStarted > width) {
            state.mouseXStarted = width;
            return;
          }

          // adjust by Mouse X-axio
          let movePx = e.clientX - state.mouseXStarted;
          let unitCnt = parseInt(movePx / props.unitWidth);
          if (unitCnt != 0) {
            state.mouseXStarted = e.clientX;
            emit("edit-schedule-data", props.rowIndex, props.keyNo, unitCnt);
          }
        } else {
          // adjust by current unit-div number
          if (props.dragenterKeyIndex > state.startLineNo) {
            state.mouseXStarted += props.unitWidth;
            let unitCnt = parseInt(props.dragenterKeyIndex - state.endLineNo);
            emit("edit-schedule-data", props.rowIndex, props.keyNo, unitCnt);
          }
        }
      }
    };
    /**
     * End edit and set new data
     *
     * @returns void
     */
    const editEnd = () => {
      if (state.isEdit) {
        emit("edit-event", props.startText, props.endText);
      }
      state.isEdit = false;
      state.styleObject.Opacity = 1;
      state.mouseXStarted = null;
    };
    /**
     * Set the block to move status
     *
     * @param Object e
     */
    const moveStart = (e) => {
      if (props.isSelecting) {
        e.preventDefault();
        return;
      }
      if (!state.isEdit) {
        state.styleObject.Opacity = 0.5;
        state.isMove = true;
        state.mouseXStarted = e.clientX;
        emit("set-dragenter-row-and-index", props.rowIndex, null);
      }
    };
    /**
     * End move and set new data
     *
     * @returns void
     */
    const moveEnd = (e) => {
      let mouseXEnd = e.clientX;
      // Check move status and move block
      if (
        state.isMove &&
        (mouseXEnd != state.mouseXStarted ||
          props.dragenterRowIndex != props.rowIndex)
      ) {
        // get x-axis moved count
        let moveXPx = mouseXEnd - state.mouseXStarted;
        let unitCnt = parseInt(moveXPx / props.unitWidth);
        let halfWidth = parseInt(props.unitWidth / 2);
        let modXPx = parseInt(moveXPx % props.unitWidth);
        if (moveXPx < 0 && Math.abs(modXPx) >= halfWidth) {
          unitCnt--;
        }
        state.mouseXStarted = null;

        if (unitCnt != 0 || props.dragenterRowIndex != props.rowIndex) {
          // result pass to father component's method
          emit("move-schedule-data", props.rowIndex, props.keyNo, unitCnt);
        }
      }

      // Return block all status and opacity
      state.isEdit = false;
      state.isMove = false;
      state.styleObject.Opacity = 1;
    };
    /**
     * Delete this event
     */
    const deleteEvent = () => {
      emit("delete-schedule-data", props.rowIndex, props.keyNo);
    };
    /**
     * Mouse move event for Add new schedule
     */
    const mousemove = (e) => {
      if (
        props.rowIndex == props.isSelectingRowIndex &&
        props.keyNo == props.isSelectingIndex
      ) {
        if (props.isSelecting && state.mouseXStarted) {
          let movePx = e.clientX - state.mouseXStarted;
          let unitCnt = parseInt(movePx / props.unitWidth);
          if (unitCnt != 0 && unitCnt < 0) {
            state.mouseXStarted = e.clientX + props.unitWidth;
            emit("edit-schedule-data", props.rowIndex, props.keyNo, unitCnt);
          }
        }
        if (props.isSelecting && !state.mouseXStarted) {
          state.mouseXStarted = e.clientX;
          state.styleObject.Opacity = 0.5;
        }
      }
    };
    /**
     * Mouse up event for Add new schedule
     */
    const mouseup = () => {
      emit("mouse-up", props.startText, props.endText);
    };
    /**
     * Get minutes diff between date1 and date2
     *
     * @param Object date1 DateObject1
     * @param Object date2 DateObject2
     *
     * @returns Int
     */
    const getMinutesDiff = (date1, date2) => {
      const diffTime = date2 - date1;
      return Math.ceil(diffTime / (1000 * 60));
    };

    watch(
      () => props.startText,
      (newVal, oldVal) => {
        if (newVal != oldVal) {
          setLeftPosition();
        }
      }
    );

    watch(
      () => props.endText,
      (newVal, oldVal) => {
        if (newVal != oldVal) {
          setWidth();
          if (
            props.isSelecting &&
            state.mouseXStarted &&
            props.rowIndex == props.isSelectingRowIndex &&
            props.keyNo == props.isSelectingIndex
          ) {
            let diff = getMinutesDiff(new Date(oldVal), new Date(newVal));
            let cnt = parseInt(diff / props.unit);
            state.mouseXStarted += props.unitWidth * cnt;
          }
        }
      }
    );

    watch(
      () => props.dragenterKeyIndex,
      (newVal, oldVal) => {
        if (
          newVal != oldVal &&
          props.dragenterRowIndex == props.rowIndex &&
          state.isEdit
        ) {
          editting();
        }
      }
    );

    watch(
      () => props.clearSwitch,
      (newVal, oldVal) => {
        if (newVal != oldVal) {
          editEnd();
        }
      }
    );

    if (
      new Date(props.startText) - new Date(props.minDate) < 0 &&
      new Date(props.endText) - new Date(props.minDate) < 0
    ) {
      return;
    }
    setLeftPosition();
    setWidth();
    state.isShow = true;

    return {
      state,
      setLeftPosition,
      setWidth,
      editStart,
      editting,
      editEnd,
      moveStart,
      moveEnd,
      deleteEvent,
      mousemove,
      mouseup,
      getMinutesDiff,
    };
  },
});
</script>

<style scoped>
</style>
