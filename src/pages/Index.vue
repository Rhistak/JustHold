<template>
  <div class="q-pa-md">
    <transition-group
      appear
      enter-active-class="animated fadeInDown"
      leave-active-class="animated slideOutUp"
    >
      <q-form @submit="addCurrency" v-if="show" key="QForm">
        <q-select
          filled
          v-model="crypto"
          label="Crypto"
          use-input
          hide-selected
          fill-input
          input-debounce="0"
          :options="keyWord"
          @filter="filterFn"
          lazy-rules
          label-color="white"
          bg-color="accent"
          :rules="[val => (val && val.length > 0) || 'Please type something']"
        >
          <template v-slot:no-option>
            <q-item>
              <q-item-section class="text-grey">
                No results
              </q-item-section>
            </q-item>
          </template>
        </q-select>

        <q-input
          filled
          type="number"
          v-model="amount"
          label="Amount"
          label-color="white"
          bg-color="accent"
          lazy-rules
          :rules="[
            val => (val !== null && val !== '') || 'Please type something'
          ]"
        />
        <q-input
          filled
          type="float"
          v-model="price"
          label="Buy price"
          label-color="white"
          bg-color="accent"
          lazy-rules
          :rules="[
            val => (val !== null && val !== '') || 'Please type something'
          ]"
        />
        <div style="display: flex;flex-direction: column-reverse;">
          <q-btn label="ADD" type="submit" class="right" color="primary" />
        </div>
      </q-form>
    </transition-group>

    <div
      v-for="currency in currencies"
      :key="currency.id"
      style="max-width: 400px"
    >
      <q-list separator class="q-ma-md">
        <q-item style="background-color:#6e7c7c" class="rounded-borders">
          <div v-ripple v-for="coin in coins" :key="coin.id">
            <q-item-section
              v-if="coin.name.toUpperCase() == currency.crypto.toUpperCase()"
            >
              <q-item-label style="color:#ffffff;font-size:large"
                >{{ currency.crypto }}
              </q-item-label>
              <q-item-label overline style="color:#f2edd7">{{
                currency.amount
              }}</q-item-label>
            </q-item-section>
            <q-item-section
              class="absolute-right q-mr-md"
              v-if="coin.name.toUpperCase() == currency.crypto.toUpperCase()"
            >
            <div v-if="(currency.amount *(coin.current_price - currency.price) > 0)">
              <q-item-label overline style="color:#9ede73;font-size:large" >
               ${{(currency.amount *(coin.current_price - currency.price)).toFixed(2)}}
                </q-item-label
              >
            </div>
            <div v-else>
              <q-item-label overline style="color:#ff7171;font-size:large">
                ${{(currency.amount *(coin.current_price - currency.price)).toFixed(2)}}
              </q-item-label>
            </div>
            </q-item-section>
          </div>
        </q-item>
      </q-list>
    </div>

    <q-page-sticky position="bottom-right" :offset="[18, 18]">
      <q-btn fab icon="add" color="primary" v-on:click="show = !show" />
    </q-page-sticky>
  </div>
</template>

<script>
import Localbase from "localbase";
import axios from "axios";
let db = new Localbase("db");

export default {
  data() {
    return {
      //form input
      crypto: "",
      amount: "",
      price: "",
      timer: "",
      //form visibility
      show: false,
      //get userDbdata
      currencies: [],
      someCurrencies: [],
      //search options
      options: [],
      //search keyword
      keyWord: [],
      //coins data
      coins: [],
      coinData: [],
      //test
      condition: false
    };
  },
  methods: {
    getCoinsData() {
      axios
        .get(
          "https://api.coingecko.com/api/v3/coins/markets?vs_currency=usd&ids=&order=market_cap_desc&per_page=250&page=1&sparkline=false&price_change_percentage=1h"
        )
        .then(response => {
          // this.options = response.data.filter(e => (e.name).toLowerCase() == this.crypto);
          this.options = response.data.map(e => e.name);
          this.coins = response.data;
          console.log("api call");
        });
    },
    addCurrency() {
      let newCurrency = {
        id: Date.now(),
        crypto: this.crypto,
        amount: this.amount,
        price: this.price
      };
      db.collection("Currencies").add(newCurrency);
      this.currencies.push(newCurrency);
    },
    getCurrencys() {
      db.collection("Currencies")
        .get()
        .then(currency => {
          const seen = new Set();
          this.currencies = currency.filter(el => {
            const duplicate = seen.has(el.crypto);
            seen.add(el.crypto);
            return !duplicate;
          });
        });
    },
    getDuplicated() {
      db.collection("Currencies")
        .get()
        .then(currency => {
          const someSeen = new Set();
          this.someCurrencies = currency.filter(el => {
            const duplicate = someSeen.has(el.crypto);
            someSeen.add(el.crypto);
            return duplicate;
          });
          console.log(this.someCurrencies);
        });
    },
    filterFn(val, update, abort) {
      update(() => {
        const needle = val.toLowerCase();
        this.keyWord = this.options.filter(
          v => v.toLowerCase().indexOf(needle) > -1
        );
      });
    },
    startInterval: function() {
      setInterval(() => {
        this.getCoinsData();
      }, 30000);
    },
  },
  created() {
    this.getDuplicated();
    this.getCurrencys();
    this.startInterval();
  },
  mounted() {
    this.getCoinsData();
  },
  name: "PageIndex"
};
</script>
