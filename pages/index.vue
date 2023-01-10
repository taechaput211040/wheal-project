<template>
  <section class="vue-winwheel" id="home">
    <!-- currency -->
    <div class="navigator">
      <div class="d-flex" >
        <div class="col-3">
          <a :href="redirex" v-if="!status">
            <img
              :src="require('~/assets/FT-icon-back-arrow.png')"
              class="back"
              @click="redirex"
            />
          </a>
        </div>
        <div class="col-9 px-0">
          <div class="d-flex justify-content-end">
            <div class="d-flex currency_bg" style="position: relative;">
              <img src="~/assets/FT-wheel-mb.png" width="30" height="30" />
              <div class="content col">รอบสปิน : {{status ? '1': '0'}}</div>
            </div>
          </div>
          <div class="d-flex justify-content-end pt-2">
            <div class="d-flex currency_bg" style="position: relative;">
              <img src="~/assets/FT-icon-coin.png" width="30" height="30" />
              <div class="content col">เครดิต : {{credit}}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <!-- new header -->
    <div class="header2">
      <img :src="require('~/assets/header2.png')" alt="" class="img-fluid" />
    </div>
    <!-- <div class="header">
      <img :src="require('~/assets/header2.png')" alt="" class="img-fluid" />
    </div> -->
    <!-- spinner -->
    <div v-if="display" class="spinner-padding">
      <VueWinwheel
      :segments="this.dynamicOption"
      :score="this.dynamicScore"
      :url="url_savescore"
      :reward="this.dynamicReward"
      :beforeSpin="beforeSpin"
      :dataPost="this.dynamicData"
      :redirex="redirex"
      :bg_wheel="this.dynamicbg_wheel"
      :status="this.dynamicStatus"
      :bycredit_amount="this.bycredit_amount"
      :freespin="this.dynamicFreespin"
      ref="foo"
    />
    </div>


      <!-- :key="this.dynamicReftoken" -->
    <div class="bg_vdo">
      <!-- <video playsinline autoplay muted loop poster="" id="bgvid">
        <source :src="require('~/assets/wing.mp4')" type="video/mp4" />
      </video> -->
      <!-- <img :src="require('~/assets/wing.gif')" alt="" style="    width: 560px;"> -->
      <!-- <my-video :sources="video.sources" :options="video.options"></my-video> -->
    </div>
    <dir>
      <!-- <audio controls autoplay loop id="sound" >
        <source :src="require('~/assets/alberta.mp3')" type="audio/mpeg">

      </audio> -->

    </dir>
  </section>
</template>

<script>
// import { isMobile } from "mobile-device-detect";
// import VueWinwheel from "vue-winwheel/vue-winwheel";
export default {
  components: {
    dirs: ["~/components/vue-winwheel"],
    // VueWinwheel
  },
  data() {
    return {
      options: [],
      reward: {
        stopAt: 50,
      },
      url_savescore: process.env.API_PROXY_URL+"/SaveScore",
      setting: {},
      obj_score: [],
      data_post: {
        "ref_token":''
      },
      redirex: "",
      bg_wheel: "",
      video: {
        sources: [
          {
            src: require("~/assets/wing.mp4"),
            type: "video/mp4",
          },
        ],
        options: {
          autoplay: true,
          volume: 0,
          poster: "",
          loop: true,
        },
      },
      credit: 200,
      status: 0,
      bycredit_amoun: 0,
      bycredit_amount: 100,
      // isMobile: isMobile,
      free_spin:0,
      display:false
    };
  },
  computed: {
    dynamicOption() {
      return this.options;
    },
    dynamicScore() {
      if(this.reward){
        return this.reward.stopAt || 0;
      }else{
        return ''
      }


    },
    dynamicbg_wheel() {
      return this.bg_wheel;
    },
    dynamicReward() {
      return this.reward;
    },
    dynamicStatus() {
      return this.status;
    },
    dynamicFreespin() {
      return this.free_spin;
    },
    dynamicReftoken(){
      return this.data_post.ref_token
    },
    dynamicData(){
      if(this.data_post){
        return this.data_post[0]
      }else{
        return ''
      }

    }



  },
  beforecreated() {},
  async created() {
    let get = await this.GetData();
    // document.querySelector("audio").play();
    // this.sendData();
  },
  mounted() {

  },
  methods: {
    AlertPrize(indicatedSegment) {
      console.log(indicatedSegment);
    },
    async GetData() {
      try {
        const token = this.$route.query.token;

        // console.log("0000 ",token);
        var url_getdata =  process.env.API_PROXY_URL+"/getDataByToken/"+token

        const data_setting = await this.$axios
          .$get(url_getdata)
          .catch((error) => {
            if (this.$axios.isCancel(error)) {
              console.log("Request canceled", error);
            } else {
              this.$swal("ไม่สามารถ ติดต่อกับ Server ได้");
              return false;
            }
          });
        if (data_setting) {


        //  console.log("setting =",data_setting )
          this.bg_wheel =
            process.env.API_URL + "/" + data_setting["image_luckydraw"];
          this.options = data_setting["options"];
          this.reward = data_setting["reward"];
          this.data_post = data_setting["data_post"];
          this.redirex = data_setting["callback_redirect"];
          this.status = data_setting["status"];
          this.free_spin = data_setting["free_spin"];
          this.credit = data_setting["credit"];
          this.status_buy = data_setting["status_buy"];
          this.bycredit_amount = data_setting["bycredit_amount"];
          // setTimeout(() => this.$refs.foo.resetWheel(), 1000);
          this.display = true
        } else {
          this.display = false
          this.$swal("Token ถูกใช้งานแล้ว");
          // console.log("error = " + error);
          // this.$router.push('/')
          return false;
        }
      } catch (error) {
        this.$swal("ไม่สามารถเชื่อมต่อได้");
        console.log("error = " + error);
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


<style lang="scss" scoped>
.bg_vdo {
  position: absolute;
  top: 0;
  width: 100%;
  height: auto;
  z-index: -50;
  video {
    overflow: hidden;
    width: 100%;
    height: 100vh;
  }
}
.vue-winwheel  {
  height: 100vh;
  background-image: url("../assets/BG_web.jpg");
  background-size: cover;
  overflow-y: hidden;
}
.vue-winwheel .wheel-wrapper {
  margin-top: 121px;
}
// .vue-winwheel {
//   margin-top: 110px;
// }
.bg_vdo {
  background-image: url("../assets/BG_web.jpg");
  text-align: center;
}
.navigator {
  width: 500px;
  z-index: 1001;
  margin: 5vh auto 2vh auto;
  .back {
    width: 70px;
  }
}
// .header {
//   text-align: center;
//   position: absolute;
//   width: 100%;
//   z-index: 1000;
//   top: 100px;
//   img {
//     width: 300px;
//   }
// }

.header2 {
  display: flex;
  justify-content: center;
  position: relative;
  z-index: 1000;
  img {
    width: 300px;
  }
}

.spinner-padding{
  transform: translate(0px, 10px);
}

.currency_bg{
  background-color: #00000080;
  border-radius: 50px;
  width: 150px;
  height: 30px;
  .content{
    color: white;
  }
}

@keyframes headerani {
  0% {
    transform: rotate(0deg);
  }
  25% {
    transform: rotate(-13deg);
  }
  50% {
    transform: rotate(0deg);
  }
  75% {
    transform: rotate(-13deg);
  }
  100% {
    transform: rotate(0deg);
  }
}

/* The element to apply the animation to */
// .header img {
//   transition: all 1s;
//   transform: rotate(-13deg);
//   animation: headerani linear infinite;
//   animation-duration: 2s;
// }

.header2 img {
  transition: all 1s;
  transform: rotate(-13deg);
  animation: headerani linear infinite;
  animation-duration: 2s;
}

@media (max-width: 1400px) {
  .navigator{
    width: 450px;
    .back{
      width: 60px;
    }
  }
  .header  {
    img{
      width: 300px;
    }
  }
  .spinner-padding{
    transform: translate(0px, 10px);
  }

}

@media (max-width: 1024px) {
  .navigator{
    width: 450px;
    .back{
      width: 60px;
    }
  }
  .header2 {
    img{
      width: 280px;
    }
  }
  .spinner-padding{
    transform: translate(0px, 30px);
  }

}

@media (max-width: 768px) {
  .navigator{
    width: 450px;
    .back{
      width: 60px;
    }
  }
  .header2 {
    img{
      width: 260px;
    }
  }
  .spinner-padding{
    transform: translate(0px, 30px);
  }

}

@media (max-width: 500px) {
  .navigator{
    width: 320px;
    .back{
      width: 60px;
    }
  }
  .header2{
    img{
      width: 220px;
    }
  }
  .spinner-padding{
    transform: translate(0px, 10px);
  }

}
</style>

