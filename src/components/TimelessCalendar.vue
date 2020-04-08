<template>
  <div
    class="timeless-calendar"
    :style="{
      'grid-template-rows': gridTemplateRowsCss,
      'grid-template-columns': gridTemplateColumnsCss
    }"
  >
    <div
      v-for="(hour, i) in hours"
      :key="hour"
      class="item"
      :style="{
        'grid-row': `${(i * 4) + 2}/${(i * 4) + 4 + 2}`,
        'grid-column': '1',
      }"
    >{{hour}}:00</div>

    <div
      v-for="(date, i) in dates"
      :key="date"
      class="item"
      :style="{
        'grid-row': '1',
        'grid-column': `${i + 2}`,
      }"
    >{{date}}</div>

    <div
      v-for="timebean in timetable"
      :key="`${timebean.date}${timebean.startHour}${timebean.startMin}`"
      class="item"
      :style="{
      'grid-row': calcRow(timebean),
      'grid-column': calcColumn(timebean),
    }"
    >
      <div>{{timebean.body}}</div>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import { uniq } from 'lodash-es';

interface Plan {
  date: string;
  startHour: number;
  startMin: number;
  unit: number;
  body: string;
}

export default Vue.extend({
  name: 'TimelessCalendar',
  props: {
    timetable: Array as Vue.PropType<Plan[]>,
  },
  data() {
    return {
      hours: [
        5,
        6,
        7,
        8,
        9,
        10,
        11,
        12,
        13,
        14,
        15,
        16,
        17,
        18,
        19,
        20,
        21,
        22,
        23,
        24,
        25,
        26,
        27,
        28,
      ],
    };
  },
  computed: {
    gridTemplateRowsCss() {
      // eslint-disable-next-line @typescript-eslint/no-unused-vars
      return `50px ${[...Array(1000)].map((t) => '15px').join(' ')}`;
    },
    gridTemplateColumnsCss() {
      return `200px ${Object.keys(this.timetable)
        // eslint-disable-next-line @typescript-eslint/no-unused-vars
        .map((t) => '300px')
        .join(' ')}`;
    },
    dates() {
      const { timetable } = this;
      return uniq(timetable.map((tb) => tb.date)).sort();
    },
  },
  methods: {
    calcRow(plan: Plan): string {
      const { startHour, startMin, unit } = plan;
      const start = ((startHour - 5) * 60 + startMin) / 15 + 2;
      return `${start}/${start + unit}`;
    },
    calcColumn(plan: Plan): string {
      const i = this.dates.findIndex((date) => date === plan.date);
      return `${i + 2}`;
    },
  },
});
</script>

<style scoped>
.timeless-calendar {
  display: grid;
  background: #1b1b77;
  overflow-x: auto;
  overflow-y: scroll;
  white-space: nowrap;
  height: calc(100vh - 100px);
  -webkit-overflow-scrolling: touch;
}

.item {
  display: inline-block;
  margin: 1px;
  background: #f5f0f0;
}
</style>
