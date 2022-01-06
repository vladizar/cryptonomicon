<template>
  <div class="container">
    <section>
      <div class="flex">
        <div class="max-w-xs">
          <label for="wallet" class="block text-sm font-medium text-gray-700"
            >Тикер</label
          >
          <div class="mt-1 relative rounded-md shadow-md">
            <input
              v-model="tickerInput"
              @input="updateAutocomplete"
              @keydown.enter="addTicker"
              type="text"
              name="wallet"
              id="wallet"
              class="block w-full pr-10 border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm rounded-md"
              placeholder="Например DOGE"
            />
          </div>
          <div
            v-if="autocompleteSuggestions.length"
            class="flex bg-white shadow-md p-1 rounded-md flex-wrap"
          >
            <span
              v-for="suggestion in autocompleteSuggestions"
              :key="suggestion.symbol"
              @click="addSuggestedTicker(suggestion.symbol)"
              class="inline-flex items-center px-2 m-1 rounded-md text-xs font-medium bg-gray-300 text-gray-800 cursor-pointer"
            >
              {{ suggestion.symbol }}
            </span>
          </div>
          <div v-if="invalidInput" class="text-sm text-red-600">
            Такой тикер уже добавлен
          </div>
        </div>
      </div>
      <button
        @click="addTicker"
        type="button"
        class="my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
      >
        <!-- Heroicon name: solid/mail -->
        <svg
          class="-ml-0.5 mr-2 h-6 w-6"
          xmlns="http://www.w3.org/2000/svg"
          width="30"
          height="30"
          viewBox="0 0 24 24"
          fill="#ffffff"
        >
          <path
            d="M13 7h-2v4H7v2h4v4h2v-4h4v-2h-4V7zm-1-5C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 18c-4.41 0-8-3.59-8-8s3.59-8 8-8 8 3.59 8 8-3.59 8-8 8z"
          ></path>
        </svg>
        Добавить
      </button>
    </section>

    <template v-if="tickers.length">
      <hr class="w-full border-t border-gray-600 my-4" />
      <div>
        <button
          v-if="page > 1"
          @click="page = Math.max(1, page - 1)"
          class="my-4 mx-2 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
        >
          Назад
        </button>
        <button
          v-if="page < Math.ceil(getFilteredTickers().length / 6)"
          @click="
            page = Math.min(
              Math.ceil(getFilteredTickers().length / 6),
              page + 1
            )
          "
          class="my-4 mx-2 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
        >
          Вперед
        </button>
        <div>
          Фильтр:
          <input v-model="filterInput" />
        </div>
      </div>
      <hr class="w-full border-t border-gray-600 my-4" />
      <dl class="mt-5 grid grid-cols-1 gap-5 sm:grid-cols-3">
        <div
          v-for="ticker in getCurrentPageTickers(getFilteredTickers())"
          :key="ticker.name"
          :class="{
            'border-4': ticker.name === selectedTicker?.name,
          }"
          @click="selectedTicker = ticker"
          class="bg-white overflow-hidden shadow rounded-lg border-purple-800 border-solid cursor-pointer"
        >
          <div class="px-4 py-5 sm:p-6 text-center">
            <dt class="text-sm font-medium text-gray-500 truncate">
              {{ ticker.name }} - USD
            </dt>
            <dd class="mt-1 text-3xl font-semibold text-gray-900">
              {{ ticker.price }}
            </dd>
          </div>
          <div class="w-full border-t border-gray-200"></div>
          <button
            @click.stop="deleteTicker(ticker.name)"
            class="flex items-center justify-center font-medium w-full bg-gray-100 px-4 py-4 sm:px-6 text-md text-gray-500 hover:text-gray-600 hover:bg-gray-200 hover:opacity-20 transition-all focus:outline-none"
          >
            <svg
              class="h-5 w-5"
              xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 20 20"
              fill="#718096"
              aria-hidden="true"
            >
              <path
                fill-rule="evenodd"
                d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z"
                clip-rule="evenodd"
              ></path></svg
            >Удалить
          </button>
        </div>
      </dl>
      <hr class="w-full border-t border-gray-600 my-4" />
    </template>
    <section v-if="selectedTicker" class="relative">
      <h3 class="text-lg leading-6 font-medium text-gray-900 my-8">
        {{ selectedTicker?.name }} - USD
      </h3>
      <div class="flex items-end border-gray-600 border-b border-l h-64">
        <div
          v-for="(price, idx) in normalizePriceHistory(
            selectedTicker?.priceHistory
          )"
          :key="idx"
          :style="{
            height: `${price}%`,
          }"
          class="bg-purple-800 border w-10"
        ></div>
      </div>
      <button
        @click="selectedTicker = null"
        type="button"
        class="absolute top-0 right-0"
      >
        <svg
          xmlns="http://www.w3.org/2000/svg"
          xmlns:xlink="http://www.w3.org/1999/xlink"
          xmlns:svgjs="http://svgjs.com/svgjs"
          version="1.1"
          width="30"
          height="30"
          x="0"
          y="0"
          viewBox="0 0 511.76 511.76"
          style="enable-background: new 0 0 512 512"
          xml:space="preserve"
        >
          <g>
            <path
              d="M436.896,74.869c-99.84-99.819-262.208-99.819-362.048,0c-99.797,99.819-99.797,262.229,0,362.048    c49.92,49.899,115.477,74.837,181.035,74.837s131.093-24.939,181.013-74.837C536.715,337.099,536.715,174.688,436.896,74.869z     M361.461,331.317c8.341,8.341,8.341,21.824,0,30.165c-4.16,4.16-9.621,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    l-75.413-75.435l-75.392,75.413c-4.181,4.16-9.643,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    c-8.341-8.341-8.341-21.845,0-30.165l75.392-75.413l-75.413-75.413c-8.341-8.341-8.341-21.845,0-30.165    c8.32-8.341,21.824-8.341,30.165,0l75.413,75.413l75.413-75.413c8.341-8.341,21.824-8.341,30.165,0    c8.341,8.32,8.341,21.824,0,30.165l-75.413,75.413L361.461,331.317z"
              fill="#718096"
              data-original="#000000"
            ></path>
          </g>
        </svg>
      </button>
    </section>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      tickerInput: "",
      tickers: [],
      selectedTicker: null,
      invalidInput: false,
      coinList: [],
      autocompleteSuggestions: [],
      page: 1,
      filterInput: "",
      API: "631710474c42d3740b4ddcbdd31ad1227e802a440eca0ff242de182eb9d89c52",
    };
  },
  watch: {
    tickers: {
      deep: true,
      handler() {
        this.updateLocalStorage();
      },
    },
    filterInput() {
      this.selectedTicker = null;
      this.page = 1;

      this.updateURL();
    },
    page() {
      this.updateURL();
    },
  },
  mounted() {
    setInterval(this.fetchTickerPrices, 12000);
    this.fetchCoinList();

    const tickersData = localStorage.getItem("cryptonomicon-tickers");

    if (!tickersData) {
      return;
    }

    JSON.parse(tickersData).map((tickerName) => {
      this.tickers.push(this.getNewTicker(tickerName));
    });

    const windowData = Object.fromEntries(
      new URL(window.location).searchParams.entries()
    );

    if (windowData.filter) {
      this.filterInput = windowData.filter;
    }

    if (windowData.page) {
      this.page = windowData.page;
    }
  },
  methods: {
    updateURL() {
      window.history.pushState(
        null,
        document.title,
        `${window.location.pathname}?filter=${this.filterInput}&page=${this.page}`
      );
    },
    getTickerNames() {
      return this.tickers.map((t) => t.name);
    },
    getFilteredTickers() {
      return this.tickers.filter((t) =>
        t.name.includes(this.filterInput.toUpperCase())
      );
    },
    getCurrentPageTickers(tickersList) {
      const firstIndex = (this.page - 1) * 6;
      const lastIndex = this.page * 6;

      return tickersList.slice(firstIndex, lastIndex);
    },
    updateAutocomplete() {
      this.invalidInput = false;

      const tickerName = this.tickerInput.toUpperCase();

      if (!tickerName) {
        this.autocompleteSuggestions = [];
        return;
      }

      const suggestions = this.coinList.filter((t) =>
        t.fullName.includes(tickerName)
      );

      this.autocompleteSuggestions = suggestions.slice(0, 4);
    },
    updateLocalStorage() {
      localStorage.setItem(
        "cryptonomicon-tickers",
        JSON.stringify(this.getTickerNames())
      );
    },
    getNewTicker(tickerName) {
      return {
        name: tickerName,
        price: "-",
        priceHistory: [],
      };
    },
    addTicker() {
      const tickerName = this.tickerInput.toUpperCase();

      if (this.getTickerNames().includes(tickerName)) {
        this.invalidInput = true;
        return;
      }

      this.tickers.push(this.getNewTicker(tickerName));

      this.tickerInput = "";
      this.filterInput = "";

      this.updateAutocomplete();
    },
    addSuggestedTicker(tickerName) {
      this.tickerInput = tickerName;

      this.addTicker();
    },
    deleteTicker(tickerName) {
      this.tickers = this.tickers.filter((t) => t.name !== tickerName);

      if (tickerName === this.selectedTicker?.name) {
        this.selectedTicker = null;
      }

      this.page = Math.max(
        1,
        Math.min(Math.ceil(this.getFilteredTickers().length / 6), this.page)
      );
    },
    async fetchCoinList() {
      const f = await fetch(
        "https://min-api.cryptocompare.com/data/all/coinlist?summary=true"
      );

      const response = await f.json();
      const data = response.Data;

      for (const coin in data) {
        const coinProperties = data[coin];

        this.coinList.push({
          fullName: coinProperties.FullName.toUpperCase(),
          symbol: coinProperties.Symbol,
        });
      }
    },
    async fetchTickerPrices() {
      const tickerNames = this.getTickerNames();

      if (!tickerNames.length) {
        return;
      }

      const f = await fetch(
        `https://min-api.cryptocompare.com/data/pricemultifull?fsyms=${tickerNames.join()}&tsyms=USD&api_key=${
          this.API
        }`
      );

      const response = await f.json();
      const data = response.RAW;

      this.tickers.map((t) => {
        try {
          const tickerPrice = data[t.name].USD.PRICE;
          t.price =
            tickerPrice > 1
              ? tickerPrice.toFixed(2)
              : tickerPrice.toPrecision(2);
          t.priceHistory.push(tickerPrice);
        } catch (error) {
          t.price = "ERROR";
        }
      });
    },
    normalizePriceHistory(priceHistory) {
      if (!priceHistory) {
        return;
      }

      const maxPrice = Math.max(...priceHistory);
      const minPrice = Math.min(...priceHistory);

      return priceHistory.map(
        (price) =>
          ((price - minPrice) * 90) / (maxPrice - minPrice + 0.000000001) + 10
      );
    },
  },
};
</script>

<style></style>
