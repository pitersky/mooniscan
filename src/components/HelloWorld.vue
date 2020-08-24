<template>
  <div class="w-full text-sm">
    <form class="bg-darkest rounded-lg px-8 pt-6 pb-8 mb-4">
      <div class="grid sm:grid-cols-1 md:grid-cols-2 lg:grid-cols-2 xl:grid-cols-4 gap-4">

        <div class="mb-2">
          <label class="whitespace-no-wrap inline-block text-gray-700 text-sm font-bold mb-2 w-1/4 text-right pr-4" for="username">
            Volume >=
          </label>
          <input type="number" class="bg-darkest appearance-none border rounded-lg w-3/4 py-2 px-3 text-gray-300 leading-tight focus:outline-none focus:shadow-outline border-gray-700" id="username" placeholder="Volume" v-model="volumeUSD_gte">
        </div>

        <div class="mb-2">
          <label class="whitespace-no-wrap inline-block text-gray-700 text-sm font-bold mb-2 w-1/4 text-right pr-4" for="tx_count">
            Tx count >=
          </label>
          <input type="number" class="whitespace-no-wrap bg-darkest appearance-none border rounded-lg w-3/4 py-2 px-3 text-gray-300 leading-tight focus:outline-none focus:shadow-outline border-gray-700" id="tx_count" placeholder="Tx count" v-model="txCount_gte">
        </div>

        <div class="mb-2">
          <label class="whitespace-no-wrap inline-block text-gray-700 text-sm font-bold mb-2 w-1/4 text-right pr-4" for="reserve">
            Reserve >=
          </label>
          <input class="bg-darkest appearance-none border rounded-lg w-3/4 py-2 px-3 text-gray-300 leading-tight focus:outline-none focus:shadow-outline border-gray-700" id="reserve" type="number" placeholder="Reserve" v-model="reserveUSD_gte">
        </div>

        <div class="mb-2">
          <label class="inline-block text-gray-700 text-sm font-bold mb-2 w-1/4 text-right pr-4" for="days">
            Days &lt;
          </label>
          <input class="bg-darkest appearance-none border rounded-lg w-3/4 py-2 px-3 text-gray-300 leading-tight focus:outline-none focus:shadow-outline border-gray-700" id="days" type="number" placeholder="Days" v-model="days">
        </div>

      </div>
      
      <div class="mt-8 flex items-center justify-end">
        <button @click="search" class="w-full bg-blue-900 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-lg focus:outline-none focus:shadow-outline" type="button">
          Search
        </button>
      </div>
    </form>

    <div class="bg-darkest rounded-lg px-8 pt-6 pb-8 mb-4">
      <table class="table-auto w-full">
        <thead>
          <tr class="whitespace-no-wrap">
            <th class="px-4 py-2">Pair</th>
            <th @click="handleClick('reserveUSD')"  class="px-4 py-2 cursor-pointer hover:text-gray-100">Liquidity ($)</th>
            <th @click="handleClick('volumeUSD')"   class="px-4 py-2 cursor-pointer hover:text-gray-100">Volume ($)</th>
            <th @click="handleClick('volToLiq')"    class="px-4 py-2 cursor-pointer hover:text-gray-100">Vol / Liq</th>
            <th @click="handleClick('txCount')"     class="px-4 py-2 cursor-pointer hover:text-gray-100">Tx count</th>
            <th @click="handleClick('days')"        class="px-4 py-2 cursor-pointer hover:text-gray-100">Days active</th>
          </tr>
        </thead>
        <tbody>
          <tr class="border-0 hover:text-gray-100" v-for="pair of sortedPairs" :key="pair.id">
            <!-- <td class="px-4 py-2">
              <a class="hover:underline" :href="`https://mooniswap.info/token/${pair.token0.id}`" target="_blank"> {{ pair.token0.symbol }} </a> -
              <a class="hover:underline" :href="`https://mooniswap.info/token/${pair.token1.id}`" target="_blank"> {{ pair.token1.symbol }} </a>
            </td> -->
            <td class="px-4 py-2">
              <a class="hover:underline" :href="`https://mooniswap.info/pair/${pair.id}`" target="_blank"> {{ pair.name }}</a>
            </td>
            <td class="px-4 py-2">{{ formatCurrency(pair.reserveUSD) }}</td>
            <td class="px-4 py-2">{{ formatCurrency(pair.volumeUSD) }}</td>
            <td class="px-4 py-2">{{ formatCurrency(pair.volToLiq) }}</td>
            <td class="px-4 py-2">{{ pair.txCount }}</td>
            <td class="px-4 py-2">{{ pair.days }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    <div>
      <a href="https://github.com/Mooniscan/mooniscan">Github</a> | <a href="https://twitter.com/OgurchickYura">Twitter</a>
    </div>
  </div>
</template>

<script>
const api = "https://api.thegraph.com/subgraphs/name/krboktv/try-second-graph"

export default {
  name: 'HelloWorld',
  data () {
    return {
      volumeUSD_gte: 10000,
      txCount_gte: 100,
      reserveUSD_gte: 10000,
      days: 3,
      pairs: [],
      sortColumn: 'days',
      sortDir: 'asc'
    }
  },
  mounted() {
    this.search()
  },
  computed: {
    createdAtTimestamp () {
      const queryAt = new Date()
      const date = queryAt.getDate()
      queryAt.setDate(date - this.days)

      return Math.floor(queryAt.getTime() / 1000)
    },
    sortedPairs() {
      const sortedPairs = this.pairs.slice()

      return sortedPairs.sort((a, b) => {
        if (this.sortDir === 'asc') {
          if (Number(a[this.sortColumn]) < Number(b[this.sortColumn])) {
            return -1
          }
          if (Number(a[this.sortColumn]) > Number(b[this.sortColumn])) {
            return 1
          }
          return 0
        } else {
          if (Number(b[this.sortColumn]) < Number(a[this.sortColumn])) {
            return -1
          }
          if (Number(b[this.sortColumn]) > Number(a[this.sortColumn])) {
            return 1
          }
          return 0
        }
      })
    }
  },
  methods: {
    search () {
      const headers = new Headers()
      headers.append("Content-Type", "application/json")

      const query = `{
        pairs(
          first: 15 
          where: {
            createdAtTimestamp_gte: ${this.createdAtTimestamp} 
            volumeUSD_gte: ${this.volumeUSD_gte} 
            txCount_gte: ${this.txCount_gte} 
            reserveUSD_gte: ${this.reserveUSD_gte} 
          } )
        {
          id 
          token0 { id name symbol tradeVolumeUSD } 
          token1 { id name symbol tradeVolumeUSD } 
          volumeUSD 
          txCount 
          reserveUSD 
          createdAtTimestamp 
        } }`

      fetch(api, {
        method: 'POST',
        headers,
        body: JSON.stringify({query}),
      })
        .then(response => response.json())
        .then(result => this.pairs = this.calculateFields(result.data.pairs))
        .catch(error => console.error(error))
    },
    getDays (timestamp) {
      const now = new Date()
      return ((now.getTime() - timestamp * 1000) / 24 / 60 / 60 / 1000).toFixed(1)
    },
    handleClick (name) {
      if (this.sortColumn != name) {
        this.sortColumn = name
      } else {
        this.sortDir = this.sortDir === 'asc' ? 'desc' : 'asc'
      }
    },
    calculateFields (pairs) {
      return pairs.map((pair) => {
        return {
          ...pair,
          name: `${pair.token0.symbol} - ${pair.token1.symbol}`,
          reserveUSD: Number(pair.reserveUSD).toFixed(),
          volumeUSD: Number(pair.volumeUSD).toFixed(),
          volToLiq: Number(pair.volumeUSD / pair.reserveUSD).toFixed(2),
          days: this.getDays(pair.createdAtTimestamp),
        }
      })
    },
    formatCurrency (number) {
      return new Intl.NumberFormat('en-US', { maximumSignificantDigits: 3 }).format(number)
    }
  }
}
</script>

<style scoped>
.bg-darkest {
  background: #212429
}

 /* Remove ugly arrows for input=number */
 /* Chrome, Safari, Edge, Opera */
input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

/* Firefox */
input[type=number] {
  -moz-appearance: textfield;
}
</style>
