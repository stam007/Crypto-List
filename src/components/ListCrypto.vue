<template>
  <div>
    <nav class="navbar navbar-light bg-light shadow text-uppercase">
      <a class="navbar-brand" href="#"> cryptocurrency list </a>
    </nav>

    <div class="container" style="margin-top: 70px">
      <table class="table text-uppercase text-dark">
        <thead>
          <tr>
            <th scope="col">Name</th>
            <th scope="col">Cost of a single transaction (USD)</th>
            <th scope="col">Cost of a multisig transaction (USD)</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(y, x) in List_Result" v-bind:key="x">
            <th scope="row">{{ y.name + "-" + "(" + y.symbol + ")" }}</th>
            <td>{{ y.single }}</td>
            <td>{{ y.multisig }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "ListCrypto",
  props: {
    msg: String,
  },
  data() {
    return {
      Bitcoin_Take: 0,
      Bitcoin_SV_Take: 0,
      Ethereum_Take: 0,
      Binance_Smart_Chain_Take: 0,
      Current_Currency_Rates: {},
      List_Result: [],
      list_symbol: ["BTC", "ETH", "BSC", "BSV"],
      action1: false,
      action2: false,
      action3: false,
      action4: false,
      action5: false,
      interval: null,
    };
  },

  methods: {
    getAllData: function () {
      axios
        .get("https://api.blockchain.info/mempool/fees")
        .then((response) => {
          this.Bitcoin_Take = response.data["regular"];
          console.log("Bitcoin_Take", this.Bitcoin_Take);
          this.action1 = true;
        })
        .catch((error) => {
          console.log(error);
        });

      //-------------------------1------------//
      axios
        .get("https://mapi.taal.com/mapi/feeQuote")
        .then((response) => {
          this.Bitcoin_SV_Take = JSON.parse(response.data["payload"])[
            "fees"
          ][0]["relayFee"]["satoshis"];
          console.log("Bitcoin_SV_Take", this.Bitcoin_SV_Take);
          this.action2 = true;
        })
        .catch((error) => {
          console.log(error);
        });

      //-------------------------2------------//
      axios
        .get(
          "https://api.etherscan.io/api?module=gastracker&action=gasoracle&apikey"
        )
        .then((response) => {
          this.Ethereum_Take = response.data["result"]["FastGasPrice"];
          console.log("Ethereum_Take", this.Ethereum_Take);
          this.action3 = true;
        })
        .catch((error) => {
          console.log(error);
        });

      //-------------------------3------------//
      axios
        .get(
          "https://api.bscscan.com/api?module=gastracker&action=gasoracle&apikey="
        )
        .then((response) => {
          this.Binance_Smart_Chain_Take =
            response.data["result"]["FastGasPrice"];

          this.Current_Currency_Rates["BSC"] =
            response.data["result"]["UsdPrice"];
          console.log(
            "Binance_Smart_Chain_Take",
            this.Binance_Smart_Chain_Take
          );
          this.action4 = true;
        })
        .catch((error) => {
          console.log(error);
        });

      //-------------------------4------------//
      axios
        .get("https://api.coincap.io/v2/assets")
        .then((response) => {
          for (let x of this.list_symbol) {
            var rate = 0;
            try {
              rate = response.data["data"].filter((kk) => kk.symbol == x)[0][
                "priceUsd"
              ];

              this.Current_Currency_Rates[x] = rate;
            } catch (error) {
              console.log(x);
            }
          }

          // this.Current_Currency_Rates = response.data;
          console.log("Current_Currency_Rates", this.Current_Currency_Rates);
          this.action5 = true;
        })
        .catch((error) => {
          console.log(error);
        });

      this.interval = setInterval(() => {
        var cond =
          this.action1 &&
          this.action2 &&
          this.action3 &&
          this.action4 &&
          this.action5;

        console.log(cond);

        if (cond == true) {
          var items = [
            { name: "Bitcoin", single: 0, multisig: "A", symbol: "BTC" },
            { name: "Ethereum", single: 0, multisig: "A", symbol: "ETH" },
            {
              name: "Binance Smart Chain",
              single: 0,
              multisig: "A",
              symbol: "BSC",
            },
            { name: "Bitcoin SV", single: 0, multisig: "N", symbol: "BSV" },
          ];

          items[0]["single"] =
            1e-8 * 192 * this.Current_Currency_Rates["BTC"] * this.Bitcoin_Take;
          items[2]["single"] =
            1e-8 *
            192 *
            this.Current_Currency_Rates["BSC"] *
            this.Binance_Smart_Chain_Take;

          items[1]["single"] =
            1e-9 *
            21000 *
            this.Current_Currency_Rates["ETH"] *
            this.Ethereum_Take;

          items[3]["single"] =
            1e-9 *
            21000 *
            this.Current_Currency_Rates["BSV"] *
            this.Bitcoin_SV_Take;

          this.List_Result = items;

          this.action1 = false;
          this.action2 = false;
          this.action3 = false;
          this.action4 = false;
          this.action5 = false;

          clearInterval(this.interval);
        }
      }, 2000);
    },
  },
  mounted() {
    this.getAllData();
    setInterval(() => {
      this.getAllData();
    }, 600000);
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
