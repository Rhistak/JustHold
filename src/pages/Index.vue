<template>
  <div class="q-pa-md" v-if="coins">
    <transition-group
      appear
      enter-active-class="animated fadeInDown"
      leave-active-class="animated slideOutUp"
    >
      <!-- FORM -->
      <q-form @submit="addCurrency" v-if="show" key="QForm" class="q-mb-md">
        <q-select
          filled
          v-model="crypto"
          placeholder="Select Crypto"
          use-input
          hide-selected
          fill-input
          input-debounce="0"
          :options="keyWord"
          @filter="filterFn"
          lazy-rules
          label-color="white"
          bg-color="accent"
          :rules="[val => val && val.length > 0]"
        >
          <template v-slot:no-option>
            <q-item>
              <q-item-section class="text-grey">
                No results
              </q-item-section>
            </q-item>
          </template>
        </q-select>
        <div class="inputDiv">
          <q-input
            filled
            type="text"
            v-model="amount"
            placeholder="Amount"
            bg-color="accent"
            lazy-rules
            :rules="[val => val !== null && val !== '']"
          />
          <q-input
            filled
            type="text"
            v-model="price"
            placeholder="Buy Price"
            label-color="white"
            bg-color="accent"
            lazy-rules
            :rules="[val => val !== null && val !== '']"
          />
        </div>
        <div class="btnDiv">
          <q-btn label="add" type="submit" class="right" icon="add_box" />
        </div>
      </q-form>
    </transition-group>
    <!-- ITEM  HEADER -->
    <div class="separatorItems">
      <q-list v-for="s in someCurrencies" :key="s.id">
        <div v-for="coin in coins" :key="coin.id">
          <q-expansion-item
            v-if="coin.name.toLowerCase() == s.name.toLowerCase()"
            default-opened
            header-class="bg-secondary text-white rounded-borders q-mb-sm"
          >
            <!-- ITEM  HEADER NAME -->
            <template v-slot:header>
              <q-item-section>
                {{ s.name }}
              </q-item-section>
              <!-- ITEM  HEADER PERCENT -->
              <q-item-section class="absolute-center">
                <div v-if="coin.price_change_percentage_24h > 0">
                  <q-badge
                    rounded-borders
                    color="positive"
                    class="q-pa-xs"
                    style="font-weight:bold"
                  >
                    {{ coin.price_change_percentage_24h.toFixed(2) }}%
                  </q-badge>
                </div>
                <div v-else-if="coin.price_change_percentage_24h < 0">
                  <q-badge
                    rounded-borders
                    color="rgb(255, 113, 113)"
                    class="q-pa-xs"
                    style="font-weight:bold"
                  >
                    {{ coin.price_change_percentage_24h.toFixed(2) }}%
                  </q-badge>
                </div>
                <div v-else>
                  <q-badge
                    rounded-borders
                    color="primary"
                    class="q-pa-xs"
                    style="font-weight:bold"
                  >
                    {{ coin.price_change_percentage_24h.toFixed(2) }}%
                  </q-badge>
                </div>
              </q-item-section>
              <!-- ITEM  HEADER PRICE -->
              <q-item-section class="absolute-right q-mr-xl">
                <q-item-label style="margin:0;margin-left:.5em">
                  ${{ coin.current_price }}
                </q-item-label>
              </q-item-section>
            </template>
            <q-list v-for="c in currencies" :key="c.id" separator>
              <!-- SLIDE ITEM  -->
              <q-slide-item
                @right="deleteCurrency(c.id)"
                dark
                class="q-ma-sm rounded-borders"
                right-color="primary"
                v-if="c.name == s.name"
                :class="c.name"
              >
                <template v-slot:right>
                  <q-icon name="delete" />
                </template>
                <!-- ITEM  -->
                <q-item
                  style="background-color:#5d7290"
                  class="rounded-borders"
                >
                  <div v-for="coin in coins" :key="coin.id">
                    <!-- COIN DATE  -->
                    <q-item-section
                      class="absolute-left"
                      v-if="coin.name.toLowerCase() == c.name.toLowerCase()"
                    >
                      <q-item-label
                        class="q-ml-md"
                        overline
                        style="color:#f2edd7"
                        v-if="dateOrBuy === false"
                        v-on:click="dateOrBuy = !dateOrBuy"
                      >
                        {{ c.id | niceDate }}
                        <!-- ${{c.price}} -->
                      </q-item-label>

                      <q-item-label
                        class="q-ml-md"
                        overline
                        style="color:#f2edd7"
                        v-else
                        v-on:click="dateOrBuy = !dateOrBuy"
                      >
                        ${{ c.price }}
                        <!-- ${{c.price}} -->
                      </q-item-label>
                    </q-item-section>
                    <!-- COIN AMOUNT  -->
                    <q-item-section
                      class="absolute-center"
                      side
                      v-if="coin.name.toLowerCase() == c.name.toLowerCase()"
                    >
                      <q-badge rounded color="warning" class="q-pa-sm">{{
                        c.amount
                      }}</q-badge>
                    </q-item-section>
                    <!-- COIN PRICE -->
                    <q-item-section
                      class="absolute-right q-mr-md"
                      v-if="coin.name.toLowerCase() == c.name.toLowerCase()"
                    >
                      <div v-if="c.amount * (coin.current_price - c.price) > 0">
                        <q-item-label overline style="color:#9ede73">
                          ${{
                            (c.amount * (coin.current_price - c.price)).toFixed(
                              2
                            )
                          }}
                        </q-item-label>
                      </div>
                      <div
                        v-else-if="
                          c.amount * (coin.current_price - c.price) < 0
                        "
                      >
                        <q-item-label overline style="color:#ff7171">
                          ${{
                            (c.amount * (coin.current_price - c.price)).toFixed(
                              2
                            )
                          }}
                        </q-item-label>
                      </div>
                      <div v-else>
                        <q-item-label overline style="color:#bbbbbb">
                          ${{
                            (c.amount * (coin.current_price - c.price)).toFixed(
                              2
                            )
                          }}
                        </q-item-label>
                      </div>
                    </q-item-section>
                  </div>
                </q-item>
              </q-slide-item>
            </q-list>
          </q-expansion-item>
        </div>
      </q-list>
    </div>

    <!-- FUNCTION BUTTON -->

    <q-page-sticky position="bottom-right" :offset="[18, 18]">
      <q-btn fab icon="add" color="primary" @click="actionAddButton()" />
    </q-page-sticky>
  </div>
  <!-- PRELOAD -->
  <div v-else class="q-pa-md">
    <!-- FORM -->
    <q-form @submit="addCurrency" v-if="show" key="QForm" class="q-mb-md">
      <q-select
        filled
        v-model="crypto"
        placeholder="Select Crypto"
        use-input
        hide-selected
        fill-input
        input-debounce="0"
        :options="keyWord"
        @filter="filterFn"
        lazy-rules
        label-color="white"
        bg-color="accent"
        :rules="[val => val && val.length > 0]"
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
        type="text"
        v-model="amount"
        placeholder="Amount"
        bg-color="accent"
        lazy-rules
        :rules="[val => val !== null && val !== '']"
      />
      <q-input
        filled
        type="text"
        v-model="price"
        placeholder="Buy Price"
        label-color="white"
        bg-color="accent"
        lazy-rules
        :rules="[val => val !== null && val !== '']"
      />
      <div style="display: flex;flex-direction: column-reverse;">
        <q-btn
          label="ADD"
          type="submit"
          class="right"
          style="background-color:#e84545;color:#ececec"
          icon="add_box"
        />
      </div>
    </q-form>
    <!-- $KELETON -->
    <div v-for="c in someCurrencies" :key="c.id" class="q-mb-md">
      <q-card-section style="margin:0;padding:0">
        <q-skeleton type="QToolbar" style="background-color:#4b5d67" />
      </q-card-section>
    </div>
    <!-- FUNCTION BUTTON -->
    <q-page-sticky position="bottom-right" :offset="[18, 18]">
      <q-btn fab icon="add" color="primary" @click="actionAddButton()" />
    </q-page-sticky>
  </div>
</template>

<script>
import Localbase from "localbase";
import axios from "axios";
import { date } from "quasar";
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
      //delete visibility
      deleteVisibility: false,
      //get userDbdata
      currencies: null,
      someCurrencies: null,
      //search options
      options: [],
      //search keyword
      keyWord: [],
      //coins data
      coins: null,
      //action button
      fabPos: [18, 18],
      draggingFab: false,
      // scroll UP
      visible: false,
      // date or buyPrice visibility
      dateOrBuy: false
    };
  },
  // METHODS //
  methods: {
    getCoinsData() {
      axios
        .get(
          "https://api.coingecko.com/api/v3/coins/markets?vs_currency=usd&ids=&order=market_cap_desc&per_page=250&page=1&sparkline=false&price_change_percentage=1h"
        )
        .then(response => {
          this.options = response.data.map(e => e.name);
          const seen = new Set();
          this.someCurrencies.forEach(x => {
            response.data.filter(e => {
              if (e.name.toLowerCase() == x.name.toLowerCase()) {
                seen.add(e);
                return (this.coins = seen);
              }
            });
          });
          console.log("api call");
        });
    },
    addCurrency() {
      let newCurrency = {
        id: Date.now(),
        name: this.crypto,
        amount: this.amount,
        price: this.price
      };
      db.collection("Currencies").add(newCurrency);
      this.currencies.unshift(newCurrency);
      const seen = new Set();
      this.someCurrencies = this.currencies.filter(el => {
        const duplicate = seen.has(el.name);
        seen.add(el.name);
        return !duplicate;
      });

      this.getCoinsData();
      this.show = false;
      this.crypto = "";
      this.price = "";
      this.amount = "";
    },
    getCurrencys() {
      db.collection("Currencies")
        .get()
        .then(currency => {
          this.currencies = currency;
          const seen = new Set();
          this.someCurrencies = this.currencies.filter(el => {
            const duplicate = seen.has(el.name);
            seen.add(el.name);
            return !duplicate;
          });
        });
    },
    deleteCurrency(id) {
      let index = this.currencies.findIndex(c => c.id === id);
      db.collection("Currencies")
        .doc({ id: id })
        .delete();
      this.currencies.splice(index, 1);
      const seen = new Set();
      this.someCurrencies = this.currencies.filter(el => {
        const duplicate = seen.has(el.name);
        seen.add(el.name);
        return !duplicate;
      });
    },
    //action add button > visibility and scroll up
    actionAddButton() {
      this.intervalId = setInterval(() => {
        if (window.pageYOffset === 0) {
          clearInterval(this.intervalId);
        }
        window.scroll(0, window.pageYOffset - 50);
      }, 20);

      this.visible = window.scrollY > 150;

      this.show = !this.show;
      this.deleteVisibility = false;
    },
    //api call every x seconds
    startCoinsApiCall: function() {
      setInterval(() => {
        this.getCoinsData();
      }, 2500);
    },
    //filter keywords
    filterFn(val, update, abort) {
      if (val.length < 2) {
        abort();
        return;
      }
      update(() => {
        const needle = val.toLowerCase();
        this.keyWord = this.options.filter(
          v => v.toLowerCase().indexOf(needle) > -1
        );
      });
    },
    //action button
    moveFab(ev) {
      this.draggingFab = ev.isFirst !== true && ev.isFinal !== true;

      this.fabPos = [this.fabPos[0] - ev.delta.x, this.fabPos[1] - ev.delta.y];
    }
  },
  //date filter
  filters: {
    niceDate(value) {
      return date.formatDate(value, "MM/DD/YY");
    }
  },
  created() {
    this.startCoinsApiCall();
    this.getCurrencys();
  },
  mounted() {
    window.addEventListener("scroll", this.scrollListener);
  },

  beforeDestroy: function() {
    window.removeEventListener("scroll", this.scrollListener);
  },
  name: "PageIndex"
};
</script>
