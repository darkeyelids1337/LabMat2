<template>
  <div class="settings-container">
    <div class="settings">
      <button @click="isFile = !isFile">Файл</button>
      <button v-show="isFile" @click="isFileModal = !isFileModal">
        Новая система
      </button>
    </div>
    <div class="settings">
      <button @click="isParam = !isParam">Параметры</button>
      <button v-show="isParam" @click="isGeneralModal = !isGeneralModal">
        Общие параметры
      </button>
      <button v-show="isParam" @click="isCoeffModal = !isCoeffModal">
        Параметры видов
      </button>
    </div>
    <div class="settings">
      <button @click="() => {isStart = !isStart; toStart()}">Запуск модели</button>
    </div>
  </div>
  <div style="margin-top: 5%">
    <div>Общее количество особей: {{ totalPopulation }}</div>
    <div style="margin-top: 2%">Текущее время: {{ data[data.length - 1].day }}</div>
  </div>
  <div class="file-modal" v-if="isFileModal">
    <div class="close-modal" @click="isFileModal = !isFileModal">X</div>
    <label>
      Число видов:
      <input type="number" v-model="populationsCount" />
    </label>
    <button
      @click="
        () => {
          isFileModal = !isFileModal;
          isParamModal = !isParamModal;
        }
      "
    >
      OK
    </button>
  </div>
  <div class="file-modal" v-if="isGeneralModal">
    <div class="close-modal" @click="isGeneralModal = !isGeneralModal">X</div>
    <div>
      <label>
        Шаг дифференцирования:
        <input type="number" v-model="step" />
      </label>
    </div>
    <div>
      <label>
        Время моделирования:
        <input type="number" v-model="duration" />
      </label>
    </div>
    <div>
      <button
        @click="
          () => {
            isParam = !isParam;
            isGeneralModal = !isGeneralModal;
          }
        "
      >
        OK
      </button>
    </div>
  </div>
  <div class="file-modal" style="width: 40%" v-if="isCoeffModal">
    <div class="close-modal" @click="isCoeffModal = !isCoeffModal">X</div>
    <div style="display: flex; gap: 20px">
      <div
        class="populations-table"
        style="display: flex; flex-direction: column"
      >
        Численность популяций i-го вида и прирост
        <div style="margin-top: 5%">
          <div style="display: flex; justify-content: space-around">
            <div style="width: 15%">
              {{ populations.length }}
            </div>
            <div style="width: 15%">Количество</div>
            <div style="width: 15%">Прирост</div>
          </div>
          <div
            v-for="(item, index) in populations"
            :key="index"
            class="populations-item"
          >
            <div>
              {{ index }}
            </div>
            <div>
              <input
                :value="item.amount"
                @change="
                  changePopulations($event.target.value, index, 'amount')
                "
              />
            </div>
            <div>
              <input
                :value="item.growth"
                @change="
                  changePopulations($event.target.value, index, 'growth')
                "
              />
            </div>
          </div>
        </div>
      </div>
      <div class="coeff-table-container">
        <div>Коэффициенты взаимодействия популяций</div>

        <div class="coeff-table">
          <div style="display: flex; justify-content: space-evenly">
            <div class="table-item"></div>
            <div
              v-for="(item, index) in coeffs"
              :key="index"
              class="table-item"
            >
              {{ index }}
            </div>
          </div>
          <div
            v-for="(item, index) in coeffs"
            :key="index"
            style="display: flex; justify-content: space-evenly"
          >
            <div class="table-item">{{ index }}</div>
            <div v-for="(el, col) in item" :key="col" class="table-item">
              <input
                :value="el"
                style="width: 90%"
                @change="changeCoeff($event.target, index, col)"
              />
            </div>
          </div>
        </div>
      </div>
    </div>
    <div>
      <button
        @click="
          () => {
            isParam = !isParam;
            isCoeffModal = !isCoeffModal;
            console.log(step, duration);
          }
        "
        style="margin-top: 2%"
      >
        OK
      </button>
    </div>
  </div>
  <Chart
    :size="{ width: 700, height: 400 }"
    :data="data"
    :margin="{ top: 15, right: 30, left: 20, bottom: 5 }"
    :direction="direction"
    :axis ='axis'
  >
    <template #layers>
      <Grid strokeDasharray="3,3" />
      <Line
        v-for="(item, index) in populations"
        :key="item"
        :lineStyle="{ stroke: colors[index % colors.length] }"
        :dotStyle="{strokeWidth: 1}"
        :dataKeys="['day', index]"
        type="monotone"
      />
    </template>
  </Chart>
</template>

<script>
import { Chart, Grid, Line } from "vue3-charts";

export default {
  name: "App",
  components: { Chart, Grid, Line },
  data() {
    return {
      isFile: false,
      isFileModal: false,
      isGeneralModal: false,
      isParam: false,
      isParamModal: false,
      isCoeffModal: false,
      isStart: false,
      populationsCount: 2,
      step: 1,
      duration: 1000,
      coeffs: new Array(2)
        .fill(new Array(2).fill(0.0001))
        .map((row, rowIdx) =>
          row.map((num, colIdx) => (rowIdx === colIdx ? 0 : num))
        ),
      populations: new Array(2)
        .fill(0)
        .map((item, index) => ({
          amount: 100 * (index + 1),
          growth: index === 0 ? 0.01 : -0.01,
        })),
      data: new Array(1).fill(0).map(() => {
        const len = this.populationsCount ? this.populationsCount : 2;
        const obj = {
          day: 0,
        };
        for (let i = 0; i < len; ++i) {
          obj[i] = this.populations ? this.populations[i].amount : 100 * (i+1);
        }
        return obj;
      }),
      direction: "horizontal",
      axis: {
        primary: {
        domain: ['dataMin', 'dataMax + 100'],
        type: 'linear',
        ticks: 16
      },
      secondary: {
       type:'linear'
      }
      },
      colors: ['red', 'green', 'blue', 'black', 'yellow', 'grey', 'orange']
    };
  },
  watch: {
    populationsCount(newValue) {
      this.coeffs = new Array(newValue)
        .fill(new Array(newValue).fill(0.0001))
        .map((row, rowIdx) =>
          row.map((num, colIdx) => (rowIdx === colIdx ? 0 : num))
        );
      this.populations = new Array(newValue)
        .fill(0)
        .map((item, index) => ({
          amount: 100 * (index + 1),
          growth: index === 0 ? 0.01 : -0.01,
        }));
      this.data = new Array(1).fill(0).map(() => {
        const obj = {
          day: 0,
        };
        for (let i = 0; i < newValue; ++i) {
          obj[i] = this.populations[i].amount;
        }
        return obj;
      });
    },
  },
  methods: {
    changeCoeff(e, item, col) {
      this.coeffs[item][col] = +e.value;
    },
    changePopulations(value, index, item) {
      this.populations[index][item] = +value;
      this.data = new Array(1).fill(0).map(() => {
        const obj = {
          day: 0,
        };
        for (let i = 0; i < this.populationsCount; ++i) {
          obj[i] = this.populations[i].amount;
        }
        return obj;
      });
    },
    toStart() {
      let requestId;
      let day = 0;
      const startSimulation = () => {
        if (!this.isStart) {
          return;
        }
        if (day > this.duration) {
          cancelAnimationFrame(requestId);
          return;
        }
        const currentData = this.data[this.data.length - 1];
        const newData = {
          ...currentData,
          day: currentData.day + this.step,
        };
        // console.log(this.populations)
        for (let k = 0; k < this.step; ++k) {
          for (let i = 0; i < this.populationsCount; ++i) {
            console.log(newData)
            const N = newData[i]; // amount current population
            let dN = N * this.populations[i].growth;
            console.log(dN, N)
            for (let j = 0; j < this.populationsCount; ++j) {
              dN += this.coeffs[i][j] * N * newData[j];
            }
            newData[i] = N + dN;
          }
        }
        this.data = [...this.data, newData];
        day += this.step;
        requestId = requestAnimationFrame(startSimulation);
      };
      this.isStart && startSimulation();
    },
  },
  computed: {
    chartData() {
      const obj = {
        labels: ["January", "February", "March"],
        datasets: [{ data: [40, 20, 12] }, { data: [21, 22, 23] }],
      };
      return obj;
    },
    totalPopulation(){
      let total = 0;
      for(let i =0; i < this.populationsCount; ++i){
        total += this.data[this.data.length - 1][i];
      }
      return total;
    }
  },
};
</script>

<style>
body,
html {
  width: 100%;
  height: 100%;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin: 0;
  padding: 0;
}
.settings-container {
  display: flex;
}
.settings {
  display: flex;
  flex-direction: column;
}
.file-modal {
  position: fixed;
  left: 25%;
  z-index: 3;
  width: 15%;
  padding: 20px;
  background-color: rgba(78, 75, 75, 0.527);
}
.close-modal {
  color: red;
  position: absolute;
  top: 1%;
  right: 1%;
  cursor: pointer;
}
.populations-table {
  background-color: white;
  padding: 20px;
}
.populations-item {
  display: flex;
  justify-content: space-around;
}
.populations-item div {
  margin-top: 1%;
  border: 1px solid #000000;
  width: 15%;
}
.populations-item input {
  padding: 0;
  width: 100%;
}
.coeff-table-container {
  background-color: white;
  padding: 20px;
}
.coeff-table .table-item {
  width: 15%;
  border: 1px solid #000000;
  margin-top: 5%;
}
iframe#webpack-dev-server-client-overlay{display:none!important}
</style>
