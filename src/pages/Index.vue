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
          type="text"
          v-model="amount"
          placeholder="Amount"
          bg-color="accent"
          lazy-rules
          :rules="[
            val => (val !== null && val !== '') || 'Please type something'
          ]"
        />
        <q-input
          filled
          type="text"
          v-model="price"
          placeholder="Buy Price"
          label-color="white"
          bg-color="accent"
          lazy-rules
          :rules="[
            val => (val !== null && val !== '') || 'Please type something'
          ]"
        />
        <div style="display: flex;flex-direction: column-reverse;">
          <q-btn
            label="add"
            type="submit"
            class="right"
            icon="add_box"
            style="background-color:#e84545;color:#ececec"
          />
        </div>
      </q-form>
    </transition-group>
    <!-- ITEM  HEADER -->
    <q-list v-for="s in someCurrencies" :key="s.id">
      <div v-for="coin in coins" :key="coin.id">
        <q-expansion-item
          v-if="coin.name.toLowerCase() == s.name.toLowerCase()"
          default-opened
          header-class="bg-secondary text-white rounded-borders q-mb-md"
        >
          <template v-slot:header>
            <q-item-section>
              {{ s.name }}
            </q-item-section>
            <q-item-section class="absolute-right q-mr-xl">
              <q-item-label class="text-h6">
                USD {{ coin.current_price }}
              </q-item-label>
            </q-item-section>
          </template>
          <q-list v-for="c in currencies" :key="c.id" separator class="q-ma-md">
            <!-- ITEM  -->
            <q-item
              style="background-color:#5d7290"
              class="rounded-borders"
              v-if="c.name == s.name"
              :class="c.name"
            >
              <div v-for="coin in coins" :key="coin.id">
                <!-- DELETE BUTTON -->
                <q-item-section
                  class="absolute-left"
                  v-if="coin.name.toLowerCase() == c.name.toLowerCase()"
                >
                  <q-item-label>
                    <q-btn
                      size="6px"
                      round
                      icon="clear"
                      color="dark"
                      style="top:-7px;left:-7px;position:absolute"
                      v-if="deleteVisibility"
                      @click.stop="deleteCurrency(c.id)"
                    />
                  </q-item-label>
                </q-item-section>
                <!-- COIN DATE  -->
                <q-item-section
                  class="absolute-left"
                  v-if="coin.name.toLowerCase() == c.name.toLowerCase()"
                >
                  <q-item-label
                    class="q-ml-md"
                    overline
                    style="color:#f2edd7;font-size:medium"
                  >
                    {{ c.id | niceDate }}
                  </q-item-label>
                </q-item-section>
                <!-- COIN AMOUNT  -->
                <q-item-section
                  class="absolute-center"
                  side
                  v-if="coin.name.toLowerCase() == c.name.toLowerCase()"
                >
                  <q-badge
                    rounded
                    color="primary"
                    class="q-pa-xs"
                    style="font-size:small"
                    >{{ c.amount }}</q-badge
                  >
                </q-item-section>
                <!-- COIN PRICE -->
                <q-item-section
                  class="absolute-right q-mr-md"
                  v-if="coin.name.toLowerCase() == c.name.toLowerCase()"
                >
                  <div v-if="c.amount * (coin.current_price - c.price) > 0">
                    <q-item-label
                      overline
                      style="color:#9ede73;font-size:medium"
                    >
                      ${{
                        (c.amount * (coin.current_price - c.price)).toFixed(2)
                      }}
                    </q-item-label>
                  </div>
                  <div v-else-if="c.amount * (coin.current_price - c.price) < 0">
                    <q-item-label
                      overline
                      style="color:#ff7171;font-size:large"
                    >
                      ${{
                        (c.amount * (coin.current_price - c.price)).toFixed(2)
                      }}
                    </q-item-label>
                  </div>
                  <div v-else>
                    <q-item-label
                      overline
                      style="color:#bbbbbb;font-size:large"
                    >
                      ${{
                        (c.amount * (coin.current_price - c.price)).toFixed(2)
                      }}
                    </q-item-label>
                  </div>
                </q-item-section>
              </div>
            </q-item>
          </q-list>
        </q-expansion-item>
      </div>
    </q-list>

    <!-- FUNCTION BUTTON -->
    <q-page-sticky position="bottom-right" :offset="fabPos">
      <q-fab
        icon="add"
        direction="up"
        color="primary"
        :disable="draggingFab"
        v-touch-pan.prevent.mouse="moveFab"
      >
        <q-fab-action
          color="negative"
          icon="remove"
          :disable="draggingFab"
          v-on:click="deleteVisibility = !deleteVisibility"
        />
        <q-fab-action
          @click="actionAddButton()"
          color="positive"
          icon="add"
          :disable="draggingFab"
        />
      </q-fab>
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
          type="text"
          v-model="amount"
          placeholder="Amount"
          bg-color="accent"
          lazy-rules
          :rules="[
            val => (val !== null && val !== '') || 'Please type something'
          ]"
        />
        <q-input
          filled
          type="text"
          v-model="price"
          placeholder="Buy Price"
          label-color="white"
          bg-color="accent"
          lazy-rules
          :rules="[
            val => (val !== null && val !== '') || 'Please type something'
          ]"
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
    <div v-for="c in someCurrencies" :key="c.id">
      <q-card-section>
        <q-skeleton type="QSlider" />
      </q-card-section>
    </div>
    <!-- FUNCTION BUTTON -->
    <q-page-sticky position="bottom-right" :offset="fabPos">
      <q-fab
        icon="add"
        direction="up"
        color="primary"
        :disable="draggingFab"
        v-touch-pan.prevent.mouse="moveFab"
      >
        <q-fab-action
          color="negative"
          icon="remove"
          :disable="draggingFab"
          v-on:click="deleteVisibility = !deleteVisibility"
        />
        <q-fab-action
          @click="actionAddButton()"
          color="positive"
          icon="add"
          :disable="draggingFab"
        />
      </q-fab>
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
      visible: false
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
      this.currencies.push(newCurrency);
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
    //api call every 10 seconds
    startInterval: function() {
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
    // END METHODS //
  },
  //date filter
  filters: {
    niceDate(value) {
      return date.formatDate(value, "MM/DD/YY");
    }
  },
  created() {
    this.startInterval();
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
