<template>
  <div v-if="modalVisiable" id="apart-info-modal" class="card row">
    <div class="title-bar">
      <h5 class="card-title">INFO</h5>
    </div>
    <div class="apart-title">
      {{ nowHouseName }}
    </div>
    <div class="info-body">
      <div id="bus-info-list">
        <div class="apart-info-list-title"><span>πκ°κΉμ΄ μ λ₯μ₯μ?</span></div>
        <div class="bus-top3-list">
          <ul class="list-group">
            <li
              class="list-group-item list-group-item-light"
              v-for="(item, index) in busesTop3"
              :key="index"
            >
              <div class="info-apartmentName">
                <h5>{{ item.bus_name }}</h5>
              </div>
              <div class="info-address">κ±°λ¦¬ {{ busDistance[index] }}m</div>
            </li>
          </ul>
        </div>
      </div>
      <hr />
      <div id="school-info-list">
        <div class="apart-info-list-title"><span>π«κ°κΉμ΄ νκ΅λ?</span></div>
        <div class="bus-top3-list">
          <ul class="list-group">
            <li
              class="list-group-item list-group-item-light"
              v-for="(item, index) in schoolsTop3"
              :key="index"
            >
              <div class="info-apartmentName">
                <h5>{{ item.schoolName }}</h5>
              </div>
              <div class="info-address">κ±°λ¦¬ {{ schoolDistance[index] }}m</div>
            </li>
          </ul>
        </div>
      </div>
      <hr />
      <div id="weather-info-list">
        <div class="apart-info-list-title"><span>βμ€μκ° λλ€ λ μ¨</span></div>
        <div class="weather-wrapper">
          <div class="weather-title">
            {{ this.fiexedWeather.weather[0].main }}
          </div>
          <div class="weather-contents">
            <div class="weather-text">
              νμ¬ μ¨λ: {{ this.fiexedWeather.main.temp }}Β°C
            </div>
            <div class="weather-text">
              μ²΄κ° μ¨λ: {{ this.fiexedWeather.main.feels_like }}Β°C
            </div>
            <div class="weather-text">
              μ΅λ: {{ this.fiexedWeather.main.humidity }}%
            </div>
          </div>
        </div>
      </div>
      <hr />
      <div id="apart-info-list" class="row">
        <div class="apart-info-list-title"><span>π°μ€κ±°λ λ΄μ­</span></div>
        <div class="apart-price-list">
          <ul class="list-group">
            <li
              class="list-group-item list-group-item-light"
              v-for="(item, index) in housePrices"
              :key="index"
            >
              <div class="info-apartmentName">
                <h5>{{ item.apartmentName }}</h5>
              </div>
              <div class="info-address">
                {{ item.dong }} {{ item.roadName }} {{ item.bonbun }}
              </div>
              <div class="info-dealamount">
                {{
          (parseInt(item.dealamount.replace(",", "")) * 10000) | price}}μ
           | {{ item.area }}γ‘
              </div>
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState, mapActions, mapMutations } from "vuex";
const houseStore = "houseStore";

export default {
  name: "KakaoMapModal",
  components: {},
  data() {
    return {
      busDistance: [0, 0, 0],
      schoolDistance: [0, 0, 0],
      fiexedWeather: {},
      nowHouseName: "",
    };
  },
  filters: {
    price(value) {
      if (!value) return value;
      return value.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    },
  },
  computed: {
    ...mapState(houseStore, [
      "modalVisiable",
      "housePrices",
      "selectedHouseLat",
      "selectedHouseLng",
      "selectedHouseName",
      "busesTop3",
      "schoolsTop3",
      "weather",
    ]),
  },
  watch: {
    selectedHouseLat: function () {
      //μ νν μννΈμ μ’ν κ°μ΄ λ°λλ©΄ -> κ·Έκ±Έ ν λλ‘ λ²μ€μ νκ΅ μμ 3κ° κ°μ Έμ€κΈ°
      this.searchBusStation(this.selectedHouseLat, this.selectedHouseLng);
      this.searchSchool(this.selectedHouseLat, this.selectedHouseLng);
      this.searchWeather(this.selectedHouseLat, this.selectedHouseLng);
    },
    selectedHouseName: function () {
      this.nowHouseName = this.selectedHouseName;
    },

    busesTop3: function () {
      //λ²μ€ ν3μ κ°μ΄ λ°λλ©΄ -> κ° μ§-λ²μ€ κ±°λ¦¬ κ³μ°
      this.busDistance = null;
      this.busDistance = [];
      for (var i = 0; i < this.busesTop3.length; i++) {
        var dist = this.computeDistance(
          this.busesTop3[i].gps_lati,
          this.busesTop3[i].gps_long,
          this.selectedHouseLat,
          this.selectedHouseLng
        );
        this.busDistance.push((dist * 1000).toFixed(2)); //μμμ  2μλ¦¬μκΉμ§ μΆλ ₯(λ°μ¬λ¦Ό)
      }
    },
    schoolsTop3: function () {
      //νκ΅ ν3μ κ°μ΄ λ°λλ©΄ -> κ° μ§-νκ΅ κ±°λ¦¬ κ³μ°
      this.schoolDistance = null;
      this.schoolDistance = [];
      for (var i = 0; i < this.schoolsTop3.length; i++) {
        var dist = this.computeDistance(
          this.schoolsTop3[i].gpsLati,
          this.schoolsTop3[i].gpsLong,
          this.selectedHouseLat,
          this.selectedHouseLng
        );
        this.schoolDistance.push((dist * 1000).toFixed(2));
      }
    },
  },
  created() {},
  mounted() {},
  updated() {},
  methods: {
    ...mapActions(houseStore, [
      "getBusStationTop3",
      "getSchoolTop3",
      "getDongWeather",
    ]),
    ...mapMutations(houseStore, ["SELECT_BUS", "SELECT_SCHOOL"]),
    searchBusStation(lat, lng) {
      if ((lat, lng)) this.getBusStationTop3({ lat, lng });
    },
    searchSchool(lat, lng) {
      if ((lat, lng)) this.getSchoolTop3({ lat, lng });
    },
    searchWeather(lat, lng) {
      if ((lat, lng)) this.getDongWeather({ lat, lng });
      this.fiexedWeather = this.weather;
    },
    computeDistance(startLat, startLng, endLat, endLng) {
      var startLatRads = this.degreesToRadians(startLat);
      var startLongRads = this.degreesToRadians(startLng);
      var destLatRads = this.degreesToRadians(endLat);
      var destLongRads = this.degreesToRadians(endLng);

      var Radius = 6371; //μ§κ΅¬μ λ°κ²½(km)
      var distance =
        Math.acos(
          Math.sin(startLatRads) * Math.sin(destLatRads) +
            Math.cos(startLatRads) *
              Math.cos(destLatRads) *
              Math.cos(startLongRads - destLongRads)
        ) * Radius;
      return distance;
    },
    degreesToRadians(degrees) {
      var radians = (degrees * Math.PI) / 180;
      return radians;
    },
  },
};
</script>

<style scoped>
#apart-info-modal {
  position: absolute;
  z-index: 100;
  top: 7px;
  bottom: 50px;
  right: 20px;

  width: 470px;
  border-radius: 50px;

  display: flex;
  flex-direction: row;
}

.title-bar {
  background-color: #313131;
  border-top-left-radius: 50px;
  border-top-right-radius: 50px;
  height: 70px;
  display: flex;
  justify-content: center;
  align-items: center;
  color: white;
}
.info-body {
  padding: 20px;
  height: 75%;
  overflow-y: auto;
}
.info-body::-webkit-scrollbar {
    width: 10px;
  }
.info-body::-webkit-scrollbar-thumb {
    background-color: #2f3542;
    border-radius: 10px;
    background-clip: padding-box;
    border: 2px solid transparent;
  }
.info-body::-webkit-scrollbar-track {
    background-color: rgba(214, 214, 214, 0);
    border-radius: 10px;
    box-shadow: inset 0px 0px 5px white;
  }
.apart-title {
  display: flex;
  justify-content: center;
  font-size: 20px;
  font-weight: bold;
  box-shadow: 0 8px 20px 0 rgba(0, 0, 0, 0.15);
}
#apart-info-list {
  display: flex;
  justify-content: center;
}
.apart-info-list-title {
  /* border: 1px solid black; */
  display: flex;
  justify-content: flex-start;
  font-weight: bold;

}
.apart-info-list-title span{
  background-color: rgb(70, 65, 77);
  padding-right: 7px;
  color:white;
}
.apart-price-list {
  overflow-y: auto;
}
.info-apartmentName,
.info-address,
.info-dealamount {
  display: flex;
  justify-content: flex-start;
}
.info-apartmentName{
  font-weight: bold;
  color:rgb(0, 0, 0);
  background: linear-gradient(to bottom, #fcffc8 60%, transparent 40%);
}
.weather-wrapper {
  border: 1px solid rgb(214, 214, 214);
  border-radius: 5px;
  height: 150px;
  display: flex;
  align-items: center;
}
.weather-title {
  border-right: 1px solid rgb(214, 214, 214);
  border-radius: 5px;
  width: 200px;
  height: 80%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 25px;
  font-weight: bold;
}
.weather-contents {
  display: flex;
  flex-direction: column;
  width: 100%;
  align-items: flex-start;
  padding: 0 0 0 8%;
}
.weather-text {
  font-size: 17px;
  color: gray;
  margin: 5px 0px;
}
</style>
