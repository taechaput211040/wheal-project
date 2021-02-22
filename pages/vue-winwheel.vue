<template>
  <section class="vue-winwheel">
    <VueWinwheel
      :segments="this.dynamicOption"
      :score="this.dynamicScore"
      :url="url_savescore"
      :beforeSpin="beforeSpin"
      :dataPost="data_post"
      :redirex="redirex"
      :bg_wheel="this.dynamicbg_wheel"
      ref="foo"
    />
  </section>
</template>

<script>
import VueWinwheel from "vue-winwheel/vue-winwheel";

export default {
  components: {
    VueWinwheel,
  },
  data() {
    return {
      options: [],
      score: {
        score:50
      },
      url_savescore: "/SaveScore",
      url_getdata:
        "/getDataByToken/",
      setting: {},
      obj_score: [],
      data_post:{},
      redirex:"",
      bg_wheel:""
    };
  },
  computed: {
    dynamicOption() {
      return this.options;
    },
    dynamicScore() {
      return this.score.score;
    },
    dynamicbg_wheel(){
      return this.bg_wheel;
    }
  },
  beforecreated() {},
  async created() {
    
    let get = await this.GetData()
    this.sendData();

    var catl = await this.calculatorData()
    if(catl){
      this.SetDataPost(catl)
    }

  },
  methods: {
    AlertPrize(indicatedSegment) {
      console.log(indicatedSegment);
    },
    async GetData() {
      try {
        const token = this.$route.query.token;
        let data_setting = await this.$axios.$get(this.url_getdata+token).catch(error => {
          if (this.$axios.isCancel(error)) {
            console.log('Request canceled', error)
          } else {
            this.$swal('ไม่สามารถ ติดต่อกับ Server ได้')
            return false
          }
        });
        this.bg_wheel = process.env.API_URL+"/"+data_setting['image_luckydraw']
        this.setting = data_setting
        data_setting = data_setting['setting']
        // console.log("setting ="+JSON.stringify(data_setting ))
        let data = [];
        if (data_setting) {
          // console.log("setting = "+JSON.stringify(this.setting));
          Object.keys(data_setting).forEach(function (key) {
            // console.log("setting = "+data_setting);
            var value = data_setting[key];
            if (key % 2 === 0) {
              data.push({
                // textFillStyle: "#000",
                // fillStyle: "#fadede",
                text: value.title,
                size: 32.4,
              });
            } else {
              data.push({
                // textFillStyle: "#fff",
                // fillStyle: "#000",
                text: value.title,
                size: 57.6,
              });
            }
          });
          this.options = data;
          // console.log("data = " + JSON.stringify(this.options));
        }else{
          this.$swal('Token ถูกใช้งานแล้ว')
          console.log("error = "+error)
          // this.$router.push('/') 
          return false;
        }
      } catch (error) {

        this.$swal('Token ถูกใช้งานแล้ว')
        console.log("error = "+error)
        // this.$router.push('/') 
        return false;
        
      }
    },
    SetDataPost(last_score){
      this.data_post = {
        "username": this.setting['member']['username'],
        "agent_id": this.setting['setting'][0].agent_id,
        "setting_id": last_score['id'],
        "credit": last_score['credit'],
        "type": last_score['type']
      }
      this.redirex = this.setting['member']['call_back']
    },
    beforeSpin() {
      return new Promise((resolve, reject) => {
        resolve(true);
      });
    },
    async sendData() {
      let before = await this.beforeSpin();
      if (before) {
        //console.log("Spin");
      }
    },
    calculatorData() {
      // console.log("this.setting = "+JSON.stringify(this.setting['setting']))
      let radian = 0
      for (const key in this.options) {
        const { size } = this.options[key];
        const stopAt = this.randomInt(
          parseFloat(radian),
          parseFloat(radian) + parseFloat(size)
        );
        radian = radian + size;
        if (stopAt && this.setting['setting'][key].status == 1) {
          let json_stop = {
            "score":stopAt,
            "id":this.setting['setting'][key].id,
            "credit":this.setting['setting'][key].credit,
            "type":this.setting['setting'][key].type,
            "award_percent":this.setting['setting'][key].award_percent,
            }
          this.obj_score.push(json_stop);
        }
      }
      let array_award = []
      for (const key in this.obj_score) {
        for (var i = 0; i < this.obj_score[key].award_percent; i++) {
          array_award.push(this.obj_score[key])
        }
      }
      console.log("array_award = "+JSON.stringify(this.obj_score));
      this.score = array_award[
        Math.floor(Math.random() * array_award.length)
      ];
      return this.score
    },
    randomInt(min, max) {
      return Math.floor(Math.random() * (max - min + 1) + min);
    },
  },
};
</script>


<style scoped>
.vue-winwheel .wheel-wrapper .btn.btn-play {
  padding: 0 58px !important;
  background: #c4376f;
  height: 40px;
  line-height: 40px;
  color: white;
  font-weight: bold;
  text-decoration: none;
  border-radius: 2px;
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  letter-spacing: 2px;
}
</style>
