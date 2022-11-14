<template>
  <div id="app">
    <div>
      <input type="button" value="Add new row" @click="addNewRow" />
    </div>
    <sc
      :schedule-data="scData"
      :setting="setting"
      @row-click-event="rowClickEvent"
      @date-click-event="dateClickEvent"
      @click-event="clickEvent"
      @add-event="addEvent"
      @move-event="moveEvent"
      @edit-event="editEvent"
      @delete-event="deleteEvent"
    ></sc>
    <div style="display: flex">
      <span>Guide:</span>
      <div class="sample"></div>
      <span>Normal</span>
      <div class="sample cant-res"></div>
      <span>Not BusinessDay</span>
      <div class="sample reserved"></div>
      <span>Reserved</span>
    </div>
  </div>
</template>

<script>
import selectorLite from "./components/selectorLite";

const sampleData = [
  {
    title: "Room１",
    noBusinessDate: ["2022/04/20"],
    businessHours: [
      {
        start: "00:00",
        end: "24:00",
      },
      {
        start: "00:00",
        end: "24:00",
      },
      {
        start: "00:00",
        end: "24:00",
      },
      {
        start: "00:00",
        end: "24:00",
      },
      {
        start: "00:00",
        end: "24:00",
      },
      {
        start: "00:00",
        end: "24:00",
      },
      {
        start: "00:00",
        end: "24:00",
      },
    ],
    schedule: [
      {
        text: "Mr.A reserved",
        start: "2022/04/21 06:00",
        end: "2022/04/22 01:00",
        data: {
          something: "something",
        },
      },
      {
        text: "Mr.B reserved",
        start: "2022/04/22 06:00",
        end: "2022/04/22 12:00",
        data: {
          something: "something",
        },
      },
    ],
  },
  {
    title: "Room２",
    noBusinessDate: [],
    businessHours: [
      {
        start: "10:00",
        end: "17:00",
      },
      {
        start: "10:00",
        end: "17:00",
      },
      {
        start: "10:00",
        end: "17:00",
      },
      {
        start: "10:00",
        end: "17:00",
      },
      {
        start: "10:00",
        end: "17:00",
      },
      {
        start: "10:00",
        end: "17:00",
      },
      {
        start: "10:00",
        end: "17:00",
      },
    ],
    schedule: [
      {
        text: "Mr.C reserved",
        start: "2022/04/20 12:00",
        end: "2022/04/20 17:00",
        data: {
          something: "something",
        },
      },
    ],
  },
  {
    title: "Room３",
    noBusinessDate: [],
    businessHours: [
      {
        start: "00:00",
        end: "24:00",
      },
      {
        start: "00:00",
        end: "24:00",
      },
      {
        start: "00:00",
        end: "24:00",
      },
      {
        start: "00:00",
        end: "24:00",
      },
      {
        start: "00:00",
        end: "24:00",
      },
      {
        start: "00:00",
        end: "24:00",
      },
      {
        start: "00:00",
        end: "24:00",
      },
    ],
    schedule: [
      {
        text: "Mr.D reserved",
        start: "2022/04/20 12:00",
        end: "2022/04/20 18:00",
        data: {
          something: "something",
        },
      },
    ],
  },
];

const sampleSetting = {
  startDate: "2022/04/20",
  endDate: "2022/04/26",
  weekdayText: ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"],
  unit: 60, // Minutes
  borderW: 1, // Px
  dateDivH: 25, // Px
  timeDivH: 25, // Px
  unitDivW: 25, // Px
  titleDivW: 20, // Percent
  rowH: 135, // Px
};

export default {
  name: "App",
  components: {
    sc: selectorLite,
  },
  data: function () {
    return {
      scData: sampleData,
      setting: sampleSetting,
    };
  },
  methods: {
    dateClickEvent(date) {
      console.log("------");
      console.log("DateClickEvent:");
      console.log("Date:" + date);
    },
    rowClickEvent(rowIndex, text) {
      console.log("------");
      console.log("RowClickEvent:");
      console.log("RowIndex:" + rowIndex);
      console.log("RowTitle:" + text);
    },
    clickEvent(startDate, endDate, text, other) {
      console.log("------");
      console.log("ClickEvent:");
      console.log("StartDate:" + startDate);
      console.log("EndDate:" + endDate);
      console.log("ContentText:" + text);
      if (other) {
        console.log("OtherData:");
        console.log(other);
      }
    },
    addEvent(rowIndex, startDate, endDate) {
      console.log("------");
      console.log("AddEvent:");
      console.log("RowIndex:" + rowIndex);
      console.log("StartDate:" + startDate);
      console.log("EndDate:" + endDate);
    },
    moveEvent(status, newStartDate, newEndDate) {
      console.log("------");
      console.log("MoveEvent:");
      if (status == 1) {
        console.log("NewStartDate:" + newStartDate);
        console.log("NewEndDate:" + newEndDate);
      } else if (status == 2) {
        console.log("Has other event, can't move.");
      } else {
        console.log("Not businessDay, can't move.");
      }
    },
    editEvent(newStartDate, newEndDate) {
      console.log("------");
      console.log("EditEvent:");
      console.log("NewStartDate:" + newStartDate);
      console.log("NewEndDate:" + newEndDate);
    },
    deleteEvent(row, index) {
      console.log("------");
      console.log("DeleteEvent:");
      console.log("Row:" + row);
      console.log("Index:" + index);
    },
    addNewRow() {
      let newTitle = "Room" + (this.scData.length + 1);
      this.scData.push({
        title: newTitle,
        noBusinessDate: [],
        businessHours: [
          {
            start: "00:00",
            end: "24:00",
          },
          {
            start: "00:00",
            end: "24:00",
          },
          {
            start: "00:00",
            end: "24:00",
          },
          {
            start: "00:00",
            end: "24:00",
          },
          {
            start: "00:00",
            end: "24:00",
          },
          {
            start: "00:00",
            end: "24:00",
          },
          {
            start: "00:00",
            end: "24:00",
          },
        ],
        schedule: [],
      });
    },
  },
};
</script>

<style>
.sample {
  width: 10px;
  height: 10px;
  margin: 5px;
  border: 1px solid black;
}

.cant-res {
  background-color: #999 !important;
}

.reserved {
  background-color: #ec920a !important;
}
</style>
