<template>
  <div id="app">
    <form class="form">
      <div class="controls">
        <input
          :value="values.price"
          @input="updateValueDebounced('price', $event.target.value)"
          type="number"
          step="1"
        />
        <input
          :value="values.qty"
          @input="updateValueDebounced('qty', $event.target.value)"
          type="number"
          step="1"
        />
        <input
          :value="values.amount"
          @input="updateValueDebounced('amount', $event.target.value)"
          type="number"
          step="1"
        />
        <button @click.prevent="handleClick">submit</button>
      </div>
      <div class="controls">
        <label>{{ values.price }}</label>
        <label>{{ values.qty }}</label>
        <label>{{ values.amount }}</label>
        <label>
          {{ localStorageCopy }}
        </label>
      </div>
      <div class="controls">
        <ul class="output">
          <li v-for="(item, index) in log" :key="index">{{ item }}</li>
        </ul>
      </div>
    </form>
  </div>
</template>

<script>
import debounce from "lodash/debounce";
export default {
  name: "App",
  components: {},
  debounce,
  data() {
    return {
      values: {
        price: 0,
        qty: 0,
        amount: 0,
      },
      queue: ["price", "qty", "amount"],
      nonce: 0,
      log: [],
      localStorageCopy: "",
    };
  },
  mounted() {
    this.cloneLocalStorage();
  },
  methods: {
    moveToEndQueue(value) {
      const q = this.queue;
      q.push(q.splice(q.indexOf(value), 1)[0]);
    },

    recalculate() {
      const q = this.queue;
      const v = this.values;
      if (q[0] == "qty") {
        v.qty = v.amount / v.price;
      }
      if (q[0] == "amount") {
        v.amount = v.qty * v.price;
      }
      if (q[0] == "price") {
        v.price = v.amount / v.qty;
      }
    },

    updateValue(property, newValue) {
      this.values[property] = newValue;
      this.addLog(`${property} input was changed`);
      this.moveToEndQueue(property);
      this.recalculate();
    },

    updateValueDebounced: debounce(function (property, newValue) {
      this.updateValue(property, newValue);
    }, 300),

    handleClick() {
      this.addLog("button was pressed");
      setTimeout(() => {
        if (this.values.amount % 2 === 0) {
          const data = JSON.stringify({ ...this.values, nonce: ++this.nonce });
          localStorage.setItem("savedData", data);
          this.cloneLocalStorage();
          this.addLog("{success:true}");
        } else {
          this.addLog("{success:false}");
        }
      }, 1000);
    },

    addLog(operation) {
      this.log.push(operation);
    },

    cloneLocalStorage() {
      this.localStorageCopy = JSON.parse(localStorage.getItem("savedData"));
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 120px;
}

* {
  box-sizing: border-box;
}

body {
  background-color: #ebeef3;
  margin: 0;
}

.form {
  width: 45%;
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 0 auto;
  gap: 20px;
}

.controls {
  width: 100%;
  display: flex;
  justify-content: space-between;
  gap: 20px;
}

.controls > * {
  width: 180px;
}

.controls label {
  text-align: left;
}

.controls ul {
  padding: 0;
  margin: 0;
  list-style: none;
  text-align: left;
}

.controls:last-child {
  align-self: flex-start;
}

.output {
  display: flex;
  flex-direction: column-reverse;
  align-items: flex-start;
  gap: 6px;
}

.output p {
  margin: 0;
}
</style>
