<template>
  <div class="container">
    <div class="row mt-4">
      <div class="col-sm-12 col-md-12">
        <h3 class="display-4 text-primary">Giá mặt định: bitmoon.net</h3>
      </div>
    </div>

    <div class="row mt-4">
      <div class="col-sm-12 col-md-12">
        <label>Coin</label>

        <div class="input-group">
          <input
            type="text"
            class="form-control"
            v-model="coin"
            @change="fetch()"
          />

          <div class="input-group-append">
            <button type="button" class="btn btn-primary" @click="fetch()">
              Tìm
            </button>
          </div>
        </div>
      </div>
    </div>

    <div class="row mt-4">
      <div class="col-sm-12 col-md-6 text-center">
        <h5>Giá bán ra</h5>

        <h5 class="border border-success bg-success text-white rounded p-4">
          <span v-if="isLoading">?</span>
          <span v-else>{{ Number(sell).toLocaleString() }}</span>
        </h5>
      </div>

      <div class="col-sm-12 col-md-6 text-center">
        <h5>Giá mua vào</h5>
        <h5 class="border border-danger bg-danger text-white rounded p-4">
          <span v-if="isLoading">?</span>
          <span v-else>{{ Number(buy).toLocaleString() }}</span>
        </h5>
      </div>
    </div>

    <div class="row mt-4">
      <div class="col-sm-12 col-md-6">
        <div class="form-group">
          <label>Lệch giá bán ra</label>
          <input
            type="number"
            class="form-control"
            v-model="changeSell"
            @change="calculator('coinToVnd')"
          />
        </div>
      </div>

      <div class="col-sm-12 col-md-6">
        <div class="form-group">
          <label>Lệch giá mua vào</label>
          <input
            type="number"
            class="form-control"
            v-model="changeBuy"
            @change="calculator('coinToVnd')"
          />
        </div>
      </div>

      <div class="col-sm-12 col-md-6">
        <h3>Bán ra</h3>

        <label>Coin</label>
        <div class="input-group">
          <input
            type="number"
            class="form-control"
            v-model="amountCoinSell"
            @change="calculatorSell"
          />
        </div>

        <label>Vnd</label>
        <div class="input-group">
          <input
            type="number"
            class="form-control"
            v-model="amountVndSell"
            @change="calculatorSellToCoin"
          />
        </div>
      </div>

      <div class="col-sm-12 col-md-6">
        <h3>Mua vào</h3>

        <label>Coin</label>
        <div class="input-group">
          <input
            type="number"
            class="form-control"
            v-model="amountCoinBuy"
            @change="calculatorBuy"
          />
        </div>

        <label>Vnd</label>
        <div class="input-group">
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
      coin: "doge",
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
            "\n    query getPriceBookPaging ($page_size: Int, $page: Int, $sort_name: String, $sort_type: String, $search: String) {\n      apiPricebookPaging (page_size: $page_size, page: $page, sort_name: $sort_name, sort_type: $sort_type, search: $search){\n        total,\n        data {\n        id,\n        coin_id,\n        coin_name,\n        bid_price_usd,\n        fast_bid_price,\n        ask_price_usd,\n        bid_price_btc,\n        ask_price_btc,\n        bid_price_vnd,\n        ask_price_vnd,\n        fast_ask_price,\n        fast,\n        normal,\n        sort_no,\n        buy_profit,\n        sell_profit,\n        change_24h,\n        volume,\n        coin\n      }\n    }\n  }\n",
          variables: {
            page_size: 50,
            page: 1,
            sort_name: "volume",
            sort_type: "desc",
            search: this.coin,
          },
        },
      }).then((response) => {
        this.isLoading = !this.isLoading;
        // console.log(response.data.data.apiPricebookPaging.data[0]);
        let result = response.data.data.apiPricebookPaging.data[0];
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