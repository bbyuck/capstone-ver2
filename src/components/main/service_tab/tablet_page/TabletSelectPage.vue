<template>
  <v-ons-page v-on:deviceBackButton="popPage($event)">
    <div class="picture_area">
      <img v-bind:src="'./src/assets/images/tablets/' + selectedTablet.engName + '.png'" alt="">
    </div>
    <div class="item_name">
      {{ selectedTablet.korName }}
    </div>
    <div class="date_pick">
      <HotelDatePicker 
        format="YYYY-MM-DD" 
        v-bind:maxNights="14" 
        v-bind:hoveringTooltip="true" 
        v-bind:i18n="datePickerOption" 
        v-bind:currentDateStyle="{border: '1px solid #0076ff'}"
        v-on:check-in-changed="selectStartDate($event)"
        v-on:check-out-changed="selectReturnDate($event)"
        ></HotelDatePicker>
    </div>
    <i class="far fa-calendar-alt cal_icon"></i>

    <v-ons-list>
      <v-ons-list-header class="time_header">대여 시간</v-ons-list-header>
      <v-ons-list-item>
        <div class="start_time_select">
          <v-ons-select calss="rent" v-on:modelEvent="$event === '' ? rentInfoObj.start[3] = 0 :rentInfoObj.start[3] = $event">
            <option disabled value="" class="time_option">선택</option>
            <option v-for="time in timePicker" v-bind:key="time" v-bind:value="time" class="time_option" v-if="((startDate !== '') && (((todayDate === startDate) && (time > currentTime)) || (todayDate !== startDate)))">
              {{ time >= 12 ? "PM " : "AM " }} {{ time > 12 ? time - 12 : time }}:00
            </option>
          </v-ons-select>
        </div>
      </v-ons-list-item>
      <v-ons-list-header class="time_header">반납 시간</v-ons-list-header>
      <v-ons-list-item>
        <div class="return_time_select">
          <v-ons-select v-model="returnTime" v-on:modelEvent="$event === '' ? rentInfoObj.end[3] = 0 : rentInfoObj.end[3] = $event">
            <option disabled value="" class="time_option">선택</option>
            <option v-for="time in timePicker" v-bind:key="time" v-bind:value="time" class="time_option" v-if="((returnDate !== '') && (((startDate === returnDate) && (time > startTime)) || (startDate !== returnDate)))">
              {{ time >= 12 ? "PM " : "AM " }} {{ time > 12 ? time - 12 : time }}:00
            </option>
          </v-ons-select>
        </div>
      </v-ons-list-item>

      <v-ons-list-header class="time_header">대여 장소 선택</v-ons-list-header>
      <v-ons-list-item>
        <v-ons-button v-on:click="relayout">지도보기</v-ons-button>
        <v-ons-dialog
          :visible.sync="mapOn"
          >
          <div class="cancel_btn" v-on:click="mapOn = false">
            <i class="fas fa-times-circle cancel_btn"></i>
          </div>
          <kakao-map 
            class="map_popup"
            :appkey="appkey"
            :center="center"
            :width="mapWidth"
            :height="mapHeight"
            :isShowing="mapOn"
            ></kakao-map>
        </v-ons-dialog>
      </v-ons-list-item>

      <v-ons-list-header class="time_header">반납 장소 선택</v-ons-list-header>
      <v-ons-list-item>
        <v-ons-button v-on:click="relayout">지도보기</v-ons-button>
        <v-ons-dialog
          :visible.sync="mapOn"
          >
          <div class="cancel_btn" v-on:click="mapOn = false">
            <i class="fas fa-times-circle cancel_btn"></i>
          </div>
          <kakao-map 
            class="map_popup"
            :appkey="appkey"
            :center="center"
            :width="mapWidth"
            :height="mapHeight"
            :isShowing="mapOn"
            ></kakao-map>
        </v-ons-dialog>
      </v-ons-list-item>
    </v-ons-list>

    <div class="confirm_btn">
      <!-- <v-ons-button modifier="outline" v-on:click="addListAlert">
        <i class="fas fa-clipboard-list"></i>리스트 추가
      </v-ons-button> -->
      <v-ons-button v-on:click="commit">
        <i class="fas fa-sign-out-alt"></i>대여 신청
      </v-ons-button>
    </div>

    <!-- <v-ons-alert-dialog modifier="rowfooter"
      :visible.sync="listAddDialog"
    >
      <span slot="title" v-bind:style="'fontWeight:800'">리스트 추가</span>
      리스트에 해당 아이템 추가 후<br>
      아이템 선택창으로 이동합니다.
      <template slot="footer">
        <v-ons-alert-dialog-button v-on:click="listAddDialog = false">취소</v-ons-alert-dialog-button>
        <v-ons-alert-dialog-button v-on:click="addList">추가</v-ons-alert-dialog-button>
      </template>
    </v-ons-alert-dialog> -->

  </v-ons-page>
</template>

<script>
import HotelDatePicker from 'vue-hotel-datepicker'
import KakaoMap from './KakaoMap.vue'

export default {
  key: "TabletSelectPage",
  name: "TabletSelect",
  toolBarName: "태블릿 선택",
  data: function () {
    return {
      datePickerOption: {
        night: 'Night',
        nights: 'Nights',
        'day-names': ['Sun', 'Mon', 'Tue', 'Wed', 'Thur', 'Fri', 'Sat'],
        'check-in': '대여일',
        'check-out': '반납일',
        'month-names': ['1월', '2월', '3월', '4월', '5월', '6월', '7월', '8월', '9월', '10월', '11월', '12월'],
      },
      timePicker: [9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21],
      
      startDate: "",
      returnDate: "",
      startTime: 0,
      returnTime: 0,
      
      rentInfoObj: {
        userId: window.localStorage.getItem("id"),
        itemIdx: -1,
        start: [0, 0, 0, 0, 0, 0],
        end: [0, 0, 0, 0, 0, 0],
        recommemd: 0,
        start_cabinet_idx: 1,
        end_cabinet_idx: 2
      },
      listAddDialog: false,
      mapOn: false,

      showMap: false,
      appkey:'a04f05666032a9ee976b653738fe4b63',
      center:[37.449366, 126.654386],
      mapWidth: 100,
      mapHeight: 100
    }
  },
  methods: {
    relayout: function () {
      this.mapOn = true; 
      // this.$store.state.KakaoMap.relayout();
      console.log("called");
      
    },
    commit: function () {
      console.log(JSON.parse(JSON.stringify(this.rentInfoObj)));
    },
    popPage: function ($event) {
      $event.preventDefault();
      this.$store.commit("pageStackPop");
    },
    selectStartDate: function ($event) {
      if($event === null) {
        this.startDate = "";
        return;
      }
      let startDateObj = new Date($event);
      let startYear = startDateObj.getFullYear();
      let startMonth = startDateObj.getMonth() + 1;
      let startDate = startDateObj.getDate();
      this.rentInfoObj.start[0] = startYear;
      this.rentInfoObj.start[1] = startMonth;
      this.rentInfoObj.start[2] = startDate;

      let startDateStr = startYear + "-";
      if (startMonth < 10) {
        startMonth = "0" + startMonth;
      }
      if (startDate < 10) {
        startDate = "0" + startDate;
      }
      startDateStr += startMonth;
      startDateStr += "-";
      startDateStr += startDate;

      this.startDate = startDateStr;
    },
    selectReturnDate: function ($event) { 
      if($event === null) {
        this.returnDate = "";
        return;
      }
      let returnDateObj = new Date($event);
      let returnYear = returnDateObj.getFullYear();
      let returnMonth = returnDateObj.getMonth() + 1;
      let returnDate = returnDateObj.getDate();

      this.rentInfoObj.end[0] = returnYear;
      this.rentInfoObj.end[1] = returnMonth;
      this.rentInfoObj.end[2] = returnDate;


      let returnDateStr = returnYear + "-";
      if (returnMonth < 10) {
        returnMonth = "0" + returnMonth;
      }
      if (returnDate < 10) {
        returnDate = "0" + returnDate;
      }
      returnDateStr += returnMonth;
      returnDateStr += "-";
      returnDateStr += returnDate;
      this.returnDate = returnDateStr;
    },
    addListAlert: function () {
      if(this.startDate !== "" && this.returnDate !== "" && this.startTime !== 0 && this.returnTime !== 0) {
        this.listAddDialog = true;
      } else{
        this.$ons.notification.alert("대여 정보를 입력해주세요.");
      }
    },
    addList: function () {
      if(window.sessionStorage.getItem("tablet") !== null) {
        this.listAddDialog = false;
        this.$ons.notification.alert("이미 리스트에 태블릿이 존재합니다.");
      }
      else {
        window.sessionStorage.setItem("tablet", JSON.stringify(this.rentInfoObj));
        // while(this.$store.state.pageStack.length !== 1) {
        //   this.$store.commit("pageStackPop");
        // }
        location.reload();
      }
    },
  },
  computed: {
    selectedTablet: function () {
      this.rentInfoObj.itemIdx = this.$store.getters.selectedItem.idx;
      return this.$store.getters.selectedItem;
    },
    todayDate: function () {
      let todayObj = new Date();
      let year = todayObj.getFullYear();
      let month = todayObj.getMonth() + 1;
      let date = todayObj.getDate();
      let dateStr = year + "-";
      if (month < 10) {
        month = "0" + month;
      }
      if (date < 10) {
        date = "0" + date;
      }
       dateStr += month;
      dateStr += "-";
      dateStr += date;


      return dateStr;
    },
    currentTime: function () {
      let todayObj = new Date();
      let hours = todayObj.getHours(); // 시

      return hours;
    },
    id: function () {
      return window.localStorage.getItem("id");
    }
  },
  components: {
    HotelDatePicker,
    KakaoMap
  }
}
</script>

<style>
.picture_area {
  height: 300px;
  text-align: center;
  position: relative;
}
.picture_area img {
  position: relative;
  width: 60%;
}

.item_name {
  position: relative;
  text-align: center;
  font-family: 'Noto Sans KR', sans-serif;
  font-size: 23px;
  top: 0;
}

.date_pick {
  position: relative;
  top: 5%;
}

.cal_icon {
  position: relative;
  color: #0076ff;
  font-size: 20px;
  left: 7%;
  bottom: 15px;
}

.select-input {
  font-family: 'Noto Sans KR', sans-serif;
  font-size: 23px;
  left: 10%;
  position: relative;
}

.time_select {
  position: relative;
}

.start_time_select {
  width: 85%;
  display: block;
}

.start_time_select * {
  width: 100%;
  height: 15%;
}

.return_time_select {
  width: 85%;
  display: block;
}

.return_time_select * {
  width: 100%;
  height: 15%;
}

.time_option {
  font-family: 'Noto Sans KR', sans-serif;
  font-size: 23px;
  position: absolute;
  left: 10%;
}

.confirm_btn {
  position: relative;
  margin: 10%;
  text-align: center;
}
.button{
  font-family: 'Noto Sans KR', sans-serif;
  font-size: 15px;
  width: 43%;
  margin-left: 3%;
  margin-right: 3%;
}

.list-header.time_header {
  background-color: white;
  background-image: none;
  font-family: 'Noto Sans KR', sans-serif;
  font-size: 15px;
  left: 3%;
}

.dialog {
  width: 90%;
  height: 80%;
}

.map_popup {
  height: 100%;
}

.dialog-container {
  position: relative;
  height: 100%;
}

.cancel_btn {
  display: inline-block;
  position: absolute;
  color: black;
  font-size: 30px;
  z-index: 2;
}

/*
  date Picker
 */
.datepicker__month-day--first-day-selected, .datepicker__month-day--last-day-selected, .datepicker__month-day--selected {
  background-color: #0076FF;
}
.datepicker__close-button {
  color: #0076FF;
}

.datepicker__dummy-wrapper--is-active {
  border: #0076FF;
}

.datepicker__input {
  color: #0076FF;
}

.datepicker__month-day--first-day-selected, .datepicker__month-day--last-day-selected {
  background-color: #0076ff;
}

.datepicker__clear-button {
  margin-top: 9px;
}

.datepicker__dummy-wrapper {
  border: none;
}

.datepicker__input:first-child {
  background: transparent url(http://localhost:8080/src/assets/images/dateImg/ic-arrow-right-datepicker.regular.svg) no-repeat 100%/8px;
}
</style>