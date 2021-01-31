<template>
  <div class="container bg-white my-5 p-5 rounded-lg shadow">
    <h1>bitmoon.net fast price</h1>

    <div class="row">
      <div class="col-sm-12 col-md-12 py-4">
        <label>coin name</label>

        <div class="input-group">
          <input
            type="text"
            class="form-control"
            v-model="coin"
            @change="fetch()"
          />

          <div class="input-group-append">
            <button
              type="button"
              class="btn btn-outline-primary"
              @click="fetch()"
            >
              search
            </button>
          </div>
        </div>
      </div>

      <div class="col-sm-12 col-md-6 py-4">
        <h5>go buy</h5>
        <h5 class="border border-success text-center rounded p-4">
          <span v-if="isLoading">?</span>
          <span v-else>{{ Number(sell).toLocaleString() }} vnd</span>
        </h5>
      </div>

      <div class="col-sm-12 col-md-6 py-4">
        <h5>go sell</h5>
        <h5 class="border border-danger text-center rounded p-4">
          <span v-if="isLoading">?</span>
          <span v-else>{{ Number(buy).toLocaleString() }} vnd</span>
        </h5>
      </div>

      <div class="col-sm-12 col-md-6 py-4">
        <div class="form-group">
          <label>profit buy</label>
          <input
            type="number"
            class="form-control"
            v-model="changeSell"
            @change="calculator('coinToVnd')"
          />
        </div>

        <label>amount coin</label>
        <div class="form-group">
          <input
            type="number"
            class="form-control"
            v-model="amountCoinSell"
            @change="calculatorSell"
          />
        </div>

        <label>amount vnd</label>
        <div class="form-group">
          <input
            type="number"
            class="form-control"
            v-model="amountVndSell"
            @change="calculatorSellToCoin"
          />
        </div>
      </div>

      <div class="col-sm-12 col-md-6 py-4">
        <div class="form-group">
          <label>profit sell</label>
          <input
            type="number"
            class="form-control"
            v-model="changeBuy"
            @change="calculator('coinToVnd')"
          />
        </div>

        <label>amount coin</label>
        <div class="form-group">
          <input
            type="number"
            class="form-control"
            v-model="amountCoinBuy"
            @change="calculatorBuy"
          />
        </div>

        <label>amount vnd</label>
        <div class="form-group">
          <input
            type="number"
            class="form-control"
            v-model="amountVndBuy"
            @change="calculatorBuyToCoin"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      isLoading: false,
      coin: "tron",
      buy: 0,
      changeBuy: 0,
      sell: 0,
      changeSell: 0,
      amountCoinSell: 0,
      amountVndSell: 0,
      amountCoinBuy: 0,
      amountVndBuy: 0,
    };
  },
  mounted() {
    this.fetch();
    setInterval(() => this.fetch(), 6e4);
  },
  methods: {
    fetch: function () {
      this.isLoading = !this.isLoading;
      axios({
        url: "https://api-prod.bitmoon.net/graphql",
        method: "POST",
        data: {
          query:
            "\n    query getPriceBook ($coin_id: String) {\n      apiPricebook (coin_id: $coin_id){\n        coin_id,\n        bid_price_vnd,\n        fast_bid_price,\n        fast_ask_price,\n        ask_price_vnd,\n        fast,\n        normal,\n        is_direct,\n        broker_code,\n        change_24h,\n        volume,\n        coin,\n        trade_buy_fee,\n        trade_sell_fee,\n        base_currency\n      }\n    }\n  ",
          variables: { coin_id: this.coin },
        },
      }).then((response) => {
        this.isLoading = !this.isLoading;
        let res = response.data;
        console.log(res);
        let result = res.data.apiPricebook[0];
        this.buy = result.fast_bid_price;
        this.sell = result.fast_ask_price;
      });
    },
    calculatorSell: function () {
      this.amountVndSell =
        Number(this.amountCoinSell) *
        (Number(this.sell) + Number(this.changeSell));
    },
    calculatorSellToCoin: function () {
      this.amountCoinSell =
        Number(this.amountVndSell) /
        (Number(this.sell) + Number(this.changeSell));
    },
    calculatorBuy: function () {
      this.amountVndBuy =
        Number(this.amountCoinBuy) *
        (Number(this.buy) + Number(this.changeBuy));
    },
    calculatorBuyToCoin: function () {
      this.amountCoinBuy =
        Number(this.amountVndBuy) / (Number(this.buy) + Number(this.changeBuy));
    },
  },
};
</script>

<style>
</style>