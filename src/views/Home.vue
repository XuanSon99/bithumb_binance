<template>
  <main>
    <v-container>
      <v-row>
        <v-col cols="12" md="2">
          <v-text-field v-model="krw_rate" label="Rate (KRW/VNĐ)" outlined dense clearable></v-text-field>
        </v-col>
        <v-col cols="12" md="2">
          <v-text-field v-model="usdt_rate" label="Rate (USDT/VNĐ)" outlined dense clearable></v-text-field>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12">
          <v-data-table :headers="headers" :items="bithumb_list" class="elevation-1" :mobile-breakpoint="0"
            :search="search" fixed-header>
            <template v-slot:[`item.price`]="{ item }">
              {{ formatPrice(item.price) }}
            </template>
            <template v-slot:[`item.upbit`]="{ item }">
              {{ formatPrice(upbit_price(item)) }}
            </template>
            <template v-slot:[`item.usdt_price`]="{ item }">
              {{ formatPrice(usdt_price(item)) }}
            </template>
            <template v-slot:[`item.vnd_price`]="{ item }">
              {{ formatPrice(vnd_price(item)) }}
            </template>
            <template v-slot:[`item.percent`]="{ item }">
              <span class="green-cl" v-if="item.percent > 0">{{ formatPrice(item.percent) }}</span>
              <span class="red-cl" v-if="item.percent < 0">{{ formatPrice(item.percent) }}</span>
            </template>
            <template v-slot:[`item.upbit_percent`]="{ item }">
              <span class="green-cl" v-if="item.upbit_percent > 0">{{ formatPrice(item.upbit_percent) }}</span>
              <span class="red-cl" v-if="item.upbit_percent < 0">{{ formatPrice(item.upbit_percent) }}</span>
            </template>
          </v-data-table>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="2">
          <v-text-field v-model="search" label="Tìm kiếm" outlined dense clearable></v-text-field>
        </v-col>
        <v-col cols="12" md="2">
          <v-btn @click="logout()" block>
            Đăng xuất
          </v-btn>
        </v-col>
      </v-row>
    </v-container>
  </main>
</template>

<script>
import axios from 'axios'
export default {
  data() {
    return {
      search: '',
      krw_rate: 18.2,
      usdt_rate: 25000,
      headers: [
        { text: 'Token', value: 'token', sortable: false, },
        { text: 'KRW-BT', value: 'price', sortable: false, },
        { text: 'KRW-UB', value: 'upbit', sortable: false, },
        { text: 'USDT', value: 'usdt_price', sortable: false, },
        { text: '% BT', value: 'percent' },
        { text: '% UB', value: 'upbit_percent' },
        { text: 'VND', value: 'vnd_price', sortable: false, },
      ],
      bithumb_list: [],
      binance_list: [],
      upbit_list: [],
      interval: '',
    }
  },
  mounted() {
    this.getBinance()
    this.getUpbit()
    this.interval = setInterval(() => {
      this.getBinance()
      this.getUpbit()
    }, 2000);
  },
  methods: {
    logout() {
      localStorage.removeItem('loged')
      this.$router.push("/login")
    },
    getUpbit() {
      let symbols = ['KRW-BTC', 'KRW-ETH', 'KRW-ETC', 'KRW-XRP', 'KRW-BCH', 'KRW-QTUM', 'KRW-BTG', 'KRW-EOS', 'KRW-TRX', "KRW-ICX", "KRW-ELF", "KRW-KNC", "KRW-GLM", "KRW-ZIL", "KRW-WAXP", "KRW-POWR", "KRW-STEEM", "KRW-STRAX", "KRW-ZRX", "KRW-SNT", "KRW-ADA", "KRW-BAT", "KRW-THETA", "KRW-LOOM", "KRW-WAVES", "KRW-LINK", "KRW-VET", "KRW-MTL", "KRW-IOST", "KRW-QKC", "KRW-TFUEL", "KRW-ANKR", "KRW-MBL", "KRW-SXP", "KRW-HIVE", "KRW-AERGO", "KRW-AAVE", "KRW-SAND", "KRW-STPT", "KRW-GRT", "KRW-PLA", "KRW-MANA", "KRW-PUNDIX", "KRW-CHZ", "KRW-AXS", "KRW-ONT", "KRW-SHIB", "KRW-ONG", "KRW-MATIC", "KRW-ALGO", "KRW-XLM", "KRW-XEC", "KRW-SOL", "KRW-XTZ", "KRW-JST", "KRW-DOT", "KRW-ATOM", "KRW-EGLD", "KRW-MASK", "KRW-1INCH", "KRW-FLOW", "KRW-AVAX", "KRW-GMT", "KRW-BTT", "KRW-T", "KRW-DOGE", "KRW-APT", "KRW-STX", "KRW-SEI", "KRW-ARB", "KRW-CELO", "KRW-IMX", "KRW-SUI"]
      axios.get('https://api.upbit.com/v1/ticker', {
        params: {
          markets: symbols.join(",")
        }
      }).then((res) => {
        this.upbit_list = []
        res.data.forEach(item => {
          this.upbit_list.push({
            token: item.market.split("-")[1],
            price: item.trade_price
          })
        });
      })
    },
    getBithumb() {
      axios.get('https://api.bithumb.com/public/ticker/ALL_KRW').then((res) => {
        let list = []
        for (let i in res.data.data) {
          list.push({
            token: i,
            price: res.data.data[i].closing_price
          })
        }
        list.unshift(list.splice(list.findIndex(i => i.token == "USDT"), 1)[0]);
        this.bithumb_list = []
        list.forEach(item => {
          if (this.percent(item) != 0) {
            this.bithumb_list.push({
              token: item.token,
              price: item.price,
              percent: this.percent(item),
              upbit_percent: this.upbit_percent(item)
            })
          }
        });
      }).catch((err) => {
        location.reload();
      })
    },
    getBinance() {
      let symbols = ["BTCUSDT", "ETHUSDT", "ETCUSDT", "XRPUSDT", "BCHUSDT", "QTUMUSDT", "BTGUSDT", "EOSUSDT", "ICXUSDT", "TRXUSDT", "ELFUSDT", "KNCUSDT", "GLMUSDT", "ZILUSDT", "WAXPUSDT", "POWRUSDT", "LRCUSDT", "STEEMUSDT", "STRAXUSDT", "ZRXUSDT", "SNTUSDT", "ADAUSDT", "CTXCUSDT", "BATUSDT", "THETAUSDT", "LOOMUSDT", "WAVESUSDT", "LINKUSDT", "ENJUSDT", "VETUSDT", "MTLUSDT", "IOSTUSDT", "QKCUSDT", "TFUELUSDT", "ANKRUSDT", "CHRUSDT", "MBLUSDT", "SXPUSDT", "COSUSDT", "HIVEUSDT", "ARPAUSDT", "CKBUSDT", "AERGOUSDT", "YFIUSDT", "UNIUSDT", "UMAUSDT", "COMPUSDT", "RSRUSDT", "AAVEUSDT", "RLCUSDT", "NMRUSDT", "BALUSDT", "RENUSDT", "SANDUSDT", "BELUSDT", "STPTUSDT", "GRTUSDT", "OXTUSDT", "LINAUSDT", "SNXUSDT", "CTSIUSDT", "PLAUSDT", "MANAUSDT", "LPTUSDT", "MKRUSDT", "CELRUSDT", "PUNDIXUSDT", "FRONTUSDT", "SUSHIUSDT", "OCEANUSDT", "COTIUSDT", "BNTUSDT", "CAKEUSDT", "XVSUSDT", "OGNUSDT", "ALICEUSDT", "CHZUSDT", "AXSUSDT", "DAIUSDT", "WOOUSDT", "ONTUSDT", "SHIBUSDT", "ONGUSDT", "MATICUSDT", "ALGOUSDT", "KLAYUSDT", "BNBUSDT", "SUNUSDT", "XLMUSDT", "XECUSDT", "AGIXUSDT", "SOLUSDT", "XTZUSDT", "JSTUSDT", "DOTUSDT", "ATOMUSDT", "CTKUSDT", "REIUSDT", "EGLDUSDT", "MASKUSDT", "CRVUSDT", "1INCHUSDT", "C98USDT", "DYDXUSDT", "FLOWUSDT", "BTTUSDT", "JASMYUSDT", "GALAUSDT", "REQUSDT", "AVAXUSDT", "GMTUSDT", "DARUSDT", "ACHUSDT", "SFPUSDT", "KSMUSDT", "TUSDT", "DOGEUSDT", "APTUSDT", "HOOKUSDT", "APEUSDT", "OPUSDT", "GMXUSDT", "STXUSDT", "CFXUSDT", "HFTUSDT", "ILVUSDT", "LEVERUSDT", "WLDUSDT", "SEIUSDT", "FXSUSDT", "IMXUSDT", "ARBUSDT", "RPLUSDT", "INJUSDT", "CELOUSDT", "FTMUSDT", "LDOUSDT", "FLOKIUSDT", "FETUSDT", "RNDRUSDT", "STGUSDT", "SUIUSDT", "MAVUSDT", "PEPEUSDT"]

      axios.get('https://api3.binance.com/api/v3/ticker/price', {
        params: {
          symbols: JSON.stringify(symbols)
        }
      }).then((res) => {
        this.binance_list = [{
          token: "USDTUSDT",
          price: 1,
          vnd: this.usdt_rate
        }]
        res.data.forEach(i => {
          this.binance_list.push({
            token: i.symbol,
            price: i.price,
            vnd: i.price * this.usdt_rate
          })
        })
        this.getBithumb()
      }).catch((err) => {
        location.reload();
      })
    },
    percent(item) {
      let token = this.binance_list.filter((i) => i.token == `${item.token}USDT`)[0]
      return token ? ((item.price * this.krw_rate - token.vnd) / token.vnd) * 100 : 0
    },
    usdt_price(item) {
      let token = this.binance_list.filter((i) => i.token == `${item.token}USDT`)[0]
      return token.price
    },
    vnd_price(item) {
      let token = this.binance_list.filter((i) => i.token == `${item.token}USDT`)[0]
      return token.vnd
    },
    upbit_price(item){
      let token = this.upbit_list.filter((i) => i.token == item.token)[0]
      return token ? token.price : 0
    },
    upbit_percent(item) {
      let binance = this.binance_list.filter((i) => i.token == `${item.token}USDT`)[0]
      let upbit = this.upbit_list.filter((i) => i.token == item.token)[0]
      return upbit && binance ? ((upbit.price * this.krw_rate - binance.vnd) / binance.vnd) * 100 : 0
    },
    formatPrice(value) {
      if (!value) return 0;
      if (String(value).slice(0, 6) == '0.0000') return parseFloat(value).toFixed(6);
      if (String(value).slice(0, 2) == '0.') return parseFloat(value).toFixed(4);
      return String(parseFloat(value).toFixed(2))
        .toString()
        .replace(/\B(?=(\d{3})+(?!\d))/g, ",")
        .replace('.00', '')
    },
  },
  beforeDestroy() {
    clearInterval(this.interval);
  },
};
</script>