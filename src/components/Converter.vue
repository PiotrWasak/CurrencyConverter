<template lang="pl">
    <div class="container">
        <div class="row">
            <div class="col-3"> <input v-model="userInput" type="text" class="form-control"></div>
            <div class="col-2">
                <select  class="form-select form-select ">
                    <option>PLN</option>
                </select>
                </div>
            <div class="col-2">🔄</div>
            <div class="col-3"><input :value="convertedValue" type="text" class="form-control"></div>
            <div class="col-2"> <select @change="setMoneyValue" class="form-select form-selec">
                    <option value="0" selected>Open this select menu</option>
                    <option v-for="item in currencies" :key="item.id" :value="item[1]">{{ item[0] }}</option>
                </select></div>
        </div>
    </div>
</template>
<script>

export default {
  name: 'Converter',
  data() {
    return {
      currencies: new Map(),
      userInput: "",
      results: {},
      moneyValue: 0,
    }
  },
  methods: {
    setMoneyValue() {
      const options = event.target.options;
      const selectedIndex = options.selectedIndex;
      this.moneyValue = options[selectedIndex].value;
      console.log(this.moneyValue);
    },
    fetchAPI() {
      fetch('http://api.nbp.pl/api/exchangerates/tables/A')
        .then(response => response.body)
        .then(rb => {
          const reader = rb.getReader();

          return new ReadableStream({
            start(controller) {
              // The following function handles each data chunk
              function push() {
                // "done" is a Boolean and value a "Uint8Array"
                reader.read().then(({ done, value }) => {
                  // If there is no more data to read
                  if (done) {
                    controller.close();
                    return;
                  }
                  // Get the data and send it to the browser via the controller
                  controller.enqueue(value);
                  // Check chunks by logging to the console
                  push();
                })
              }
              push();
            }
          });
        })
        .then(stream => {
          // Respond with our stream
          return new Response(stream, { headers: { "Content-Type": "text/html" } });
        })
        .then(result => result.json())
        .then((data) => {
          //console.log("data", JSON.parse(JSON.stringify(data)));
          this.results = JSON.parse(JSON.stringify(data));
          //console.log("this.results", this.results);
          return this.results;
        });
    },
    saveResults: async function () {
      const result = this.fetchAPI();
      return result;
    },
    createCurrencyArray() {
      this.results[0].rates.forEach((item) => {
        this.currencies.set(item.code, item.mid);
      });
      console.log(this.currencies);
    }
  },
  computed: {
    convertedValue: function () {
      if (!this.userInput) return;
      if (this.moneyValue == 0) return;
      console.log(this.userInput);
      return (this.userInput / this.moneyValue).toFixed(2);
    }
  },
  created: function () {
    this.saveResults();
    setTimeout(() => {
      this.createCurrencyArray();
    }, 500);
  }
}
</script>
<style scoped>
</style>