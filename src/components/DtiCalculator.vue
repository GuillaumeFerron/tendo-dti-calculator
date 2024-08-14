<template>
  <div class="w-100">
    <div class="row">
      <h5>Calculator</h5>
      <div class="col-12 col-lg-6">
        <div>
          <label for="dti">DtI (%)</label>
          <input type="number" class="form-control" step="1" id="dti" v-model="data.dti">
        </div>
        <div class="mt-3">
          <label for="payroll">Number of Payroll Dates</label>
          <select name="Payroll" id="payroll" class="form-control" v-model="data.payroll">
            <option value="1">1</option>
            <option value="2">2</option>
          </select>
        </div>
        <div class="mt-3">
          <label for="net-income">Net Income</label>
          <input type="number" class="form-control" step=".01" id="net-income" v-model="data.netIncome">
        </div>
      </div>
      <div class="col-12 col-lg-6">
        <div>
          <label for="schedule">Schedule (copied from user's account)</label>
          <textarea class="form-control" id="schedule" v-model="data.schedule" rows="4"></textarea>
        </div>
        <div class="mt-3">
          <p class="py-1"></p>
          <button class="form-control btn btn-light" @click="clear">Clear</button>
        </div>
      </div>
    </div>
    <hr class="mt-4">
    <div class="row mt-3">
      <h5>Schedule</h5>
      <small><i>The highlighted cells will show the first payment for which the available amount is 50% of the total
          amount
          the customer could avail if no pending payment (PHP{{ maxPayment }}). If none is highlighted, it means that the user will have to repay their last payment in order to avail a new loan.</i></small>
      <div class="my-2">
        <table class="table table-striped table-sm">
          <thead>
            <tr>
              <th scope="col">Due</th>
              <th scope="col">Amount</th>
              <th scope="col">Penalty</th>
              <th scope="col">Total Amount</th>
              <th scope="col">Available Amount Once Due Repaid</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="row in scheduleWithAvailable" :key="row[0]" :class="testAvailable(row) ? 'table-info' : ''">
              <td scope="row">{{ row[0] }}</td>
              <td>{{ row[1] }}</td>
              <td>{{ row[2] }}</td>
              <td>{{ row[3] }}</td>
              <td>{{ row[4] }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
  const CACHE_KEY = 'TENDOPAY_IR_CORRECTOR'
  const FORCE_CACHE = true

  export default {
    data() {
      return {
        data: {
          dti: 30,
          payroll: 2,
          netIncome: null,
          schedule: null
        },
        cacheProcessed: false
      }
    },
    watch: {
      data: {
        deep: true,
        handler() {
          if (this.cacheProcessed) {
            this.storeCache()
          }
        }
      }
    },
    mounted() {
      this.restoreCache()
    },
    computed: {
      computedSchedule () {
        return (this.data.schedule || '')
        .split(/\r?\n|\r|\n/g)
          .map(_ => _.replace(/((₱ )|(,))/g, '').split(/\t/g))
      },
      scheduleWithAvailable () {
        return this.computedSchedule.map(_ => ([..._, Math.max(Math.ceil(((this.data.dti / 100) * this.data.netIncome / this.data.payroll - _[3]) * 100) / 100, 0)]))
      },
      maxPayment() {
        return Math.ceil((this.data.dti / 100) * this.data.netIncome / this.data.payroll * 100) / 100
      }
    },
    methods: {
      testAvailable(row) {
        return row[4] >= 0.5 * this.maxPayment
      },
      storeCache() {
        const obj = {
          data: this.data
        }
        localStorage.setItem(CACHE_KEY, JSON.stringify(obj))
      },
      restoreCache() {
        const res = localStorage.getItem(CACHE_KEY)
        if (res) {
          let clear = {}
          try {
            clear = JSON.parse(res)
          } catch (err) {
            clear = {}
          }
          if (!FORCE_CACHE) {
            this.clear()
            console.log(`* DTI ADJUSTER: Invalidated cache`)
          } else {
            const data = clear.data
            Object.keys(data || {}).forEach(_ => {
              if (data[_] != null) {
                this.data[_] = data[_]
              }
            })
            console.log(`* DTI ADJUSTER: Restored cache`)
          }
        }
        this.cacheProcessed = true
      },
      clear() {
        Object.keys(this.data || {}).forEach(_ => {
          if (typeof this.data[_] === 'object') {
            this.data[_] = []
          } else {
            this.data[_] = null
          }
        })
        localStorage.removeItem(CACHE_KEY)
      }
    }
  }
</script>

<style scoped lang="scss">
</style>