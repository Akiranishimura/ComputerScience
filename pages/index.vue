<template>
  <div>
    <div class="Title" id="Title">
      <div class="Title__heading">
        東京理科大学野田キャンパス休講確率シュミレータ
      </div>
      <div class="Title__desc">
        東京理科大学野田キャンパスが休校する確率を算出します。
      </div>
    </div>
    <div class="Date">
      <div class="container">
        <p class="Date__description">日付を入力してください</p>
        <div class="Date__inputs">
          <div class="Date__inputs__forms">
            <div class="Date__inputs__ff">
              <input class="Date__inputs__box" v-model="mm" type="text" maxlength="2" placeholder="mm"/>月
            </div>
            <div class="Date__inputs__ff">
              <input class="Date__inputs__box" v-model="dd" type="text" maxlength="2" placeholder="dd"/>日
            </div>
          </div>
          <button @click="getWeekDay" class="Date__button" >曜日を取得</button>
          <button @click="TodayWasAGoodDay" class="Date__button">今日の曜日を取得</button>
        </div>
        <div class="Date__anker" id="Anker1"></div>
        <div class="Date__alart" v-if="AlartShow">
          <div class="Date__alart__txt">
          {{ alart }}
          </div>
          <button  @click="CalcPercentage" v-if="AnotherShow" class="Date__button">休講の確率を算出する</button>
        </div>
        <div class="Date__anker" id="Anker2"></div>
        <div class="Date__another" v-if="PercentageShow">
        <div class="Date__another__percentage">
          休講の確率は<p class="Date__another__percentage__emp">
            {{ percentage }}%
            </p>
        </div>
        <div class="Date__another__alart">
        {{ AlartSecond }}
        </div>
        </div>
        現在の野田市の気温：{{ temperature }}℃
        <!-- {{ warning.warnings }}/ -->
        <!-- {{ yes }} -->
      </div>
    </div>
  </div>
</template>

<script>
// import { computed } from 'vue';
export default {
  data() {
    return {
      yyyy: "",
      mm: "",
      dd: "",
      dateString: "",
      alart: "",
      AlartShow:false,
      AnotherShow:false,
      PercentageShow:false,
      string:"",
      // temperature: 49,
      warning:[],
      percentage:0,
      AlartSecond:"",
      Calc:false,
      // yes: []
    };
  },
  async asyncData({ $axios }) {
    // API を呼び出してデータを取得する
    const tempres = await $axios.$get(
      "https://api.open-meteo.com/v1/forecast?latitude=35.95&longitude=139.87&hourly=temperature_2m&timezone=Asia%2FTokyo"
    );
    const warnres = await $axios.$get(
      "https://www.jma.go.jp/bosai/warning/data/warning/120000.json"
    );
    return {
      temperature: tempres.hourly.temperature_2m[0],
      warning: warnres.areaTypes[0].areas[0]
      // warning: warnres,
      // yes: warnres.areaTypes[0].areas[0]
    };
  },
  methods: {
    ShowAlart(){ //Alart部分表示の切り替え
      this.AlartShow = true;
      return this.AlartShow
    },
    adjustDate(){ //入力された日付をyyyy/mm/ddへ移す
    if (this.mm <= 3) {
        this.yyyy = 2023;
      } else {
        this.yyyy = 2022;
      }
      console.log(this.yyyy);
      this.dateString = this.yyyy + "/" + this.mm + "/" + this.dd;
    },
    judgeHoliday(){
      //dateStringの整形
      console.log("judge" + this.dateString)
      const date = new Date(this.dateString);
      const year = date.getFullYear();
      const month =("00" + (date.getMonth()+1)).slice(-2);
      const day = ("00" + (date.getDate())).slice(-2);
      this.string = year + "/" + month + "/" + day
      console.log(this.string)
      //祝日か否か
      if (this.string >= "2022/08/07" && this.string <= "2022/09/11"){
        this.alart = "夏休みです！"
      }
      if (this.string >= "2022/12/25" && this.string <= "2023/01/09"){
        this.alart = "冬休みです！"
      }
      if (this.string >= "2023/1/29"){
        this.alart = "春休みです！"
      }
      const holidays = ["2022/05/02","2022/05/03","2022/05/04","2022/05/05","2022/07/22","2022/07/23","2022/11/03","2022/11/22","2022/11/23","2022/11/25","2022/11/28","2023/01/10","2023/01/11","2023/01/12","2023/01/13"]
      if(holidays.includes(this.string)){
        this.alart = "祝日のためお休みです！珍しい！"
      }
    },
    calcDate(){
      //曜日の判定と表示
      const date = new Date(this.dateString);
      const weekDayNumber = date.getDay();
      const weekDay = ["日曜", "月曜", "火曜", "水曜", "木曜", "金曜", "土曜"][
        weekDayNumber
      ];
      console.log(weekDay)
      this.weekDay = weekDay;
      if (weekDay == "日曜") {
        // console.log(alart)
        this.alart = "日曜のため大学がありません。おめでとうございます！！"
        return;
      } else if (weekDay == "土曜") {
        // console.log(alart)
        this.alart = "土曜のため（補講がなければ）大学がありません！おやすみなさい！"
        return;
      } else {
        this.alart = "平日です。勿論大学があります！！！！";
      }
      this.judgeHoliday()
    },
    getToday(){
      //今日の日付入手
      var now = new Date();
      console.log(now)
      this.dateString = now;
    },
    ShowAnother(){
      this.AnotherShow = true;
      return this.AnotherShow
    },
    CloseAnother(){
      this.AnotherShow = false;
      return this.AnotherShow
    },
    ShowPercentage(){
      this.PercentageShow = true;
      return this.PercentageShow
    },
    ClosePercentage(){
      this.PercentageShow = false;
      return this.PercentageShow
    },
    judgeToday(){
      //入力された日付が今日か否か
      var TFJ = new Date();
      console.log("now is"+ TFJ)
      var Tyear = TFJ.getFullYear();
      var Tmonth =("00" + (TFJ.getMonth()+1)).slice(-2);
      var Tday = ("00" + (TFJ.getDate())).slice(-2);
      const TodayString = Tyear + "/" + Tmonth + "/" + Tday
      console.log(TodayString)
      console.log("string is" + this.string)
      if(this.string == TodayString){
        console.log("その日は今日") //今日ならば確率算出へ
        this.ShowAnother()
      }
      else{
        console.log("その日は今日janai")
        this.CloseAnother()
      }
      return this.AnotherShow

    },

    getWeekDay() { //入力された日時取得→表示
      window.location.hash = "Title" //一度トップへ戻す
      this.CloseAnother()
      this.ShowAlart() //Alart表示
      window.location.hash = "Anker1" //アラート部分へ遷移
      this.adjustDate() //日にちの整形
      this.calcDate() //曜日計算・表示
      this.judgeToday() //今日か否か

    },
    TodayWasAGoodDay(){//今日を表示→取得
      window.location.hash = "Title"
      this.CloseAnother()
      this.ShowAlart()
      window.location.hash = "Anker1"
      this.getToday() //今日の取得
      this.calcDate()
      this.ShowAnother()
    },
      CalcWarning(){
        const warnings =  this.warning.warnings
        console.log(warnings)
        for (let i = 0; i < (warnings.length); i++) {
          const codes = warnings[i].code //数値型へ変換
          const num = Number(codes)
          console.log(num)
          if(num < 9 ){
            this.percentage += 15
            console.log("警報")
            this.AlartSecond = "警報が発令中です"
          }
          else if ( num >= 10 && num <=26){
            console.log("注意報")
            this.AlartSecond="注意報が発令していますが、大学は存在するでしょう"
          }
          else if( num >= 32 && num <= 36){
            this.AlartSecond = "特別警報が発令中です"
            this.percentage +=30
            console.log("特別警報")
          }
}
          return this.percentage, this.AlartSecond
      },
    CalcTemp(){
      console.log("温度計算")
      if(this.temperature <= -5){
        this.percentage += 15
        this.AlartSecond = "寒さに負けずがんばろう"
      }
      else if(this.temperature <=0){
        this.percentage += 10
        console.log(this.percentage)
        this.AlartSecond = "寒さに負けずがんばろう"
      }
      else if(this.temperature <= 30){
        this.percentage += 0
        console.log(this.percentage)
      }
      else if(this.temperature <= 40){
        this.percentage += 10
        console.log(this.percentage)
        this.AlartSecond = "暑さに負けずがんばろう"
      }
      else{
        this.percentage += 15
        this.AlartSecond = "灼熱の中でも大学はある...はず"
      }
        return this.percentage, this.AlartSecond
    },
    CalcPercentage(){
            window.location.hash = "Title"
      this.ShowPercentage()
      window.location.hash = "Anker2"
      this.percentage = 0 // Reset Percentage
      this.CalcTemp()
      this.CalcWarning()
      if(this.percentage >= 100){
        this.percentage = 100
      }
      return this.percentage, this.AlartSecond
    }
  },
  // computed:{
  //   yes: function () {
  //       return JSON.parse(this.warning)
  //   }
  // }
};

</script>

<style lang='scss' scoped>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700&family=Noto+Sans+JP:wght@400;700&display=swap');
.Date {
  padding-top:2rem;
  padding-left:5rem ;
  background-color: $Base;
  font-family: 'Inter','Noto Sans JP', sans-serif;
  // padding-top:4rem;
  &__description {
    font-weight: 700;
    font-size: 1.2rem;
    color: $Color;
  }
  &__inputs{
    // align-items: center;
    font-size:1.1rem;
    font-weight: 700;
    margin-top:1rem;
    padding-bottom: 2rem;
    &__forms{
      display:flex;
      gap:0.5rem;
    }
    &__ff{
      max-width: 40%;
    }
    &__box{
      max-width: 50%;
      margin-top:0.2rem;
      margin-bottom: 0.2rem;
      &::placeholder{
        color:$Light;
      }
      background-color: #fff;
      border: $Main 2px solid;
    }
  }
  &__button{
    margin-top: 1rem;
    // margin-left:1.5rem;
    font-size: 1.2rem;
    color:$Base;
    border:none;
    background-color: $Light;
    padding:0.2rem 1rem 0.2rem 1rem;
    &:hover{
      background-color: $Main;
      font-size:1.3rem;
      color:$Base;
    }
  }
  &__alart{
    font-size: 1.5rem;
    display:flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    font-weight: 700;
    color:$Main;
    height: 100vh;
  }
  &__another{
    height: 100vh;
    display:flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    &__percentage{
      font-size :1.1rem;
      display:flex;
      font-weight: bold;
      align-items: center;
      &__emp{
        margin-left:0.5rem;
        font-size:2rem;
        color:$Main;
      }
    }
  }
}
.Title{
    &__heading{
      font-size: 1.5rem;
      font-weight: 700;
    }
    &__desc{
      font-weight: 400;
      font-size: 0.9rem;
    }
    padding-left: 4rem;
    background-color: $Dark;
    color:$Base;
    padding-top:6rem;
    padding-bottom: 6rem;
  }


body{
  // font-family: "Helvetica Neue",
    background-color: $Base;

  font-family: 'Inter','Noto Sans JP', sans-serif;
}
</style>
