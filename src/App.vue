<template>
  <div id="app">
    <div class="text-center">
      <h1>Coin P&L Calculator</h1>
      <p>Find out how much you're losing (or gaining) on a coin purchase.</p>
    </div>
    <form class="mt-1">
      <div>
        <label>Coin to Purchase</label>
        <div class="mt-sm">
          <em>What coin/token do you want to buy?</em>
        </div>
        <div class="mt-sm">
          <v-select
            :options="coins"
            v-model="selectedCoin"
            v-on:input="fetchPrice($event)"
          ></v-select>
        </div>
      </div>
      <div class="mt-1">
        <label>Current Price</label>
        <div class="mt-sm">
          <em>Real-time price from CoinGecko</em>
        </div>
        <div class="mt-sm strong">{{ coinPrice }}</div>
      </div>
      <div class="mt-1">
        <label>Unit Price (PHP)</label>
        <div class="mt-sm">
          <em
            >How much are they charging you for 1 {{ selectedCoin.symbol }}?</em
          >
        </div>
        <div class="mt-sm">
          <input
            type="number"
            v-model="buyingPrice"
            min="0.00"
            step="0.01"
            placeholder="1000"
            @keyup="recalculate"
          />
        </div>
      </div>
      <div class="mt-1">
        <label>Amount to Spend (PHP)</label>
        <div class="mt-sm">
          <em>How much are you going to spend?</em>
        </div>
        <div class="mt-sm">
          <input
            type="number"
            v-model="buyingAmount"
            min="0.00"
            placeholder="1000"
            step="0.01"
            @keyup="recalculate"
          />
        </div>
      </div>
      <div class="mt-1">
        <label>What you should be getting</label>
        <div class="mt-sm strong">{{ shouldGet }}</div>
      </div>
      <div class="mt-1">
        <label>What you're actually getting</label>
        <div class="mt-sm strong">{{ actualGet }}</div>
      </div>
      <div class="mt-1">
        <label>Difference</label>
        <div class="mt-sm strong" :class="differenceColor">
          {{ difference }}
        </div>
      </div>
    </form>
    <div class="text-center mt-1">
      Made with ❤ by Liam Demafelix
      <br /><br />
      Found something wrong?
      <a href="mailto:hey@liam.ph">Let me know</a>.<br /><br />
      <a href="https://github.com/liamdemafelix/coin-calc">View Source</a>
      <br /><br />
      🚀 🌑
    </div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "App",
  data() {
    return {
      coins: [],
      selectedCoin: { code: "tether", label: "USDT - Tether", symbol: "USDT" },
      coinPrice: "PHP 0.00",
      coinpriceDec: 1,
      buyingPrice: 0,
      buyingAmount: 0,
      shouldGet: "Loading...",
      shouldGetDec: 0,
      actualGet: "Loading...",
      actualGetDec: 0,
      difference: "Loading...",
      differenceDec: 0,
      differenceColor: "normal",
    };
  },
  methods: {
    refreshCoinList() {
      let vScope = this;
      vScope.coins = [];
      axios.get("https://api.coingecko.com/api/v3/coins/list").then((resp) => {
        for (let x = 0; x < resp.data.length; x++) {
          vScope.coins.push({
            label:
              resp.data[x].symbol.toUpperCase() + " - " + resp.data[x].name,
            code: resp.data[x].id,
            symbol: resp.data[x].symbol.toUpperCase(),
          });
        }
      });
    },
    fetchPrice() {
      let vScope = this;
      vScope.coinPrice = "Loading...";
      axios
        .get(
          "https://api.coingecko.com/api/v3/simple/price?ids=" +
            vScope.selectedCoin.code +
            "&vs_currencies=php"
        )
        .then((resp) => {
          vScope.coinPrice =
            "PHP " +
            resp.data[vScope.selectedCoin.code].php.toLocaleString("en-US", {
              maximumFractionDigits: 2,
              minimumFractionDigits: 2,
            });
          vScope.coinPriceDec = resp.data[vScope.selectedCoin.code].php;
        });
    },
    recalculate() {
      this.shouldGetDec = this.buyingAmount / this.coinPriceDec;
      this.shouldGet = this.selectedCoin.symbol + " " + this.shouldGetDec;
      this.actualGetDec = this.buyingAmount / this.buyingPrice;
      this.actualGet = this.selectedCoin.symbol + " " + this.actualGetDec;
      this.differenceDec = this.actualGetDec - this.shouldGetDec;
      this.difference = this.selectedCoin.symbol + " " + this.differenceDec;
      if (this.differenceDec < 0) {
        this.differenceColor = "red";
      } else if (this.differenceDec > 0) {
        this.differenceColor = "green";
      } else {
        this.differenceColor = "normal";
      }
    },
  },
  mounted() {
    this.refreshCoinList();
    this.fetchPrice();
    setInterval(() => {
      this.fetchPrice();
    }, 15000)
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  max-width: 500px;
  margin: 40px auto;
}

.mt-sm {
  margin-top: 0.5em;
}

.mt-1 {
  margin-top: 2em;
}

.strong {
  font-weight: bold;
}

.text-center {
  text-align: center;
}

label {
  font-weight: bold;
}

.red {
  color: red;
}

.green {
  color: green;
}

.normal {
  color: #2c3e50;
}

input {
  width: 99%;
  color: #2c3e50;
  border: 1px solid rgba(60, 60, 60, 0.26);
  border-radius: 4px;
  padding: 8px 4px;
}
</style>
