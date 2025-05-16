<script>
  export default {
    data () {
      return {
        colRowsOpt: [
          { value: 2, text: '2' },
          { value: 3, text: '3' },
          { value: 4, text: '4' },
          { value: 5, text: '5' },
          { value: 6, text: '6' },
          { value: 7, text: '7' },
          { value: 8, text: '8' },
          { value: 9, text: '9' },
          { value: 10, text: '10' },
        ],
        colsOpt: null,
        rowsOpt: null,
        summs: [],
        checkSum: null,
        matrValues: null,
        stocksValues: null,
        needsValues: null,
        showTable: false,
        showAnswer: false,
        northWestValues: null,
        northWestAnswer: null,
        potentials: null,
        deltas: null,
        maxOfDeltas: null,
        endResult: [],
        countRoutes: 0,
      }
    },
    methods: {
      createTables() {
        if (this.colsOpt != null && this.rowsOpt != null) {
          this.matrValues = []
          for (let i = 0; i < this.rowsOpt; i++) {
            this.matrValues.push([])
            for (let j = 0; j < this.colsOpt; j++) {
              this.matrValues[i].push(0)
            }
          }
          this.stocksValues = []
          for (let i = 0; i < this.rowsOpt; i++) {
            this.stocksValues.push(0)
          }
          this.needsValues = []
          for (let i = 0; i < this.colsOpt; i++) {
            this.needsValues.push(0)
          }
          this.showTable = true
          this.showAnswer = false
        }
      },
      printTables() {
        for(let i = 0; i < this.rowsOpt; i++) {
          for (let j = 0; j < this.colsOpt; j++) {
            this.matrValues[i][j] = parseInt(this.matrValues[i][j])
          }
        }
        for (let i = 0; i < this.stocksValues.length; i++) {
          this.stocksValues[i] = parseInt(this.stocksValues[i])
        }
        for (let i = 0; i < this.needsValues.length; i++) {
          this.needsValues[i] = parseInt(this.needsValues[i])
        }
        this.showAnswer = true
        this.showTable = false
        this.summarize(1, 0);
        this.summarize(0, 1);
        this.checkSumms()
        this.northWest()
        this.buildAnswer()
        this.potentials = this.calculatePotentials(this.northWestValues, this.matrValues)
        this.deltas = this.calculateDelta(this.northWestValues, this.matrValues, this.potentials)
        this.maxOfDeltas = this.checkDeltas(this.deltas)
        this.createRoute(this.northWestValues, this.maxOfDeltas)
      },
      summarize(a, b) {
        if (a) {
          let s = 0
          for (let i = 0; i < this.stocksValues.length; i++) {
            s += this.stocksValues[i]
          }
          return this.summs.push(s);
        }
        else {
          let s = 0
          for (let i = 0; i < this.needsValues.length; i++) {
            s += this.needsValues[i]
          }
          return this.summs.push(s);
        }
      },
      northWest() {
        let i = 0
        let j = 0
        let rows = [...this.stocksValues];
        let cols = [...this.needsValues];
        let res = [];
        for (let i = 0; i < rows.length; i++) {
          res.push([]);
          for (let j = 0; j < cols.length; j++) {
            res[i].push(null);
          }
        }
        while (i < rows.length && j < cols.length) {
          res[i][j] = Math.min(rows[i], cols[j]);
          rows[i] -= res[i][j];
          cols[j] -= res[i][j];
          if (rows[i] == 0) {
            i++;
          }
          if (cols[j] == 0) {
            j++;
          }
        }
        this.northWestValues = res;
      },
      calculateCount() {
        let count = 0;
        for (let i = 0; i < this.northWestValues.length; i++) {
          for (let j = 0; j < this.northWestValues[i].length; j++) {
            if (this.northWestValues[i][j] != null) {
              count++;
            }
          }
        }
        return count;
      },
      calculateAnswer(values, matr) {
        let answer = 0;
        for (let i = 0; i < values.length; i++) {
          for (let j = 0; j < values[i].length; j++) {
            answer += values[i][j] * matr[i][j];
          }
        }
        return answer;
      },
      showCalculate(values, matr) {
        let s = ``;
        for (let i = 0; i < values.length; i++) {
          for (let j = 0; j < values[i].length; j++) {
            if (values[i][j] != null) {
              s += `${values[i][j]} * ${matr[i][j]} + `;
            }
          }
        }
        return s.slice(0, -3);
      },
      calculatePotentials(northwest, matr) {
        let potentials = null
        let values = [...matr];
        let u = [];
        let v = [];
        for (let i = 0; i < this.rowsOpt; i++) {
          u.push(null);
        }
        for (let i = 0; i < this.colsOpt; i++) {
          v.push(null);
        }
        u[0] = 0;
        let repeat = []
        for (let i = 0; i < this.rowsOpt; i++) {
          for (let j = 0; j < this.colsOpt; j++) {
            if (northwest[i][j] != null) {
              if (u[i] != null && v[j] == null) {
                v[j] = values[i][j] - u[i];
              } else if (u[i] == null && v[j] != null) {
                u[i] = values[i][j] - v[j];
              } else if (u[i] == null && v[j] == null) {
                repeat.push([i, j])
              }
            }
          }        
        }
        while (repeat.length != 0) {
          for (let k = 0; k < repeat.length; k++) {
            let i = repeat[k][0];
            let j = repeat[k][1];
            if (u[i] != null && v[j] == null) {
              v[j] = values[i][j] - u[i];
              repeat.splice(k, 1);
            } else if (u[i] == null && v[j] != null) {
              u[i] = values[i][j] - v[j];
              repeat.splice(k, 1);
            }
          }
        }
        potentials = [u, v];
        return potentials
      },
      calculateDelta(northwest, matr, potentials) {
        let delta = [];
        let values = [...matr];
        for (let i = 0; i < this.rowsOpt; i++) {
          delta.push([]);
        }
        for (let i = 0; i < this.rowsOpt; i++) {
          for (let j = 0; j < this.colsOpt; j++) {
            delta[i].push(null);
          }
        }
        for (let i = 0; i < this.rowsOpt; i++) {
          for (let j = 0; j < this.colsOpt; j++) {
            if (northwest[i][j] == null) {
              delta[i][j] = potentials[0][i] + potentials[1][j] - values[i][j];
            }
          }
        }
        return delta
      },
      checkDeltas(deltas) {
        let delta = [...deltas];
        let max = null
        let flag = true;
        for (let i = 0; i < delta.length; i++) {
          for (let j = 0; j < delta[i].length; j++) {
            if (delta[i][j] > 0) flag = false
          }
        }
        if (flag) return max = [0, 0, 0];
        else {
          let maxim = 0;
          let maxi = 0
          let maxj = 0
          for (let i = 0; i < delta.length; i++) {
            for (let j = 0; j < delta[i].length; j++) {
              if (delta[i][j] > maxim) {
                maxim = delta[i][j]
                maxi = i
                maxj = j
              }
            }
          }
          max = [maxim, maxi, maxj]
          return max
        }
      },
      checkSumms() {
        let Ai = this.summs[0];
        let Bj = this.summs[1];
        if (Ai == Bj) return this.checkSum = 1;
        else if (Ai > Bj) { 
          this.needsValues.push(Ai - Bj)
          for (let i = 0; i < this.matrValues.length; i++) {
            this.matrValues[i].push(0);
          }
          this.colsOpt++
          return this.checkSum = 2;
        }
        else if (Ai < Bj) {
          this.stocksValues.push(Bj - Ai)
          this.matrValues.push([]);
          for (let j = 0; j < this.colsOpt; j++) {
            this.matrValues[this.matrValues.length - 1].push(0);
          }
          this.rowsOpt++
          return this.checkSum = 3;
        }
      },
      buildAnswer() {
        let need = this.rowsOpt + this.colsOpt - 1
        let have = this.calculateCount()
        if (have != need) {
          let Idxi = 0
          let Idxj = 0
          let lose_value = this.northWestValues[0][0]
          for (let i = 1; i < this.northWestValues.length; i++) {
            for (let j = 1; j < this.northWestValues[i].length; j++) {
              if (this.northWestValues[i][j - 1] == null && this.northWestValues[i - 1][j] == null) {
                lose_value = this.matrValues[i][j]
                Idxi = i
                Idxj = j
              }
            }
          }
          let minElem = this.matrValues[Idxi][0];
          let minIdxI = Idxi;
          let minIdxJ = 0;

          // два обхода
          for (let j = 0; j < Idxj; j++) {
            if (this.matrValues[minIdxI][j] < minElem) {
              minElem = this.matrValues[minIdxI][j];
              minIdxI = Idxi;
              minIdxJ = j;
            }
          }
          for (let i = 0; i < Idxi; i++) {
            if (this.matrValues[i][Idxj] < minElem) {
              minElem = this.matrValues[i][Idxj];
              minIdxI = i;
              minIdxJ = Idxj;
            }
          }

          this.northWestValues[minIdxI][minIdxJ] = 0
        }
      },
      createRoute(northwest, max_delta) {
        let matr = JSON.parse(JSON.stringify(northwest))
        matr[max_delta[1]][max_delta[2]] = 0
        // Удаляем строки с 1 элементом
        let c1 = true
        let c2 = true
        while (c1 || c2) {
          c1 = false
          for (let i = 0; i < matr.length; i++) {
            let cnt = 0
            for (let j = 0; j < matr[i].length; j++) {
              if (matr[i][j] != null) cnt++
            }
            if (cnt == 1) {
              for (let j = 0; j < matr[i].length; j++) {
                matr[i][j] = null
                c1 = true
              }
            }
          }
          // Удаляем столбцы с 1 элементом
          c2 = false
          for (let j = 0; j < this.colsOpt; j++) {
            let cnt = 0
            for (let i = 0; i < this.rowsOpt; i++) {
              if (matr[i][j] != null) cnt++
            }
            if (cnt == 1) {
              for (let i = 0; i < this.rowsOpt; i++) {
                matr[i][j] = null
                c2 = true
              }
            }
          }
        }
        if (this.endResult.length == 0) {
          this.endResult.push([0, this.northWestValues, this.maxOfDeltas])
        }
        this.endResult.push([matr])
        console.log(this.northWestValues, this.deltas)
        console.log(matr, max_delta)
        this.newRoute(this.endResult[this.endResult.length - 1][0], max_delta)
      },
      newRoute(matr, max_delta) {
        let start_i = max_delta[1]
        let start_j = max_delta[2]
        let symbol = '-'
        let directions = ['right', 'down', 'left', 'up']
        let dirIdx = 0
        let i = null
        let j = null
        matr[start_i][start_j] = '+'
        let isFirstStep = true
        let cycleClosed = false
        do {
          if (isFirstStep) {
            i = start_i
            j = start_j
          }

          console.log(directions[dirIdx])
          let result = this.goto(matr, i, j, directions[dirIdx])
          if (result != null) {
            [i, j] = result
            matr[i][j] = symbol
            if (symbol == '+') symbol = '-'
            else symbol = '+'
            dirIdx = (dirIdx + 1) % directions.length
            isFirstStep = false
          }
          else {
            dirIdx = (dirIdx + 2) % directions.length
          }

          if (i == start_i && j == start_j && !isFirstStep) {
            cycleClosed = true
          }
        }
        while (!cycleClosed)
        console.log(matr)
        return this.newPlan(matr, this.endResult[this.endResult.length - 2][1])
      },
      goto(matr, i, j, side) {
        if (side == 'right') {
          for (let nj = j + 1; nj < matr[i].length; nj++) {
            if (matr[i][nj] != null) return [i, nj]
          }
        }
        if (side == 'left') {
          for (let nj = j - 1; nj >= 0; nj--) {
            if (matr[i][nj] != null) return [i, nj]
          }
        }
        if (side == 'down') {
          for (let ni = i + 1; ni < matr.length; ni++) {
            if (matr[ni][j] != null) return [ni, j]
          }
        }
        if (side == 'up') {
          for (let ni = i - 1; ni >= 0; ni--) {
            if (matr[ni][j] != null) return [ni, j]
          }
        }
        return null
      },
      newPlan(matr, values) {
        let newNorthWest = JSON.parse(JSON.stringify(values))
        // Находим минимальное среди отрицательных клеток
        let minim = 10 ** 10
        let min_i = null
        let min_j = null
        for (let i = 0; i < values.length; i++) {
          for (let j = 0; j < values[i].length; j++) {
            if (matr[i][j] == '-' && values[i][j] < minim) {
              minim = values[i][j]
              min_i = i
              min_j = j
            }
          }
        }
        // Убираем ячейку
        newNorthWest[min_i][min_j] = null
        // Заполняем новый опорный план
        for (let i = 0; i < values.length; i++) {
          for (let j = 0; j < values[i].length; j++) {
            if (matr[i][j] == '+') newNorthWest[i][j] += minim
            else if (matr[i][j] == '-' && i != min_i && j != min_j) newNorthWest[i][j] -= minim
          }
        }
        this.endResult[this.endResult.length - 1].push(newNorthWest)
        this.calcNewPlan(this.endResult[this.endResult.length - 1][1], this.matrValues)
      },
      calcNewPlan(northwest, values) {
        this.countRoutes++
        let newsum = this.calculateAnswer(northwest, values)
        let show = this.showCalculate(northwest, values)
        let potentials = this.calculatePotentials(northwest, values)
        let deltas = this.calculateDelta(northwest, values, potentials)
        let maxofDeltas = this.checkDeltas(deltas)
        let route = this.endResult[this.endResult.length - 1][0]
        console.log({
          newsum, show, potentials, deltas, maxofDeltas, northwest, route
        })
        this.endResult[this.endResult.length - 1].push(maxofDeltas, potentials, newsum, show, deltas)
        if (maxofDeltas[0] != 0) {
          this.createRoute(northwest, maxofDeltas)
        }
        else return console.log('Задача решена!')
      }
    },
    computed: {
      rowIndexes() {
        return Array.from({ length: this.rowsOpt }, (_, i) => i);
      },
      colIndexes() {
        return Array.from({ length: this.colsOpt }, (_, i) => i);
      }
    }
  }
</script>

<template>
  <BContainer class="mt-5 position-relative">
    <h1 class="text-center">Решение транспортной задачи<br>линейного программирования</h1>

    <BForm class="mt-5">
      <BFormGroup
        label-cols-sm="4"
        label-cols-lg="3"
        content-cols-sm
        content-cols-ls="7"
        id="input-group-1"
        label="Количество заказчиков"
        description="Введите количество столбцов таблицы"
        label-for="input-1"
        class="mb-2"
      >
          <BFormSelect
            id="input-1"
            :options="this.colRowsOpt"
            v-model="colsOpt"
            v-on:change="createTables()"
            :disabled="this.showTable || this.showAnswer"
            required />
      </BFormGroup>
      <BFormGroup
        label-cols-sm="4"
        label-cols-lg="3"
        content-cols-sm
        content-cols-ls="7"
        id="input-group-2"
        label="Количество поставщиков"
        description="Введите количество строк таблицы"
        label-for="input-2"
        class="mb-2"
      >
        <BFormSelect
          id="input-2"
          :options="this.colRowsOpt"
          v-model="rowsOpt"
          v-on:change="createTables()"
          :disabled="this.showTable || this.showAnswer"
          required />
      </BFormGroup>
    </BForm>
  </BContainer>
  <br>
  <BContainer v-if="showTable" class="mt-5">
    <h4 class="text-center mb-3">Таблица</h4>
    <table class="mx-auto">
      <thead>
        <tr>
          <td>
            <p class="text-center mb-3">Запасы, <span class="math-style">a<sub>i</sub></span></p>
          </td>
          <td :colspan="this.colsOpt">
            <p class="text-center mb-3">Потребности, <span class="math-style">b<sub>j</sub></span></p>
          </td>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td></td>
          <td v-for="col in colIndexes" :key="col">
            <BFormInput v-model="needsValues[col]" type="number" style="width: 100px; height: 40px; border-color: black; background-color: #e2e2e2;" />
          </td>
        </tr>
        <tr v-for="row in rowIndexes" :key="row">
          <td>
            <BFormInput v-model="stocksValues[row]" type="number" style="width: 100px; height: 40px; border-color: black; background-color: #e2e2e2;" />
          </td>
          <td v-for="col in colIndexes" :key="col">
            <BFormInput v-model="matrValues[row][col]" type="number" style="width: 100px; height: 40px; border-color: black;" />
          </td>
        </tr>
      </tbody>
      <tfoot>
        <tr>
          <td></td>
          <td :colspan="this.colsOpt">
            <p class="text-center mb-3">Матрица стоимостей одной единицы товара, <span class="math-style">c<sub>ij</sub></span></p>
          </td>
        </tr>
      </tfoot>
    </table>
    <BButton variant="primary" class="my-3 d-block mx-auto" v-on:click="printTables()">Вычислить</BButton>
    </BContainer>

    <BContainer v-if="showAnswer" class="mt-5">
      <h2 class="text-center mb-3">Решение</h2>
      <p class="text-center mb-3">Проверяем необходимое и достаточное условие разрешимости задачи:</p>
      <div class="d-flex gap-5 justify-content-center">
        <p class="text-center mb-3">Сумма запасов: <span class="math-style">&#8721a<sub>i</sub> = {{ this.summs[0] }}</span></p>
        <p class="text-center mb-3">Сумма потребностей: <span class="math-style">&#8721b<sub>j</sub> = {{ this.summs[1] }}</span></p>
      </div>
      <p class="text-center mb-3" v-if="this.checkSum == 1">Из найденных сумм выше следует, что <span class="math-style">&#8721a<sub>i</sub></span> = <span class="math-style">&#8721b<sub>j</sub></span>, следовательно, модель транспортной задачи является закрытой.</p>
      <p class="text-center mb-3" v-else-if="this.checkSum == 2">
        Из найденных сумм выше следует, что <span class="math-style">&#8721a<sub>i</sub></span> > <span class="math-style">&#8721b<sub>j</sub></span>, следовательно, модель транспортной задачи закрытой не является.
        <br>
        Добавим мнимого поставщика.
      </p>
      <p class="text-center mb-3" v-else-if="this.checkSum == 3">
        Из найденных сумм выше следует, что <span class="math-style">&#8721a<sub>i</sub></span> < <span class="math-style">&#8721b<sub>j</sub></span>, следовательно, модель транспортной задачи закрытой не является.
        <br>
        Добавим мнимого потребителя.
      </p>
      <p class="text-center mb-3">Получаем следующую задачу:</p>
      <div class="d-flex gap-5 justify-content-center align-items-center mt-4">
        <div class="">
          <math display="block" xmlns="http://www.w3.org/1998/Math/MathML">
            <mi>A</mi>
            <mo>=</mo>
            <mrow class="matrix">
              <mo fence="true" form="prefix" stretchy="true">(</mo>
              <mtable>
                <mtr v-for="row in rowIndexes" :key="row">
                  <mtd><mn>{{ this.stocksValues[row] }}</mn></mtd>
                </mtr>
              </mtable>
              <mo fence="true" form="postfix" stretchy="true">)</mo>
            </mrow>
          </math>
        </div>
        <div class="">
          <math display="block">
            <mi>B</mi>
            <mo>=</mo>
            <mrow class="matrix">
              <mo fence="true" form="prefix" stretchy="true">(</mo>
              <mtable>
                <mtr>
                  <mtd v-for="col in colIndexes" :key="col"><mn>{{ this.needsValues[col] }}</mn></mtd>
                </mtr>
              </mtable>
              <mo fence="true" form="postfix" stretchy="true">)</mo>
            </mrow>
          </math>
        </div>
        <div class="">
          <math display="block">
            <mi>C</mi>
            <mo>=</mo>
            <mrow class="matrix">
              <mo fence="true" form="prefix" stretchy="true">(</mo>
              <mtable>
                <mtr v-for="row in rowIndexes" :key="row">
                  <mtd v-for="col in colIndexes" :key="col"><mn>{{ this.matrValues[row][col] }}</mn></mtd>
                </mtr>
              </mtable>
              <mo fence="true" form="postfix" stretchy="true">)</mo>
            </mrow>
          </math>
        </div>
      </div>
      <div class="mt-5">
        <h4 class="text-center">Этап I. Поиск первого опорного плана.</h4>
        <p class="text-center mb-3">Используем метод северо-западного угла для получения первоначального опорного плана:</p>
        <p class="text-center mb-3">В результате получаем следующую таблицу:</p>
        <table class="mx-auto table-special my-3">
          <tr class="table-border">
            <td class="table-border"></td>
            <td v-for="col in colIndexes" class="table-border">
              {{ this.needsValues[col] }}
            </td>
          </tr>
          <tr class="table-border" v-for="row in rowIndexes" :key="row">
            <td class="table-border">
              {{ this.stocksValues[row] }}
            </td>
            <td class="table-border" v-for="col in colIndexes" :key="col">
              <span v-if="this.northWestValues[row][col] != null" style="
                font-size: 1.4rem;
                color: red;
                font-weight: 800;
              ">{{ this.northWestValues[row][col] }}</span>
              <sub style="
                font-size: 1em;
                font-style: normal;
                margin-left: 40px;
              ">{{ this.matrValues[row][col] }}</sub>
            </td>
          </tr>
        </table>
        <p class="text-center">В результате получен первый опорный план, который является допустимым, так как все грузы от поставщиков вывезены, потребность магазинов удовлетворена, а план соответствует системе ограничений транспортной задачи.</p>
        <p class="text-center">Подсчитаем число занятых клеток таблицы, их <span class="math-style">{{ this.calculateCount() }}</span>, а должно быть <span class="math-style">m + n - 1 = {{ this.rowsOpt + this.colsOpt - 1 }}</span>. Следовательно, опорный план является невырожденным.</p>
        <p class="text-center">Значение целевой функции для данного опорного плана равно:</p>
        <p class="text-center math-style mb-3">F(x) = {{ this.showCalculate(this.northWestValues, this.matrValues) }} = {{ this.calculateAnswer(this.northWestValues, this.matrValues) }}</p>
        <h4 class="text-center mt-5 mb-3">Этап II. Оптимизация опорного плана.</h4>
        <p class="text-center mb-3">Для оптимизации опорного плана используем метод потенциалов.</p>
        <p class="text-center mb-3">Найдем предварительные потенциалы <span class="math-style">u<sub>i</sub>, v<sub>j</sub></span>. по занятым клеткам таблицы, в которых <span class="math-style">u<sub>i</sub> + v<sub>j</sub> = c<sub>ij</sub></span>, полагая, что <span class="math-style">u<sub>1</sub> = 0</span>.</p>
        <p class="text-center mb-3">Выписываем оценки свободных ячеек, для которых должно выполняться условие:</p>
        <p class="text-center mb-3 math-style">u<sub>i</sub> + v<sub>j</sub> - c<sub>ij</sub> ⩽ 0</p>
        <div class="d-flex gap-3">
          <table class="table-special mt-3 mb-5 mx-auto">
          <tr class="table-border">
            <td class="table-border"></td>
            <td v-for="col in colIndexes" :key="col" class="table-border">
              <p class="text-center">{{ this.needsValues[col] }}</p>
            </td>
            <td class="table-border"><span class="math-style">u<sub>i</sub></span></td>
          </tr>
          <tr v-for="row in rowIndexes" :key="row" class="table-border">
            <td class="table-border">
              <p class="text-center">{{ this.stocksValues[row] }}</p>
            </td>
            <td v-for="col in colIndexes" :key="col" class="table-border" :style="{
              backgroundColor: row == this.maxOfDeltas[1] && col == this.maxOfDeltas[2] ? 'yellow' : 'white',
            }">
              <span v-if="this.northWestValues[row][col] != null" style="
                font-size: 1.4rem;
                color: red;
                font-weight: 800;
              ">{{ this.northWestValues[row][col] }}</span>
              <sub style="
                font-size: 1em;
                font-style: normal;
                margin-left: 40px;
              ">{{ this.matrValues[row][col] }}</sub>
            </td>
            <td class="table-border position-relative">
              <p class="position-absolute math-style" style="z-index: 100; color: #006d77; right: 0; left: 0; font-size: 2.4rem; text-align: center;">{{ this.potentials[0][row] }}</p>
              <p class="math-style" style="color: #d2d2d2; z-index: 0; padding: 0 10px 0 10px;">u<sub>{{ row + 1 }}</sub></p>
            </td>
          </tr>
          <tr class="table-border">
            <td class="table-border"><span class="math-style">v<sub>j</sub></span></td>
            <td v-for="col in colIndexes" :key="col" class="table-border position-relative">
              <p class="position-absolute math-style" style="z-index: 100; color: #006d77; right: 0; left: 0; font-size: 2.4rem; text-align: center;">{{ this.potentials[1][col] }}</p>
              <p class="math-style" style="color: #d2d2d2; z-index: 0;">v<sub>{{ col + 1 }}</sub></p>
            </td>
          </tr>
        </table>
        <table class="mx-auto mt-3 mb-5">
          <tr class="table-border" v-for="row in rowIndexes" :key="row">
            <td class="table-border" v-for="col in colIndexes" :key="col" :style="{
              backgroundColor: this.deltas[row][col] == this.maxOfDeltas[0] && row == this.maxOfDeltas[1] && col == this.maxOfDeltas[2] ? 'yellow' : 'white',
            }">
              <p class="math-style" style="font-size: 1rem;" v-if="this.deltas[row][col] != null">Δ<sub>{{ row + 1 }}{{ col + 1 }}</sub> = {{ this.deltas[row][col] }}</p>
            </td>
          </tr>
        </table>
        </div>
        <div v-for="i in (endResult.length - 1)" :key="i">
          <p class="text-center mb-4">
            Опорный план не является оптимальным, так как существуют оценки свободных клеток, для которых <span class="math-style">u<sub>i</sub> + v<sub>j</sub> > c<sub>ij</sub></span>
          </p>
          <h3 class="text-center mb-3">Итерация {{ i }}</h3>
          <p class="text-center mb-3">
            Для данного опорного плана строим следующий цикл:
          </p>
          <div class="d-flex gap-3">
            <table class="table-special mt-3 mb-5 mx-auto">
              <tr class="table-border">
                <td class="table-border"></td>
                <td v-for="col in colIndexes" :key="col" class="table-border">
                  <p class="text-center">{{ this.needsValues[col] }}</p>
                </td>
              </tr>
              <tr v-for="row in rowIndexes" :key="row" class="table-border">
                <td class="table-border">
                  <p class="text-center">{{ this.stocksValues[row] }}</p>
                </td>
                <td v-for="col in colIndexes" :key="col" class="table-border" :style="{
                  backgroundColor: row == this.endResult[i - 1][2][1] && col == this.endResult[i - 1][2][2] ? 'yellow' : 'white',
                }">
                  <span v-if="this.endResult[i - 1][1][row][col] != null" style="
                    font-size: 1.4rem;
                    color: red;
                    font-weight: 800;
                  ">{{ this.endResult[i - 1][1][row][col] }}</span>
                  <sup style="
                    font-size: 1.6rem;
                    color: green;
                    padding-left: 5px;
                  " v-if="this.endResult[i][0][row][col] == '+' || this.endResult[i][0][row][col] == '-'" >
                    {{ this.endResult[i][0][row][col] }}
                  </sup>
                  <sub style="
                    font-size: 1em;
                    font-style: normal;
                    margin-left: 40px;
                  ">{{ this.matrValues[row][col] }}</sub>
                </td>
              </tr>
            </table>
            <table class="table-special mt-3 mb-5 mx-auto">
              <tr class="table-border">
                <td class="table-border"></td>
                <td v-for="col in colIndexes" :key="col" class="table-border">
                  <p class="text-center">{{ this.needsValues[col] }}</p>
                </td>
              </tr>
              <tr v-for="row in rowIndexes" :key="row" class="table-border">
                <td class="table-border">
                  <p class="text-center">{{ this.stocksValues[row] }}</p>
                </td>
                <td v-for="col in colIndexes" :key="col" class="table-border">
                  <span v-if="this.endResult[i][1][row][col] != null" style="
                    font-size: 1.4rem;
                    color: red;
                    font-weight: 800;
                  ">{{ this.endResult[i][1][row][col] }}</span>
                  <sub style="
                    font-size: 1em;
                    font-style: normal;
                    margin-left: 40px;
                  ">{{ this.matrValues[row][col] }}</sub>
                </td>
              </tr>
            </table>
          </div>
          <p class="text-center">Значение целевой функции для данного опорного плана равно:</p>
          <p class="text-center math-style mb-3">F(x) = {{ this.endResult[i][5] }} = {{ this.endResult[i][4] }}</p>
          <p class="text-center mb-3">Найдем предварительные потенциалы <span class="math-style">u<sub>i</sub>, v<sub>j</sub></span>. по занятым клеткам таблицы, в которых <span class="math-style">u<sub>i</sub> + v<sub>j</sub> = c<sub>ij</sub></span>, полагая, что <span class="math-style">u<sub>1</sub> = 0</span>.</p>
          <p class="text-center mb-3">Выписываем оценки свободных ячеек, для которых должно выполняться условие:</p>
          <p class="text-center mb-3 math-style">u<sub>i</sub> + v<sub>j</sub> - c<sub>ij</sub> ⩽ 0</p>
          <div class="d-flex gap-3">
          <table class="table-special mt-3 mb-5 mx-auto">
          <tr class="table-border">
            <td class="table-border"></td>
            <td v-for="col in colIndexes" :key="col" class="table-border">
              <p class="text-center">{{ this.needsValues[col] }}</p>
            </td>
            <td class="table-border"><span class="math-style">u<sub>i</sub></span></td>
          </tr>
          <tr v-for="row in rowIndexes" :key="row" class="table-border">
            <td class="table-border">
              <p class="text-center">{{ this.stocksValues[row] }}</p>
            </td>
            <td v-for="col in colIndexes" :key="col" class="table-border">
              <span style="
                font-size: 1.4rem;
                color: red;
                font-weight: 800;
              ">{{ this.endResult[i][1][row][col] }}</span>
              <sub style="
                font-size: 1em;
                font-style: normal;
                margin-left: 40px;
              ">{{ this.matrValues[row][col] }}</sub>
            </td>
            <td class="table-border position-relative">
              <p class="position-absolute math-style" style="z-index: 100; color: #006d77; right: 0; left: 0; font-size: 2.4rem; text-align: center;">{{ this.endResult[i][3][0][row] }}</p>
              <p class="math-style" style="color: #d2d2d2; z-index: 0; padding: 0 10px 0 10px;">u<sub>{{ row + 1 }}</sub></p>
            </td>
          </tr>
          <tr class="table-border">
            <td class="table-border"><span class="math-style">v<sub>j</sub></span></td>
            <td v-for="col in colIndexes" :key="col" class="table-border position-relative">
              <p class="position-absolute math-style" style="z-index: 100; color: #006d77; right: 0; left: 0; font-size: 2.4rem; text-align: center;">{{ this.endResult[i][3][1][col] }}</p>
              <p class="math-style" style="color: #d2d2d2; z-index: 0;">v<sub>{{ col + 1 }}</sub></p>
            </td>
          </tr>
        </table>
        <table class="mx-auto mt-3 mb-5">
          <tr class="table-border" v-for="row in rowIndexes" :key="row">
            <td class="table-border" v-for="col in colIndexes" :key="col" :style="{
              backgroundColor: this.endResult[i][6][row][col] == this.endResult[i][2][0] && row == this.endResult[i][2][1] && col == this.endResult[i][2][2] ? 'yellow' : 'white',
            }">
              <p class="math-style" style="font-size: 1rem;" v-if="this.endResult[i][6][row][col] != null">Δ<sub>{{ row + 1 }}{{ col + 1 }}</sub> = {{ this.endResult[i][6][row][col] }}</p>
            </td>
          </tr>
        </table>
        </div>
        </div>
        <h5 class="text-center">
            Опорный план является оптимальным, так как для всех оценок выполняется условие <span class="math-style">u<sub>i</sub> + v<sub>j</sub> ⩽ c<sub>ij</sub></span>
        </h5>
        <h5 class="text-center mb-2">
            Задача решена.
        </h5>
        <h3 class="text-center mb-3">
            Ответ
        </h3>
        <p class="text-center">Минимальные затраты составят</p>
        <p class="text-center math-style mb-3">F(x) = {{ this.endResult[this.endResult.length - 1][5] }} = {{ this.endResult[this.endResult.length - 1][4] }}</p>
        <h4 class="text-center">Анализ оптимального плана</h4>
        <p class="text-center" v-for="i in endResult[endResult.length - 1][1].length">Из {{ i }}-го склада необходимо направить груз в <span v-for="j in endResult[endResult.length - 1][1][i - 1].length">{{ this.endResult[endResult.length - 1][1][i - 1][j - 1] != null ? `${j}-й магазин (${this.endResult[endResult.length - 1][1][i - 1][j - 1]} ед); ` : '' }}</span></p>
      </div>
    </BContainer>
  </template>

  <style>
    @import url(https://fonts.googleapis.com/css?family=Unbounded:200,300,regular,500,600,700,800,900);
    @import url(https://fonts.googleapis.com/css?family=Inter:100,200,300,regular,500,600,700,800,900,100italic,200italic,300italic,italic,500italic,600italic,700italic,800italic,900italic);


  * {
    font-family: 'Inter', sans-serif;
  }

  h1, h2, h3, h4, h5, h6 {
    font-family: 'Unbounded', sans-serif;
  }

  p {
    font-family: 'Inter', sans-serif;
  }

  .math-style, sub {
    font-family: Latin Modern;
    font-size: 1.5em;
    font-weight: 800;
    font-style: italic;
  }

  mrow, mi, mo, mtable, mtr, mtd, mn {
    font-family: Latin Modern Math;
    font-weight: 600;
  }

  .table-special {
    border: 1px solid black;
    border-collapse: collapse;
    width: 50%;
    margin-top: 20px;
  }

  .table-borderless {
    border: none;
    padding: 15px;
    text-align: center;
  }

  .table-border {
    border: 1px solid black;
    text-align: center;
    padding: 15px;
  }
</style>