<!---
 time-slot-selector
 
 Experience the Ultimate in Scheduling Flexibility with a Drag and Drop Time Slot Selector Built on Vue 3.0
 
 @date   2022/10/11
 @author Xun Zenghui(1730932805@qq.com)
 @see https://github.com/xzh1730932805/timeslotselector

 (c) 2022 Xun Zenghui
 Released under the MIT License.
--->

<template>
  <div class="schedule" @dragover="disableDragendAnimation">
    <div>
      <div
        class="sc-rows"
        :style="{
          width: state.settingData.titleDivW + '%',
          height: state.contentH + 'px',
        }"
      >
        <div class="sc-rows-scroll" :style="{ width: '100%' }">
          <div
            v-for="(row, index) in scheduleData"
            :key="index"
            :class="'timeline title'"
            :style="{
              height: state.settingData.rowH + 'px',
              'padding-top': index == 0 ? state.padding + 'px' : '0px',
            }"
            @click="$emit('row-click-event', index, row.title)"
          >
            <span style="cursor: pointer">{{ row.title }}</span>
          </div>
        </div>
      </div>
      <div
        class="sc-main-box"
        :style="{ width: 100 - state.settingData.titleDivW + '%' }"
      >
        <div class="sc-main-scroll" :style="{ width: state.contentW + 'px' }">
          <div class="sc-main">
            <div
              class="timeline"
              :style="{
                height: state.settingData.dateDivH + 'px',
                background: 'black',
              }"
            >
              <div
                v-for="n in state.dateCnt"
                :key="n"
                class="sc-time"
                :style="{ width: state.dateDivW + 'px', cursor: 'pointer' }"
                @click="$emit('date-click-event', getHeaderDate(n - 1))"
              >
                {{ getHeaderDate(n - 1) }}
              </div>
            </div>
            <div
              class="timeline"
              :style="{
                height: state.settingData.timeDivH + 'px',
                background: '#6187AE',
              }"
            >
              <div
                v-for="n in state.dateCnt * 24"
                :key="n"
                class="sc-time"
                :style="{ width: state.timeDivW + 'px' }"
              >
                {{ getHeaderTime(n - 1) }}
              </div>
            </div>
            <div
              v-for="(row, index) in scheduleData"
              :key="index"
              :class="'timeline'"
              :style="{ height: state.settingData.rowH + 'px' }"
              @dragenter="setDragenterRow(index)"
            >
              <unit-div
                v-for="n in state.unitCnt"
                :key="'unit' + n"
                :row-index="index"
                :key-index="n"
                :row-data="row"
                :is-business="isBusiness(index, n - 1)"
                :is-selecting="state.isSelecting"
                :is-selecting-row-index="state.isSelectingRowIndex"
                :width="state.settingData.unitDivW + 'px'"
                @mouse-down="selectStartTime"
                @mouse-enter="adjustTimeRange"
                @mouse-up="selectEndTime"
                @set-dragenter-row-and-index="setDragenterRowAndIndex"
              ></unit-div>
              <reserved-div
                v-for="(detail, keyNo) in row.schedule"
                :key="'res' + keyNo"
                :schedule-detail="detail"
                :row-index="index"
                :key-no="keyNo"
                :start-text="detail.start"
                :end-text="detail.end"
                :content-text="detail.text"
                :unit-width="state.settingData.unitDivW"
                :unit-height="state.settingData.rowH"
                :title-div-width="state.settingData.titleDivW"
                :border-width="state.settingData.borderW"
                :min-date="state.settingData.startDate"
                :max-date="state.settingData.endDate"
                :unit="state.settingData.unit"
                :clear-switch="state.clearSwitch"
                :dragenter-row-index="state.dragenterRowIndex"
                :dragenter-key-index="state.dragenterKeyIndex"
                :is-selecting="state.isSelecting"
                :is-selecting-row-index="state.isSelectingRowIndex"
                :is-selecting-index="state.isSelectingIndex"
                @set-dragenter-row-and-index="setDragenterRowAndIndex"
                @move-schedule-data="moveScheduleData"
                @edit-schedule-data="editScheduleData"
                @delete-schedule-data="deleteScheduleData"
                @mouse-up="selectEndTime"
                @move-event="$emit('move-event')"
                @edit-event="$emit('edit-event', detail.start, detail.end)"
                @click-event="
                  $emit(
                    'click-event',
                    detail.start,
                    detail.end,
                    detail.text,
                    detail.data
                  )
                "
              ></reserved-div>
            </div>
          </div>
        </div>
      </div>
      <br class="clear" />
    </div>
  </div>
</template>

<script>
/* eslint-disable */
import { defineComponent, reactive, watch } from "vue";
import UnitDiv from "./unitDiv.vue";
import ReservedDiv from "./reservedDiv.vue";

export default defineComponent({
  name: "time-slot-selector",
  components: {
    "unit-div": UnitDiv,
    "reserved-div": ReservedDiv,
  },
  props: {
    scheduleData: {
      type: Array,
    },
    setting: {
      type: Object,
    },
  },
  setup(props, { emit }) {
    const state = reactive({
      settingData: {
        startDate: "2022/04/20",
        endDate: "2022/04/26",
        weekdayText: ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"],
        unit: 60,
        borderW: 1,
        dateDivH: 25,
        timeDivH: 25,
        unitDivW: 25,
        titleDivW: 200,
        rowH: 135,
      },
      padding: 0,
      timeDivW: 0,
      dateDivW: 0,
      contentH: 0,
      contentW: 0,
      dateCnt: 0,
      unitCnt: 0,
      isSelecting: false,
      isSelectingRowIndex: null,
      isSelectingIndex: null,
      dragenterRowIndex: null,
      dragenterKeyIndex: null,
      clearSwitch: false,
    });

    /**
     * Draggable Enter Event
     *
     * @param int rowIndex Row Index
     *
     * @returns void
     */
    const setDragenterRow = (rowIndex) => {
      state.dragenterRowIndex = rowIndex;
    };
    /**
     * Draggable Enter Event
     *
     * @param int rowIndex     Row Index
     * @param int currentIndex Current Index
     *
     * @returns void
     */
    const setDragenterRowAndIndex = (rowIndex, currentIndex) => {
      state.dragenterRowIndex = rowIndex;
      state.dragenterKeyIndex = currentIndex;
    };
    /**
     * Disable HTML5 DragEnd animation and Set mouse position
     *
     * @param Obejct e Event
     */
    const disableDragendAnimation = (e) => {
      e.preventDefault();
    };
    /**
     * Get header area date-text
     *
     * @param Int n Col index
     *
     * @returns String
     */
    const getHeaderDate = (n) => {
      let startDate = addDays(new Date(state.settingData.startDate), n);
      return dateFormatter(startDate, true);
    };
    /**
     * Get header area time-text
     *
     * @param {*} n Col index
     */
    const getHeaderTime = (n) => {
      return n % 24;
    };
    /**
     * Get diff day between date1 and date2
     *
     * @param Object date1 DateObejct1
     * @param Object date2 DateObejct2
     *
     * @returns Int
     */
    const getDateDiff = (date1, date2) => {
      const diffTime = Math.abs(date2 - date1);
      return Math.ceil(diffTime / (1000 * 60 * 60 * 24));
    };
    /**
     * Custom date formatter
     *
     * @param Obejct  dateObj     DateObejct
     * @param Boolean withWeekDay If u need weekday, set True
     *
     * @returns String
     */
    const dateFormatter = (dateObj, withWeekDay) => {
      let year = dateObj.getFullYear();
      let month = dateObj.getMonth() + 1;
      if (month < 10) {
        month = "0" + month;
      }
      let date = dateObj.getDate();
      if (withWeekDay) {
        let day = dateObj.getDay();
        let dayText = state.settingData.weekdayText[day];
        return year + "/" + month + "/" + date + "(" + dayText + ")";
      }
      return year + "/" + month + "/" + date;
    };
    /**
     * Custom datetime formatter
     *
     * @param Obejct  dateObj     DateObejct
     *
     * @returns String
     */
    const datetimeFormatter = (dateObj) => {
      let year = dateObj.getFullYear();
      let month = dateObj.getMonth() + 1;
      if (month < 10) {
        month = "0" + month;
      }
      let date = dateObj.getDate();
      let hours = dateObj.getHours();
      if (hours < 10) {
        hours = "0" + hours;
      }
      let minutes = dateObj.getMinutes();
      if (minutes < 10) {
        minutes = "0" + minutes;
      }
      return year + "/" + month + "/" + date + " " + hours + ":" + minutes;
    };
    /**
     * Add days to object
     *
     * @param Obejct dateObj DateObject
     * @param Int    n       Add number
     *
     * @returns Object
     */
    const addDays = (dateObj, n) => {
      dateObj.setTime(dateObj.getTime() + n * 60 * 60 * 24 * 1000);
      return dateObj;
    };
    /**
     * Add minutes to date object
     *
     * @param Obejct dateObj DateObject
     * @param Int    n       Add number
     *
     * @returns Object
     */
    const addMinutes = (dateObj, n) => {
      dateObj.setTime(dateObj.getTime() + n * 60 * 1000);
      return dateObj;
    };
    /**
     * Check this div is business or not
     *
     * @param Int rowIndex Row index
     * @param Int n        Col index
     *
     * @returns Boolean
     */
    const isBusiness = (rowIndex, n) => {
      // first check this div business day
      let startDate = new Date(state.settingData.startDate);
      let oneDayCnt = parseInt(1440 / state.settingData.unit);
      let thisDate = addDays(startDate, parseInt(n / oneDayCnt));
      let noBusinessDate = props.scheduleData[rowIndex].noBusinessDate;
      if (noBusinessDate.indexOf(dateFormatter(thisDate)) >= 0) {
        // today not business day
        return false;
      }

      // and check this div time bussiness hour
      let weekDay = thisDate.getDay();
      let businessHour = props.scheduleData[rowIndex].businessHours[weekDay];
      if (businessHour.start == "00:00" && businessHour.end == "24:00") {
        // alltime
        return true;
      }

      // has bussiness hour
      let businessStartTime = businessHour.start.replace(":", "");
      let businessEndTime = businessHour.end.replace(":", "");
      let dateMod = n % oneDayCnt;
      let divStartCnt = dateMod * state.settingData.unit;
      let divStartHour = parseInt(divStartCnt / 60);
      if (divStartHour < 10) {
        divStartHour = "0" + divStartHour;
      }
      let divStartMin = parseInt(divStartCnt % 60);
      if (divStartMin < 10) {
        divStartMin = "0" + divStartMin;
      }
      let divStartTime = divStartHour + "" + divStartMin;
      if (divStartTime >= businessStartTime && divStartTime < businessEndTime) {
        return true;
      }
      return false;
    };
    /**
     * Check this range is business or not
     *
     * @param Int    rowIndex      Row index
     * @param String startDateText StartDate text
     * @param String endDateText   EndDate text
     *
     * @returns Boolean
     */
    const isBusinessOnRange = (rowIndex, startDateText, endDateText) => {
      let startDiff = getMinutesDiff(
        new Date(state.settingData.startDate),
        new Date(startDateText)
      );
      let startCnt = parseInt(startDiff / state.settingData.unit);
      let endDiff = getMinutesDiff(
        new Date(state.settingData.startDate),
        new Date(endDateText)
      );
      let endCnt = parseInt(endDiff / state.settingData.unit);
      let result = true;
      for (var i = startCnt; i < endCnt; i++) {
        if (!isBusiness(rowIndex, i)) {
          result = false;
        }
      }
      return result;
    };
    /**
     * Check this range has other event
     *
     * @param Int    index         Data index
     * @param Int    oldRowIndex   Old row index
     * @param Int    newRowIndex   New row index
     * @param String startDateText StartDate text
     * @param String endDateText   EndDate text
     *
     * @returns Boolean
     */
    const hasOtherEvent = (
      index,
      oldRowIndex,
      newRowIndex,
      startDateText,
      endDateText
    ) => {
      for (
        var n = 0;
        n < props.scheduleData[newRowIndex].schedule.length;
        n++
      ) {
        if (n != index || oldRowIndex != newRowIndex) {
          let diffData = props.scheduleData[newRowIndex].schedule[n];
          if (
            new Date(diffData.start) - new Date(startDateText) >= 0 &&
            new Date(diffData.end) - new Date(endDateText) <= 0
          ) {
            return true;
          }
          if (
            new Date(diffData.start) - new Date(startDateText) >= 0 &&
            new Date(diffData.start) - new Date(endDateText) < 0
          ) {
            return true;
          }
          if (
            new Date(diffData.start) - new Date(startDateText) <= 0 &&
            new Date(diffData.end) - new Date(startDateText) > 0
          ) {
            return true;
          }
        }
      }
      return false;
    };
    /**
     * The column are click start event
     *
     * @param Object e Event
     *
     * @returns void
     */
    const selectStartTime = (rowIndex, keyIndex) => {
      state.isSelecting = true;
      state.isSelectingRowIndex = rowIndex;
      let addMinutes1 = (keyIndex - 1) * state.settingData.unit;
      let addMinutes2 = keyIndex * state.settingData.unit;
      let newStartDateObj = addMinutes(
        new Date(state.settingData.startDate),
        addMinutes1
      );
      let newEndDateObj = addMinutes(
        new Date(state.settingData.startDate),
        addMinutes2
      );
      props.scheduleData[rowIndex].schedule.push({
        text: "New",
        start: datetimeFormatter(newStartDateObj),
        end: datetimeFormatter(newEndDateObj),
      });
      state.isSelectingIndex =
        props.scheduleData[state.isSelectingRowIndex].schedule.length - 1;
    };
    /**
     * New event adjust event
     *
     * @param int keyIndex
     */
    const adjustTimeRange = (keyIndex) => {
      let targetIndex =
        props.scheduleData[state.isSelectingRowIndex].schedule.length - 1;
      let targetData =
        props.scheduleData[state.isSelectingRowIndex].schedule[targetIndex];
      if (targetData) {
        let addMinutes1 = keyIndex * state.settingData.unit;
        let newEndDateObj = addMinutes(
          new Date(state.settingData.startDate),
          addMinutes1
        );
        let newEndDateText = datetimeFormatter(newEndDateObj);
        let isPermission = true;

        // Check other event
        if (
          hasOtherEvent(
            targetIndex,
            state.isSelectingRowIndex,
            state.isSelectingRowIndex,
            targetData.start,
            newEndDateText
          )
        ) {
          isPermission = false;
        }

        // Check Businessday
        if (isPermission) {
          isPermission = isBusinessOnRange(
            state.isSelectingRowIndex,
            targetData.start,
            newEndDateText
          );
        }
        if (isPermission) {
          targetData.end = newEndDateText;
        }
      }
    };
    /**
     * Add new block event
     *
     * @param Boolean isAdd     Is add event action
     * @param String  startDate StartDate text
     * @param String  endDate   EndDate text
     *
     * @returns void
     */
    const selectEndTime = (startDate, endDate) => {
      if (state.isSelecting) {
        if (startDate == undefined) {
          let targetData =
            props.scheduleData[state.isSelectingRowIndex].schedule[
              props.scheduleData[state.isSelectingRowIndex].schedule.length - 1
            ];
          startDate = targetData.start;
          endDate = targetData.end;
        }
        emit("add-event", state.isSelectingRowIndex, startDate, endDate);
      }
      state.isSelecting = false;
      state.isSelectingRowIndex = null;
      state.isSelectingIndex = null;
      state.clearSwitch = !state.clearSwitch;
    };
    /**
     * Edit Schedule Datetime Text
     *
     * @param int rowIndex     Row Index
     * @param int keyNo        Key
     * @param int unitCnt      Moved unit count
     *
     * @returns void
     */
    const moveScheduleData = (rowIndex, keyNo, unitCnt) => {
      let targetData = props.scheduleData[rowIndex].schedule[keyNo];
      if (targetData) {
        let status = 0;
        let isBusinessFlag = true;
        let isBusinessChecked = false;
        let changeDatetimeText = (datetimeText) => {
          let addMinutes1 = unitCnt * state.settingData.unit;
          let dateObj = new Date(datetimeText);
          let newDateObj = addMinutes(dateObj, addMinutes1);
          return datetimeFormatter(newDateObj);
        };
        let newStartDatetime = changeDatetimeText(targetData.start);
        let newEndDatetime = changeDatetimeText(targetData.end);

        if (unitCnt != 0) {
          if (
            hasOtherEvent(
              keyNo,
              rowIndex,
              state.dragenterRowIndex,
              newStartDatetime,
              newEndDatetime
            )
          ) {
            status = 2;
          } else {
            isBusinessFlag = isBusinessOnRange(
              state.dragenterRowIndex,
              newStartDatetime,
              newEndDatetime
            );
            if (isBusinessFlag) {
              targetData.start = newStartDatetime;
              targetData.end = newEndDatetime;
              status = 1;
            }
            isBusinessChecked = true;
          }
        }
        if (
          rowIndex != state.dragenterRowIndex &&
          props.scheduleData[state.dragenterRowIndex]
        ) {
          if (isBusinessChecked && !isBusinessFlag) {
            emit("move-event", status);
            return;
          }
          if (!isBusinessChecked && isBusinessFlag) {
            if (
              hasOtherEvent(
                keyNo,
                rowIndex,
                state.dragenterRowIndex,
                newStartDatetime,
                newEndDatetime
              )
            ) {
              status = 2;
              emit("move-event", status);
              return;
            }
            isBusinessFlag = isBusinessOnRange(
              state.dragenterRowIndex,
              targetData.start,
              targetData.end
            );
            isBusinessChecked = true;
          }
          if (isBusinessChecked && isBusinessFlag) {
            props.scheduleData[state.dragenterRowIndex].schedule.push(
              targetData
            );
            props.scheduleData[rowIndex].schedule.splice(keyNo, 1);
            status = 1;
          }
        }
        emit("move-event", status, targetData.start, targetData.end);
      }
    };
    /**
     * Edit Schedule Datetime Text
     *
     * @param int rowIndex  Row
     * @param int keyNo     Key
     * @param int unitCnt   Moved unit count
     *
     * @returns void
     */
    const editScheduleData = (rowIndex, keyNo, unitCnt) => {
      let targetData = props.scheduleData[rowIndex].schedule[keyNo];
      if (targetData) {
        let changeDatetimeText = (datetimeText) => {
          let addMinutes1 = unitCnt * state.settingData.unit;
          let dateObj = new Date(datetimeText);
          let newDateObj = addMinutes(dateObj, addMinutes1);
          return datetimeFormatter(newDateObj);
        };
        let newEndText = changeDatetimeText(targetData.end);
        if (
          hasOtherEvent(keyNo, rowIndex, rowIndex, targetData.start, newEndText)
        ) {
          return;
        }
        targetData.end = newEndText;
      }
    };
    /**
     * Delete Schedule
     *
     * @param int rowIndex  Row
     * @param int keyNo     Key
     *
     * @returns void
     */
    const deleteScheduleData = (rowIndex, keyNo) => {
      props.scheduleData[rowIndex].schedule.splice(keyNo, 1);
      emit("delete-event", rowIndex, keyNo);
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
      const diffTime = Math.abs(date2 - date1);
      return Math.ceil(diffTime / (1000 * 60));
    };

    state.settingData = Object.assign(state.settingData, props.setting);
    state.dateCnt =
      getDateDiff(
        new Date(state.settingData.startDate),
        new Date(state.settingData.endDate)
      ) + 1;
    let oneDayCnt = parseInt(1440 / state.settingData.unit);
    state.unitCnt = oneDayCnt * state.dateCnt;
    state.padding =
      state.settingData.dateDivH +
      state.settingData.timeDivH +
      state.settingData.borderW * 4;
    state.dateDivW =
      state.settingData.unitDivW * oneDayCnt +
      (oneDayCnt - state.settingData.borderW);
    state.contentH =
      state.padding +
      (state.settingData.rowH + state.settingData.borderW * 2) *
        props.scheduleData.length;
    state.contentW =
      state.dateDivW * state.dateCnt +
      state.dateCnt * state.settingData.borderW;
    state.timeDivW =
      (60 / state.settingData.unit) *
        (state.settingData.unitDivW + state.settingData.borderW) -
      1;

    return {
      state,
      setDragenterRow,
      setDragenterRowAndIndex,
      disableDragendAnimation,
      getHeaderDate,
      getHeaderTime,
      getDateDiff,
      dateFormatter,
      datetimeFormatter,
      addDays,
      addMinutes,
      isBusiness,
      isBusinessOnRange,
      hasOtherEvent,
      selectStartTime,
      adjustTimeRange,
      selectEndTime,
      moveScheduleData,
      editScheduleData,
      deleteScheduleData,
      getMinutesDiff,
    };
  },
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.clear {
  clear: both;
  height: 0;
  line-height: 0;
}

.sc-rows {
  float: left;
  font-weight: bold;
  background: #6187ae;
  border-color: #c0c0c0;
  color: white;
  position: relative;
}

.sc-rows .title {
  background: #6187ae;
}

.sc-rows .sc-rows-scroll {
  position: absolute;
  left: 0;
  top: 0;
}

.sc-main-scroll .sc-time {
  color: #ffffff;
  padding: 4px 0;
  line-height: 18px;
  height: 18px;
  display: block;
  float: left;
  border-right: solid 1px #ccc;
  text-align: center;
}

.sc-main-box {
  float: left;
  overflow-x: auto;
  overflow-y: auto;
}

.sc-main {
  position: relative;
}

.timeline {
  position: relative;
}

.sc-bar {
  position: absolute;
  color: #fff;
  background: #ff4800;
  cursor: pointer;
  z-index: 10;
  box-shadow: 2px 2px 4px #333;
  -moz-box-shadow: 2px 2px 4px #333;
  -webkit-box-shadow: 2px 2px 4px #333;
}

.ui-draggable-dragging,
.ui-resizeable {
  z-index: 20;
}

.timeline,
.sc-main .tb {
  border-bottom: solid 2px #666;
}

.sc-rows .timeline {
  overflow: hidden;
}

.sc-rows .timeline span {
  padding: 10px;
  display: block;
}

.sc-rows .timeline span.photo {
  float: left;
  width: 36px;
  height: 36px;
  padding: 10px 0 10px 10px;
}

.sc-rows .timeline span.title {
  float: left;
  padding: 10px 0 10px 10px;
  width: 92px;
}

.sc-main-scroll .sc-main .tl {
  float: left;
  height: 100%;
  border-right: solid 1px #ccc;
}

.sc-main-scroll .sc-main .tl:hover {
  background: #f0f0f0;
}

.sc-time {
  font-size: 10px;
}

.cant-res {
  background-color: #999 !important;
}

.isMe {
  background-color: #108000 !important;
}

.notMe {
  background-color: #ec920a !important;
}

/deep/ .sc-bar .head {
  display: block;
  padding: 6px 8px 0;
  font-size: 12px;
  height: 16px;
  overflow: hidden;
}

/deep/ .sc-bar .text {
  display: block;
  padding: 5px 15px 0;
  font-weight: bold;
  height: 18px;
  overflow: hidden;
}

/deep/ .resizable-e {
  cursor: e-resize;
  width: 10px;
  right: -5px;
  top: 0;
  height: 100%;
  z-index: 90;
  position: absolute;
  font-size: 0.1px;
  display: block;
}
</style>
