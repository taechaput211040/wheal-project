<template>
  <section class="vue-winwheel" id="home">
    <!-- pc frame -->
    <div class="backgroundFrame">
      <!-- currency -->
      <div class="navigator">
        <div class="d-flex">
          <div class="col-3">
            <a :href="redirex" v-if="free_spin == 0">
              <img
                :src="require('~/assets/FT-icon-back-arrow.png')"
                class="back backbtn"
                @click="redirex"
              />
            </a>
          </div>
          <div class="col-9 px-0">
            <div class="d-flex justify-content-end">
              <div class="d-flex currency_bg" style="position: relative;">
                <img src="~/assets/FT-wheel-mb.png" width="30" height="30" />
                <div class="content px-2">รอบสปิน : {{ free_spin }}</div>
              </div>
            </div>
            <div class="d-flex justify-content-end pt-2">
              <div class="d-flex currency_bg" style="position: relative;">
                <img src="~/assets/FT-icon-coin.png" width="30" height="30" />
                <div class="content px-2">เครดิต : {{ amount }}</div>
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
          :buyURL="this.url_buyspin"
          :buyToken="this.$route.query.token"
          :buy_feature="this.buy_feature"
          :reward="this.dynamicReward"
          :dataPost="this.data_post"
          :redirex="redirex"
          :bg_wheel="this.dynamicbg_wheel"
          :status="this.dynamicStatus"
          :buy_amount="this.buy_amount"
          :freespin="this.dynamicFreespin"
          ref="foo"
          @eventRefresh="$fetch()"
        />
      </div>
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
    dirs: ["~/components/vue-winwheel"]
    // VueWinwheel
  },
  data() {
    return {
      options: [],
      reward: {
        stopAt: 50
      },
      url_savescore: process.env.API_PROXY_URL + "/api/v1/SaveScore",
      url_buyspin: process.env.API_PROXY_URL + "/api/v1/BuyLuckydraw",
      setting: {},
      obj_score: [],
      data_post: {
        ref_token: ""
      },
      redirex: "",
      bg_wheel: "",
      video: {
        sources: [
          {
            src: require("~/assets/wing.mp4"),
            type: "video/mp4"
          }
        ],
        options: {
          autoplay: true,
          volume: 0,
          poster: "",
          loop: true
        }
      },
      amount: 0,
      status: 0,
      bycredit_amoun: 0,
      buy_feature: false,
      by_amount: 100,
      // isMobile: isMobile,
      free_spin: 0,
      display: false
    };
  },
  computed: {
    dynamicOption() {
      return this.options;
    },
    dynamicScore() {
      if (this.reward) {
        return this.reward.stopAt || 0;
      } else {
        return "";
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
    dynamicReftoken() {
      return this.data_post.ref_token;
    }
  },
  beforecreated() {},
  async fetch() {
    await this.GetData();
    // document.querySelector("audio").play();
    // this.sendData();
  },
  mounted() {},
  methods: {
    AlertPrize(indicatedSegment) {
      console.log(indicatedSegment);
    },
    async GetData() {
      try {
        const token = this.$route.query.token;
        let data = await this.$axios.$get(
          `${process.env.API_PROXY_URL}/api/v1/getDataByToken/${token}`
        );

        //  console.log("setting =",data_setting )
        this.bg_wheel = `${process.env.API_URL}/${data["image_luckydraw"]}`;
        this.options = data.options;
        this.reward = data.reward;
        this.data_post = data.data_post.length > 0 ? data.data_post[0] : "";
        this.redirex = data.callback_redirect;
        this.status = data.status;
        this.free_spin = data.free_spin;
        this.buy_feature = data.buy_feature;
        this.amount = data.amount;
        this.status_buy = data.status_buy;
        this.buy_amount = data.buy_amount;
        // setTimeout(() => this.$refs.foo.resetWheel(), 1000);
        this.display = true;
        console.log(this.data_post, " this.data_post this.data_post");
      } catch (error) {
        this.$swal("ไม่สามารถเชื่อมต่อได้");
        console.log("error = " + error);
        // this.$router.push('/')
      }
    },

    async sendData() {
      if (before) {
        //console.log("Spin");
      }
    }
  }
};
</script>

<style lang="scss" scoped>
.backbtn {
  position: absolute;
  cursor: pointer;
}
.backgroundFrame {
  width: 600px;
  height: 100vh;
  background: #00000080;
  margin: 0 auto;
}
.bg_vdo {
  position: absolute;
  top: 0;
  width: 100%;
  height: auto;
  z-index: -50;
  video {
    // overflow: hidden;
    width: 100%;
    height: 100%;
  }
}
.vue-winwheel {
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
  // padding: 5vh auto 2vh auto;
  padding-top: 5vh;
  padding-bottom: 2vh;
  margin: 0 auto;
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
    width: 260px;
    margin-bottom: 10px;
  }
}

.spinner-padding {
  transform: translate(0px, 10px);
}

.currency_bg {
  background-color: #00000080;
  border-radius: 50px;
  width: 150px;
  height: 30px;
  .content {
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
  .navigator {
    width: 510px;
    .back {
      width: 60px;
    }
  }
  .header {
    img {
      width: 250px;
    }
  }
  .spinner-padding {
    transform: translate(0px, 10px);
  }
}

@media (max-width: 1024px) {
  .navigator {
    width: 510px;
    .back {
      width: 50px;
    }
  }
  .header2 {
    img {
      width: 220px;
    }
  }
  .spinner-padding {
    transform: translate(0px, 30px);
  }
}

@media (max-width: 768px) {
  .navigator {
    width: 450px;
    .back {
      width: 60px;
    }
  }
  .header2 {
    img {
      width: 260px;
    }
  }
  .spinner-padding {
    transform: translate(0px, 30px);
  }
}

@media (max-width: 500px) {
  .backgroundFrame {
    width: 100%;
    background: unset;
  }
  .navigator {
    width: 320px;
    .back {
      width: 60px;
    }
  }
  .header2 {
    img {
      width: 180px;
    }
  }
  .spinner-padding {
    transform: translate(0px, 10px);
  }
}
@media (max-width: 360px) {
  .backgroundFrame {
    width: 100%;
    background: unset;
  }
  .navigator {
    position: relative;
    width: 320px;
    .back {
      width: 60px;
    }
  }
  .header2 {
    transform: translate(0px, -20px);
    img {
      width: 160px;
    }
  }
  .spinner-padding {
    transform: translate(0px, 0px);
  }
}
</style>
