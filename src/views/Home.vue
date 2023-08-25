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
        <v-col cols="12" md="6">
          <v-data-table :headers="headers" :items="bithumb_list" class="elevation-1" :mobile-breakpoint="0">
            <template v-slot:[`item.price`]="{ item }">
              {{ formatMoney(item.price) }}
            </template>
            <template v-slot:[`item.percent`]="{ item }">
              <span class="green-cl" v-if="item.percent > 0">{{ formatPrice(item.percent) }}</span>
              <span class="red-cl" v-if="item.percent < 0">{{ formatPrice(item.percent) }}</span>
            </template>
          </v-data-table>
        </v-col>
        <v-col cols="12" md="6">
          <v-data-table :headers="headers2" :items="binance_list" class="elevation-1" :mobile-breakpoint="0">
            <template v-slot:[`item.price`]="{ item }">
              {{ formatPrice(item.price) }}
            </template>
            <template v-slot:[`item.vnd`]="{ item }">
              {{ formatMoney(item.vnd) }}
            </template>
          </v-data-table>
        </v-col>
      </v-row>
      <v-btn class="mt-5" @click="logout()">
        Đăng xuất
      </v-btn>
    </v-container>
  </main>
</template>

<script>
import axios from 'axios'
export default {
  data() {
    return {
      krw_rate: 20.669,
      usdt_rate: 23680,
      headers: [
        { text: 'Token', value: 'token' },
        { text: 'Price', value: 'price' },
        { text: '%', value: 'percent' },
      ],
      headers2: [
        { text: 'Token', value: 'token' },
        { text: 'Price', value: 'price' },
        { text: 'VND', value: 'vnd' },
      ],
      bithumb_list: [],
      binance_list: [],
      interval: '',
    }
  },
  mounted() {
    this.getBinance()
    this.interval = setInterval(() => {
      this.getBinance()
    }, 2000);
  },
  methods: {
    logout(){
      localStorage.removeItem('loged')
      this.$router.push("/login")
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
        this.bithumb_list = []
        list.forEach(item => {
          if (this.percent(item) != 0) {
            this.bithumb_list.push({
              token: item.token,
              price: item.price,
              percent: this.percent(item)
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
        this.binance_list = []
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
    formatMoney(value) {
      if (!value) return 0;
      return String(parseFloat(value).toFixed(0))
        .toString()
        .replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    },
    formatPrice(value) {
      let val = (value / 1).toFixed(2).replace(".", ",");
      return val.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".");
    },
  },
  beforeDestroy() {
    clearInterval(this.interval);
  },
};
</script>