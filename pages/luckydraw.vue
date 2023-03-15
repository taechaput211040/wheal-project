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
      reward: {
        stopAt:50
      },
      url_savescore: process.env.API_PROXY_URL+"/api/v1/SaveScore",
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
      return this.reward.stopAt;
    },
    dynamicbg_wheel(){
      return this.bg_wheel;
    }
  },
  beforecreated() {},
  async created() {
    let get = await this.GetData()
    this.sendData();
  },
  methods: {
    AlertPrize(indicatedSegment) {
      console.log(indicatedSegment);
    },
    async GetData() {
      try {
        const token = this.$route.query.token;
        const data_setting = await this.$axios.$get(this.url_getdata+token).catch(error => {
          if (this.$axios.isCancel(error)) {
            console.log('Request canceled', error)
          } else {
            this.$swal('ไม่สามารถ ติดต่อกับ Server ได้')
            return false
          }
        });
        if (data_setting) {
          // console.log("setting ="+JSON.stringify(data_setting ))
          this.bg_wheel = process.env.API_URL+"/"+data_setting['image_luckydraw']
          this.options = data_setting['options']
          this.reward = data_setting['reward']
          this.data_post = data_setting['data_post']
          this.redirex = data_setting['call_back']
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
