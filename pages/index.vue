<template>
  <section class="vue-winwheel" id="home">
    <div class="header">
      <img :src="require('~/assets/header2.png')" alt="" class="img-fluid" />
    </div>
    <VueWinwheel
      :segments="this.dynamicOption"
      :score="this.dynamicScore"
      :url="url_savescore"
      :reward="this.dynamicReward"
      :beforeSpin="beforeSpin"
      :dataPost="data_post"
      :redirex="redirex"
      :bg_wheel="this.dynamicbg_wheel"
      :status="this.dynamicStatus"
      :key="data_post.ref_token"
      ref="foo"
    />
    <div class="bg_vdo">
      <video playsinline autoplay muted loop poster="" id="bgvid">
        <source :src="require('~/assets/wing.mp4')" type="video/mp4" />
      </video>
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
import { isMobile } from "mobile-device-detect";
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
      url_savescore: process.env.API_URL+"/SaveScore",
      url_getdata: process.env.API_URL+"/getDataByToken/",
      setting: {},
      obj_score: [],
      data_post: {},
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
      status: 0,
      isMobile: isMobile,
    };
  },
  computed: {
    dynamicOption() {
      return this.options;
    },
    dynamicScore() {
      return this.reward.stopAt;
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
        const data_setting = await this.$axios
          .$get(this.url_getdata + token)
          .catch((error) => {
            if (this.$axios.isCancel(error)) {
              console.log("Request canceled", error);
            } else {
              this.$swal("ไม่สามารถ ติดต่อกับ Server ได้");
              return false;
            }
          });
        if (data_setting) {
          // console.log("setting ="+JSON.stringify(data_setting ))
          this.bg_wheel =
            process.env.API_URL + "/" + data_setting["image_luckydraw"];
          this.options = data_setting["options"];
          this.reward = data_setting["reward"];
          this.data_post = data_setting["data_post"];
          this.redirex = data_setting["call_back"];
          this.status = data_setting["status"];
          setTimeout(() => this.$refs.foo.resetWheel(), 1500);
        } else {
          // this.$swal("Token ถูกใช้งานแล้ว");
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
  z-index: -50;
  video {
    width: 100%;
    height: 100vh;
  }
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
.header {
  text-align: center;
  position: absolute;
  width: 100%;
  z-index: 1000;
  top: 0;
  img {
    width: 300px;
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
.header img {
  transition: all 1s;
  transform: rotate(-13deg);
  animation: headerani linear infinite;
  animation-duration: 2s;
}
@media (max-width: 960px) {
  .vue-winwheel {
    margin-top: 50vw;
  }
  .header img {
    width: 250px;
  }
}
</style>

