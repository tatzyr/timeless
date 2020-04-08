<template>
  <div id="app">
    <div class="head">
      <h1>Timeless</h1>
    </div>
    <div class="buttons">
      <button @click="reset">Reset</button>
      <button @click="download">Download</button>
    </div>
    <div class="editor">
      <textarea class="editor-textarea" v-model="text"></textarea>
    </div>
    <TimelessCalendar class="calendar" :timetable="timetable" />
  </div>
</template>

<script lang="ts">
import 'modern-normalize';
import Vue from 'vue';
import { debounce } from 'lodash-es';
import TimelessCalendar from './components/TimelessCalendar.vue';

const defaultText = `# format
#
# YYYY-MM-DD
# hhmm-U: <text>
# hhmm-U: <text>
# hhmm-U: <text>
#
# (* 1U = 15 minutes)

2020-04-11
1000-4: eu nisl nunc mi ipsum
1100-2: in aliquam sem fringilla ut
1130-2: turpis egestas maecenas
1330-6: viverra accumsan in nisl

2020-04-12
1130-2: eu turpis
1230-6: amet consectetur adipiscing elit
1700-4: viverra nam libero
1800-2: pretium fusce id

2020-04-14
1400-4: integer malesuada nunc vel
1500-2: commodo viverra maecenas
1930-2: enim nec dui
2230-3: eu nisl nunc mi

2020-04-15
0800-4: neque viverra
1200-4: senectus et netus et
1430-2: id venenatis a condimentum
1730-6: sapien pellentesque

2020-04-16
1000-4: iaculis nunc sed
1100-2: sed odio morbi quis
1130-6: phasellus faucibus scelerisque
1330-6: facilisis sed
`;

interface Plan {
    date: string ;
    startHour: number;
    startMin: number;
    unit: number;
    body: string;
}

export default Vue.extend({
  name: 'App',
  components: {
    TimelessCalendar,
  },
  data() {
    return {
      text: '',
      // eslint-disable-next-line @typescript-eslint/no-empty-function
      debouncedPersist: (): void => {},
    };
  },
  created() {
    const timeless = localStorage.getItem('timeless');
    if (timeless) {
      try {
        this.text = JSON.parse(timeless).text;
      } catch (e) {
        localStorage.removeItem('timeless');
      }
    } else {
      this.text = defaultText;
    }
    this.debouncedPersist = debounce(this.persist, 500);
  },
  computed: {
    timetable(): Array<Plan> {
      const data = [];
      const lines = this.text.split('\n');
      let date;
      // eslint-disable-next-line no-restricted-syntax
      for (const line of lines) {
        const dateM = /^(\d{4}-?(\d{2})-?(\d{2}))$/.exec(line);
        if (dateM) {
          [, date] = dateM;
          // eslint-disable-next-line no-continue
          continue;
        }
        const yoteiM = /^(\d{2}):?(\d{2})-?(\d+):\s*(.+)$/.exec(line);
        if (yoteiM) {
          if (!date) {
            // eslint-disable-next-line no-continue
            continue;
          }
          data.push({
            date,
            startHour: Number(yoteiM[1]),
            startMin: Number(yoteiM[2]),
            unit: Number(yoteiM[3]),
            body: yoteiM[4],
          });
          // eslint-disable-next-line no-continue
          continue;
        }
      }
      return data;
    },
  },
  methods: {
    persist(): void {
      const dumped = JSON.stringify({ text: this.text });
      localStorage.setItem('timeless', dumped);
    },
    reset(): void {
      // eslint-disable-next-line no-alert
      if (window.confirm('Are you sure?')) {
        this.text = defaultText;
      }
    },
    download(): void {
      const blob = new Blob([this.text], { type: 'text/plain;charset=utf-8;' });
      const link = document.createElement('a');
      const url = URL.createObjectURL(blob);
      link.setAttribute('href', url);
      link.setAttribute('download', `timeless_${Date.now()}.txt`);
      link.style.visibility = 'hidden';
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },
  },
  watch: {
    text() {
      this.debouncedPersist();
    },
  },
});
</script>

<style>
#app {
  display: grid;
  grid-template-rows: 100px 50px 1fr;
  grid-template-columns: 1fr 2fr;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}

.head {
  grid-row: 1;
  grid-column: 1/3;
  text-align: center;
}

.downloader {
  display: none;
}

.editor {
  grid-row: 3;
  grid-column: 1;
  background: #ffcccc;
}

.editor-textarea {
  height: 100%;
  width: 100%;
  font-family: 'Hack';
}

.calendar {
  grid-row: 2/4;
  grid-column: 2;
}
</style>
