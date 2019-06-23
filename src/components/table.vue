<template>
  <div class="table">
    <div class="head table_wrapper">
      <div class="headTable_item table_item">
        <table>
          <tr>
            <td class="name">Название</td>
            <td class="numbers">Стоимость</td>
            <td class="numbers desktopOnly">Рыночная капитализация</td>
            <td class="numbers desktopOnly">Суточный объем</td>
          </tr>
        </table>
      </div>
    </div>
    <div class="bodyTable table_wrapper">
      <div class="bodyTable_item table_item">
        <table class="main_table">
          <tr v-for="item in info" v-bind:key="item.id">
            <td class="name">{{item.name}}</td>
            <td class="numbers">{{item.priceUsd| currencydecimal}}</td>
            <td class="numbers desktopOnly">{{item.marketCapUsd| currencydecimal }}</td>
            <td class="numbers desktopOnly">{{item.volumeUsd24Hr| currencydecimal }}</td>
          </tr>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "table",
  data() {
    return {
      info: null
    };
  },
  mounted() {
    this.fetchCoins("https://api.coincap.io/v2/assets");
  },
  filters: {
    currencydecimal(value) {
      return parseFloat(value, 10).toFixed(3);
    }
  },
  methods: {
    infoSetup(value) {
      this.info = value;
    },
    socketOn(names) {
      const bindSetValue = (index, newValue) => {
        this.info[index].priceUsd = newValue;
      };
      const coinArray = this.info;
      const pricesWs = new WebSocket(
        "wss://ws.coincap.io/prices?assets=" + names
      );
      pricesWs.onmessage = function(msg) {
        for (let coin in coinArray) {
          if (msg.data.match(coinArray[coin].name.toLowerCase())) {
            bindSetValue(
              coin,
              JSON.parse(msg.data)[coinArray[coin].name.toLowerCase()]
            );
          }
        }
      };
    },
    async fetchCoins(source) {
      const res = await fetch(source, {
        headers: {
          "Content-Type": "application/json",
          Accept: "application/json"
        }
      });
      const resJson = await res.json();
      const val = await resJson.data
        .sort(function(a, b) {
          return b.marketCapUsd - a.marketCapUsd;
        })
        .slice(0, 15);
      await this.infoSetup(val);
      const names = await val
        .map(function(a) {
          return a.name.toLowerCase();
        })
        .join(",");
      await this.socketOn(names);
    }
  }
};
</script>


<style lang="scss">
.head {
  margin-top: 120px;
}
table {
  width: 1050px;
  // border-collapse: collapse;
}
.table_wrapper {
  background: #f8f4ff;
  display: flex;
  justify-content: center;
}
.table_item {
  width: 1080px;
}
.bodyTable_item {
  height: 212px;
  overflow: hidden;
  td:nth-child(odd) {
    background-color: rgba(255, 235, 205, 0.6);
  }
  td:nth-child(even) {
    background-color: rgba(190, 128, 204, 0.2);
  }
  &:hover {
    overflow-y: scroll;
  }
  &:hover td:nth-child(odd) {
    background-color: rgba(255, 235, 205, 1);
  }
  &:hover td:nth-child(even) {
    background-color: rgba(190, 128, 204, 0.4);
  }
}

tr {
  margin: 0;
  height: 40px;
}
td {
  padding: 10px 5px;
  border-radius: 3px;
}

.name {
  width: 36%;
}
.numbers {
  width: 22%;
}

.main_table {
  counter-reset: schetchik;
  tr {
    counter-increment: schetchik;
    &:before {
      content: counter(schetchik);
      display: table-cell;
      vertical-align: middle;
      color: #978777;
    }
  }
}
@media screen and (min-width: 1450px) {
  table {
    width: 1380px;
  }

  .table_item {
    width: 1410px;
  }
}

@media screen and (max-width: 1080px) {
  table {
    width: 880px;
  }
  .table_item {
    width: 990px;
  }
}
@media screen and (max-width: 880px) {
  table {
    width: 100%;
  }
}

@media screen and (max-width: 660px) {
  .name {
    width: auto;
  }
  .desktopOnly {
    display: none;
  }

  .main_table {
    tr {
      position: relative;
      &:before {
        content: counter(schetchik);
        color: #978777;
      }
    }
  }
}
</style>
