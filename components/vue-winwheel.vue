<template>
  <section class="vue-winwheel-component">
    <div class="mobile-container">
      <img
        class="btn_sound"
        :src="require('~/assets/volume.png')"
        style="width=35px; height: 35px;"
        @click="tougleSound()"
        v-if="!muteSound"
      />
      <img
        class="btn_sound"
        :src="require('~/assets/volume-mute.png')"
        style="width=35px; height: 35px;"
        alt="spin button"
        v-else
        @click="tougleSound()"
      />

      <div class="wheel-wrapper">
        <div class="canvas-wrapper">
          <!-- width 320 && height 770 **** very small\old phone-->
          <canvas
            id="canvas"
            width="300"
            height="300"
            v-if="smallerThanIPAD && verySmallPhone"
          >
          </canvas>
          <div class="spin_button" v-if="smallerThanIPAD && verySmallPhone">
            <img
              :src="require('~/assets/BTN-spin.png')"
              alt="spin button"
              v-if="freespin != 0"
              @click="startSpin"
            />
            <img
              :src="require('~/assets/BTN-non-spin.png')"
              style="width=100%; height:auto;"
              alt="spin button"
              v-else
            />
          </div>
          <!-- 375 > width > 768-->
          <canvas
            id="canvas"
            width="320"
            height="320"
            v-if="smallerThanIPAD && !verySmallPhone"
          >
          </canvas>
          <div class="spin_button" v-if="smallerThanIPAD && !verySmallPhone">
            <img
              :src="require('~/assets/BTN-spin.png')"
              class=" spiner"
              alt="spin button"
              v-if="
                freespin != 0 && wheelSpinning == false && checkbuySpin == false
              "
              @click="startSpin"
            />
            <img
              :src="require('~/assets/BTN-non-spin.png')"
              style="width=100%; height:auto;"
              class=" spiner"
              alt="spin button"
              v-else
            />
          </div>
          <!-- ipad or larger screen -->
          <canvas id="canvas" width="450" height="450" v-if="!smallerThanIPAD">
          </canvas>
          <div class="spin_button" v-if="!smallerThanIPAD">
            <img
              :src="require('~/assets/BTN-spin.png')"
              alt="spin button"
              v-if="
                freespin != 0 && wheelSpinning == false && checkbuySpin == false
              "
              @click="startSpin"
            />
            <img
              :src="require('~/assets/BTN-non-spin.png')"
              alt="spin button"
              v-else
            />
          </div>
          <!-- <div style="transform: translate(0px, -250px);">middle</div> /-->
        </div>
        <div class="button-wrapper">
          <div class="buyspin" v-if="freespin == 0 && buy_feature">
            ซื้อฟรีสปิน {{ buy_amount }} เครดิต
          </div>
          <!-- <a
            class="btn px-0 py-0"
            href="#"
            @click.prevent="startSpin()"
            v-if="!loadingPrize && !wheelSpinning && status == 0"
          >
          loadingPrize: {{loadingPrize}}
          wheelSpinning: {{wheelSpinning}}
          status : {{status}}
            <img :src="require('~/assets/BTN-bye.png')" alt="" />
          </a> -->
          <div class="btn px-0 py-0" v-if="freespin == 0 && buy_feature">
            <img
              :src="require('~/assets/BTN-bye.png')"
              alt=""
              @click="showBuy"
            />
          </div>
          <a
            class="btn px-0 py-0"
            :href="this.redirex"
            v-if="freespin !== 0 && buy_feature"
          >
            <img :src="require('~/assets/BTN-back.png')" alt="" />
          </a>
          <a class="btn px-0 py-0" :href="this.redirex" v-if="!buy_feature">
            <img :src="require('~/assets/BTN-back.png')" alt="" />
          </a>

          <!-- <a
            class="btn btn-play"
            href="#"
            @click.prevent="saveData()"
            v-if="!loadingPrize && !wheelSpinning"
            >saveData!</a> -->
        </div>
      </div>
    </div>

    <!-- <div class="footer">
      <h2>FreeSpin : {{ freespin - this.total_spin }}</h2>
    </div> -->

    <!-- <div class="custom-modal modal-mask" id="modalSpinwheel" v-if="modalPrize">
				<div slot="body">
					<a href="" @click.prevent="hidePrize()" class="modal-dismiss">
						<i class="icon_close"></i>
					</a>
					<h2>
						Yay you got the prize!!
					</h2>
					<h1> {{prizeName}}</h1>
				</div>
			</div> -->

    <b-modal
      ref="modal_reward"
      hide-footer
      hide-header
      centered
      no-close-on-backdrop
      modal-class="modal-bg modal_reward "
      id=""
    >
      <div class="bg_modal my-auto">
        <div class="d-block text-center">
          <h3>{{ this.reward_title }}</h3>
          <div @click="closeModel()">
            <img
              class="btn_class"
              :src="require('~/assets/btn_reward.png')"
              alt=""
            />
          </div>
        </div>
      </div>
    </b-modal>
    <b-modal
      ref="modal_fail_buy"
      hide-footer
      hide-header
      centered
      no-close-on-backdrop
      modal-class="modal_buy_fail"
      id=""
    >
      <div class="bg_modal my-auto">
        <div class="d-block text-center">
          <h3>ซื้อ ไม่สำเร็จ กรุณาลองใหม่</h3>
          <div class="d-flex justify-content-center">
            <img
              class="btn_class"
              @click="backToBuy"
              :src="require('~/assets/BTN-back.png')"
              alt=""
            />
          </div>
        </div>
      </div>
    </b-modal>
    <b-modal
      ref="modal_successful_buy"
      hide-footer
      hide-header
      centered
      no-close-on-backdrop
      modal-class="modal_buy_success"
      id=""
    >
      <div class="bg_modal my-auto">
        <div class="d-block text-center">
          <h3>ซื้อ สำเร็จ</h3>
          <div class="d-flex justify-content-center">
            <img
              class="btn_class"
              @click="refresh"
              :src="require('~/assets/btn-spsin-now.png')"
              alt=""
            />
          </div>
        </div>
      </div>
    </b-modal>
    <b-modal
      ref="modal_reward"
      hide-footer
      hide-header
      centered
      no-close-on-backdrop
      modal-class="modal-bg modal_reward "
      id=""
    >
      <div class="bg_modal my-auto">
        <div class="d-block text-center">
          <h3>{{ this.reward_title }}</h3>
          <div @click="closeModel()">
            <img
              :src="require('~/assets/btn_reward.png')"
              class="btn_class"
              alt=""
            />
          </div>
        </div>
      </div>
    </b-modal>
    <b-modal
      ref="modal_buy"
      hide-footer
      hide-header
      centered
      no-close-on-backdrop
      modal-class="modal_buy"
      id=""
    >
      <div class="bg_modal my-auto">
        <div class="d-block text-center">
          <h3>ซื้อ spin</h3>
          <div class="d-flex justify-content-center">
            <img
              class="btn_class"
              @click="buySpin"
              :src="require('~/assets/BTN-bye.png')"
              alt=""
            />
            <img
              class="btn_class"
              @click="hideBuy"
              :src="require('~/assets/BTN-cancle.png')"
              alt=""
            />
          </div>
        </div>
      </div>
    </b-modal>

    <b-modal
      ref="modal_before"
      hide-footer
      hide-header
      modal-class=" modal-bg modal_before modal-dialog-centered"
      id=""
    >
      <div class="bg_modal">
        <div class="d-block text-center">
          <img
            class="img-fluid img_welcome"
            :src="require('~/assets/motion-lucky-spin.png')"
          />
          <!--  -->
          <div class="d-flex justify-content-center">
            <img
              class="btn_class btn_welcome"
              @click="hideModal"
              :src="require('~/assets/btn-start.png')"
              alt=""
            />
          </div>

          <!-- @click.prevent="playSound('http://localhost:8001/static/alberta.mp3')" -->
        </div>
      </div>
      <!-- <b-button class="mt-3" block @click="$bvModal.hide('bv-modal-example')"
        >Close Me</b-button> -->
    </b-modal>

    <div v-if="isLoading">
      <div class="loading">Loading&#8230;</div>
    </div>
  </section>
</template>

<script>
// import Loading from "vue-loading-overlay";
// import "vue-loading-overlay/dist/vue-loading.css";

import * as Winwheel from "~/node_modules/vue-winwheel/Winwheel.js";
import { isMobile } from "mobile-device-detect";

export default {
  name: "VueWinWheel",
  props: {
    segments: {
      type: Array
    },
    score: {
      default() {
        return 30;
      }
    },

    url: {
      default() {
        return "www.hotmail.com";
      }
    },
    buyURL: {
      default() {
        return "/BuyLuckydraw";
      }
    },
    buyToken: {
      default() {
        return "";
      }
    },
    buy_feature: {
      default() {
        return false;
      }
    },
    dataPost: {
      default() {
        return "";
      }
    },
    reward: {
      default() {
        return "";
      }
    },
    status: {
      default() {
        return 0;
      }
    },
    redirex: "",
    freespin: {
      decheckbuySpinfault() {
        return "";
      }
    },

    buy_amount: {
      tyoe: Number
    }
  },
  data() {
    return {
      muteSound: false,
      checkbuySpin: false,
      loadingPrize: false,
      theWheel: null,
      modalPrize: false,
      modalBuy: false,
      wheelPower: 1,
      wheelSpinning: false,
      prizeName: "BUY 1 GET 1",
      WinWheelOptions: {
        textFontSize: 20,
        textFillStyle: "#fff",
        outterRadius: 410,
        innerRadius: 25,
        lineWidth: 8,
        drawText: true,
        animation: {}
      },
      audio: null,
      wheelImage: new Image(),
      isMobile: isMobile,
      sound_bg: new Audio(
        "https://s3.ap-southeast-1.amazonaws.com/image-storage-betkub/luckydraw/20210829153536b82885fea83a4f81aec70eb5f97c0840.mp3"
      ),
      sound_reward: new Audio(
        "https://s3.ap-southeast-1.amazonaws.com/image-storage-betkub/luckydraw/20210829153634d5d55b635d7848c4b8ec0b98dacae384.mp3"
      ),
      fontSize: 14,
      reward_title: "",
      total_spin: 0,
      isLoading: false,
      fullPage: true,
      smallerThanIPAD: false,
      verySmallPhone: false
    };
  },
  methods: {
    showPrize() {
      this.modalPrize = true;
    },
    hidePrize() {
      modal_fail_buy;
      this.modalPrize = false;
    },
    showBuy() {
      this.modalBuy = true;
      console.log("show buy popup");
      this.$refs["modal_buy"].show();
    },
    hideBuy() {
      this.modalBuy = false;
      this.$refs["modal_buy"].hide();
    },
    async buySpin() {
      this.$refs["modal_buy"].hide();
      try {
        let { data } = await this.$axios.post(this.buyURL, {
          ref_token: this.buyToken
        });
        console.log(data, "rsert");
        if (data._id) {
          this.checkbuySpin = true;
        }

        this.$refs["modal_successful_buy"].show();
      } catch (error) {
        this.checkbuySpin = false;
        this.$refs["modal_fail_buy"].show();
      }
      this.$emit("eventRefresh");
    },

    async saveData() {
      const data = await this.$axios.$get(this.url);
      // console.log(data);
    },
    async startSpin() {
      // this.beforeSpinDefault().then((result) => {
      await this.onReceive();
      if (this.wheelSpinning === false) {
        // this.theWheel.startAnimation();
        // this.wheelSpinning = true;
        this.theWheel = new Winwheel.Winwheel({
          ...this.WinWheelOptions,
          numSegments: this.segments.length,
          segments: this.segments,
          textFontSize: this.fontSize,
          // drawMode: 'segmentImage',
          // outerRadius  : 212,
          drawMode: "image",
          drawText: true,
          animation: {
            type: "spinToStop",
            duration: 15,
            spins: 8,
            callbackFinished: this.onFinishSpin,
            callbackSound: this.playSoundPin(),
            soundTrigger: "pin"
          }
          // pins: { number: 24 },
        });

        // example input prize number get from Backend
        // Important thing is to set the stopAngle of the animation before stating the spin.

        var prizeNumber = Math.floor(Math.random() * this.segments.length); // or just get from Backend
        var stopAt =
          (360 / this.segments.length) * prizeNumber -
          360 / this.segments.length / 2; // center pin
        // var stopAt = 360 / this.segments.length * prizeNumber - Math.floor(Math.random() * 60) //random location

        this.theWheel.wheelImage = this.wheelImage;
        this.theWheel.animation.stopAngle = this.reward[0].stopAt;
        this.theWheel.startAnimation();
        // this.wheelSpinning = false;
        this.wheelSpinning = true;
        if (this.reward[0] && this.reward[0].title) {
          this.reward_title = this.reward[0].title;
        } else {
          this.reward_title = "";
        }
      }
      // });
    },
    resetWheel() {
      // require("~/assets/tickticktick.mp3")
      window.screen.width < 320 || window.screen.height < 770
        ? (this.verySmallPhone = true)
        : (this.verySmallPhone = false);
      window.screen.width < 768
        ? (this.smallerThanIPAD = true)
        : (this.smallerThanIPAD = false);
      // console.log(this.isMobile);
      // console.log(this.smallerThanIPAD);

      this.audio = new Audio(
        "https://s3.ap-southeast-1.amazonaws.com/image-storage-betkub/luckydraw/202108291537181d53e6b561c84bb4ab33054f80ca2e27.mp3"
      );
      // console.log("resetWheel");
      this.theWheel = new Winwheel.Winwheel({
        ...this.WinWheelOptions,
        numSegments: this.segments.length,
        segments: this.segments,
        textFontSize: this.fontSize,
        textFillStyle: "#fff",
        // drawMode: 'segmentImage',
        drawMode: "image",
        drawText: true
      });

      if (this.wheelSpinning) {
        this.theWheel.stopAnimation(false); // Stop the animation, false as param so does not call callback function.
      }
      // console.log("Wheel",  this.theWheel.wheelImage )
      this.theWheel.wheelImage = this.wheelImage;

      this.theWheel.rotationAngle = 0; // Re-set the wheel angle to 0 degrees.
      this.theWheel.draw(); // Call draw to render changes to the wheel.
      this.wheelSpinning = false; // Reset to false to power buttons and spin can be clicked again.
    },
    initSpin() {
      this.loadingPrize = true;
      this.resetWheel();
      this.loadingPrize = false;
      console.log("this.freespin =", this.freespin);
    },
    onFinishSpin(indicatedSegment) {
      this.audio.pause();
      this.playSoundReward();
      this.prizeName = indicatedSegment.text;
      this.showPrize();
      this.$refs["modal_reward"].show();
    },
    closeModel() {
      this.wheelSpinning = false;
      this.isLoading = false;
      // remove array frist
      this.reward.shift();
      // this.dataPost.shift();
      // this.$refs["modal_reward"].hide();
      this.total_spin = this.total_spin + 1;
      // console.log("this.freespin - this.total_spin = ",this.freespin - this.total_spin);
      if (this.freespin - this.total_spin > 0) {
        this.reward_title = this.reward[0].title;
        this.loadingPrize = true;
        this.resetWheel();
        this.loadingPrize = false;
      } else {
        this.$emit("eventRefresh");
        // this.$nuxt.refresh();
      }
      this.$refs["modal_reward"].hide();
    },
    async onReceive() {
      // this.isLoading = true;
      await this.$axios
        .post(
          this.url,
          { ...this.dataPost, buy: this.checkbuySpin },
          {
            auth: {
              username: "ten",
              password: "3900"
            }
          }
        )
        .then(
          response => {
            // console.log("this.dataPost ",this.dataPost)
            console.log("close modal");
            console.log("response ", response);
            this.checkbuySpin = false;
            // this.isLoading = false;
            // // remove array frist
            // this.reward.shift();
            // // this.dataPost.shift();
            // this.$refs["modal_reward"].hide();
            // this.total_spin = this.total_spin + 1;
            // // console.log("this.freespin - this.total_spin = ",this.freespin - this.total_spin);
            // if (this.freespin - this.total_spin > 0) {
            //   this.reward_title = this.reward[0].title;
            //   this.loadingPrize = true;
            //   this.resetWheel();
            //   this.loadingPrize = false;
            // } else {
            //   location.reload();
            //   // this.$nuxt.refresh();
            // }
          },
          error => {
            this.isLoading = false;
            this.$swal("ไม่สามารถบันทึกข้อมุลได้ กรุณาลองใหม่");
            console.log(error);
            // location.reload();
          }
        );
    },
    playSoundPin() {
      this.audio.pause();
      this.audio.currentTime = 0;
      this.audio.play();
    },
    playSoundBg() {
      // var  myMusic = new sound("https://www.w3schools.com/graphics/gametheme.mp3");
      // myMusic.play();

      this.sound_bg.pause();
      this.sound_bg.currentTime = 0;
      this.sound_bg.setAttribute("crossorigin", "anonymous");
      if (this.sound_bg) {
        this.sound_bg.play();
      }
    },
    tougleSound() {
      this.muteSound = !this.muteSound;
      if (this.muteSound) {
        this.sound_bg.pause();
      } else {
        this.sound_bg.play();
      }
    },
    playSoundReward() {
      this.sound_reward.pause();
      this.sound_reward.currentTime = 0;
      if (this.sound_reward) {
        this.sound_reward.play();
      }
    },
    hideModal() {
      this.$refs["modal_before"].hide();
      this.playSoundBg();
      this.resetWheel();
    },
    playSound(sound) {
      if (sound) {
        var audio = new Audio(sound);
        audio.play();
      }
    },
    async refresh() {
      await this.$refs["modal_successful_buy"].hide();
      await this.startSpin();
    },
    backToBuy() {
      this.$refs["modal_fail_buy"].hide();
      this.$refs["modal_buy"].show();
    }
  },
  computed: {},
  // updated() {},
  // watch: {
  //   wheelImage(n , o) {
  //     console.log("Image", n, o)
  //     this.resetWheel();
  //   },
  // },
  mounted() {
    this.wheelImage = new Image();
    window.screen.width < 320 || window.screen.height < 770
      ? (this.verySmallPhone = true)
      : (this.verySmallPhone = false);
    window.screen.width < 768
      ? (this.smallerThanIPAD = true)
      : (this.smallerThanIPAD = false);
    if (this.smallerThanIPAD && this.verySmallPhone) {
      this.wheelImage.src = require("~/assets/FT-wheel-mb-xs.png");
    } else if (this.smallerThanIPAD && !this.verySmallPhone) {
      this.wheelImage.src = require("~/assets/FT-wheel-mb.png");
    } else {
      this.fontSize = 20;
      this.wheelImage.src = require("~/assets/FT-wheel-dt.png");
    }

    // console.log("status = " + this.status);

    this.$refs["modal_before"].show();
    // this.initSpin();

    // this.resetWheel()
    // this.beforeSpinDefault().then((result) => {
    // setTimeout(() => this.resetWheel(), 300);
    // });
  },

  created() {
    // if('title' in this.reward[0] ){
    //   this.reward_title = this.reward[0].title
    // }
    // if('title' in this.reward[0] ){
    // }
    // console.log("reward ",this.reward[0]);
  }
};
</script>

<style lang="scss" scoped>
.img_welcome {
  max-width: 100%;
  height: auto;
  transform: scale(1.29);
}
.btn_welcome {
  height: 100px;
  width: auto;
  bottom: 0;
  position: absolute;
}
.btn_sound {
  position: fixed;
  right: 30px;
  cursor: pointer;
  bottom: 35px;
  z-index: 999;
  filter: invert(1);
}
.btn_class {
  cursor: pointer;
}
.d-fix {
  position: fixed;
  top: 50vh;
  left: 49vw;
}
.vue-winwheel-component {
  text-align: center;
  // background-image: url("../assets/wheel.png");
  /* background-image: url("../assets/wing.mp4"); */
}
.vue-winwheel-component h1 {
  color: #b32656;
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  font-size: 36px;
  line-height: 90px;
  letter-spacing: 4px;
  margin: 0;
}
.vue-winwheel-component h2 {
  margin: 0;
}
.vue-winwheel-component #modalSpinwheel.custom-modal .content-wrapper .content {
  width: calc(100vw - 30px);
  padding-top: 52px;
}
.vue-winwheel-component
  #modalSpinwheel.custom-modal
  .content-wrapper
  .content
  h2 {
  text-transform: uppercase;
  color: #b32656;
  margin-bottom: 16px;
  margin-top: 0;
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  font-size: 18px;
  letter-spacing: 1.1px;
  margin: 0;
}
.vue-winwheel-component
  #modalSpinwheel.custom-modal
  .content-wrapper
  .content
  p {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  font-size: 14px;
  color: black;
  text-align: center;
  line-height: 25px;
}
.vue-winwheel-component
  #modalSpinwheel.custom-modal
  .content-wrapper
  .content
  p
  strong {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
}
.vue-winwheel-component
  #modalSpinwheel.custom-modal
  .content-wrapper
  .content
  .modal-dismiss {
  top: 12px;
  right: 12px;
}
.vue-winwheel-component
  #modalSpinwheel.custom-modal
  .content-wrapper
  .content
  .modal-dismiss
  i.icon_close {
  font-size: 30px;
  color: #da2a52;
}
.vue-winwheel-component canvas#canvas {
  position: relative;
}
.vue-winwheel-component .canvas-wrapper {
  position: relative;
}
.vue-winwheel-component .canvas-wrapper:after {
  content: "";
  display: block;
  width: 42px;
  /* background: #c4376f; */
  height: 42px;
  position: absolute;
  left: calc(50% - 25px);
  margin: auto;
  border-radius: 100%;
  top: calc(50% - 29px);
  /* border: 5px solid white; */
  box-sizing: content-box;
}

.vue-winwheel-component .canvas-wrapper:before {
  content: "";
  display: block;
  width: 320px;
  background: #0f0f0f;
  height: 320px;
  position: absolute;
  left: 0;
  right: 0;
  margin: 0 auto;
  border-radius: 100%;
  top: 0;
}

.spin_button {
  transform: translate(0px, -220px);
}

.spin_button img {
  width: 133px;
  height: auto;
}
@media (min-width: 768px) {
  .vue-winwheel-component .canvas-wrapper:before {
    width: 450px;
    height: 450px;
  }

  .spin_button {
    transform: translate(0px, -330px);
  }

  .spin_button img {
    width: 200px;
    height: auto;
  }
}

.vue-winwheel-component .wheel-wrapper {
  position: relative;
}
.vue-winwheel-component .wheel-wrapper:before {
  content: "";
  width: 62px;
  height: 47px;
  position: absolute;
  top: -10px;
  left: calc(50% - 31px);
  right: 0;
  display: block;
  z-index: 999;
  background-image: url("../assets/FT-point.png");
  /* background-image: url("~/node_modules/vue-winwheel/spinner-marker.svg"); */
  background-repeat: no-repeat;
  background-size: contain;
  background-position: center;
}
.vue-winwheel-component .wheel-wrapper .button-wrapper {
  margin: -100px auto 0px auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 231px;
  height: 118px;
  z-index: 10;
  position: relative;
}
.vue-winwheel-component .wheel-wrapper .btn.btn-play {
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
.btn {
  img {
    width: 170px;
  }
}
.modal_reward {
  .bg_modal {
    h3 {
      margin-top: 30%;
    }
    a {
      width: 100%;
      text-align: center;
      bottom: 24vw;
      position: fixed;
      left: 0;
    }
    img {
      width: 170px;
    }
  }
}
.modal_buy,
.modal_buy_success,
.modal_buy_fail {
  h3 {
    margin-top: 30%;
    padding-bottom: 15%;
    color: white;
  }
  img {
    width: 170px;
  }
}
.pointer {
  cursor: pointer;
  margin-top: 50%;
}
@media (max-height: 770px) {
  .vue-winwheel-component .canvas-wrapper:before {
    content: "";
    display: block;
    width: 300px;
    background: #0f0f0f;
    height: 300px;
    position: absolute;
    left: 0;
    right: 0;
    margin: 0 auto;
    border-radius: 100%;
    top: 0;
  }
  .vue-winwheel-component .wheel-wrapper .button-wrapper {
    margin: -135px auto 0px auto;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    width: 231px;
    height: 118px;
  }
}
@media (max-width: 500px) {
  .modal_buy,
  .modal_buy_success,
  .modal_buy_fail {
    h3 {
      font-size: 20px;
      margin-top: 30%;
      padding-bottom: 13%;
    }
    img {
      width: 100px;
    }
  }
  .modal-dialog .modal-md .modal-dialog-centered {
    margin: auto;
  }
}

@media (max-width: 320px) {
  .vue-winwheel-component .canvas-wrapper:before {
    content: "";
    display: block;
    width: 300px;
    background: #0f0f0f;
    height: 300px;
    position: absolute;
    left: 0;
    right: 0;
    margin: 0 auto;
    border-radius: 100%;
    top: 0;
  }
  .vue-winwheel-component .wheel-wrapper .button-wrapper {
    margin: -150px auto 0px auto;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    width: 231px;
    height: 118px;
  }
}

// .modal-bg {
//   background-image: url("../assets/bg_modal.png");
// }
</style>

<style scoped>
.buyspin {
  background: #fd572a;
  color: #fff;
  padding: 3px 30px;
  border-radius: 50px;
}

/deep/.modal_reward .modal-dialog {
  background-image: url("../assets/bg_modal.png");
  background-size: 100%;
  background-repeat: no-repeat;
  background-position: center;
  z-index: 100000;
  position: unset;
  width: 100%;
  margin: auto;
}
/deep/.modal_before .modal-content {
  background: none;
  background-size: 100%;

  border: none;
}
/deep/.modal_reward .modal-content {
  background-color: inherit;
  border: inherit;
  color: white;
}
.bg_modal {
  background: inherit;
}

/deep/.modal_buy_fail .modal-content {
  background: url("../assets/modal-alert.png");
  background-size: 100%;
  width: 600px;
  height: 400px;
  border: none;
}
/deep/.modal_buy_success .modal-content {
  background: url("../assets/modal-purchase-confirm.png");
  background-size: 100%;
  width: 600px;
  height: 400px;
  border: none;
}
/deep/.modal_buy .modal-content {
  background: url("../assets/modal-purchase.png");
  background-size: 100%;
  width: 600px;
  height: 400px;
  border: none;
}

/deep/ .modal.show {
  display: flex !important;
  flex-direction: column;
  justify-content: center;
  align-content: center;
  align-items: flex-start;
}

@media (min-width: 768px) {
  /deep/.modal_reward .modal-dialog {
    margin: auto;
  }
  /* /deep/.modal_reward .bg_modal h3 {
    margin-top: 9vw;
  } */
  .bg_modal a {
    bottom: 10vw;
  }
  .vue-winwheel-component .wheel-wrapper .button-wrapper {
    margin: -180px auto 0px auto;
  }
}

@media (max-width: 500px) {
  /deep/.modal_buy_fail .modal-content {
    position: relative;
    display: flex;
    flex-direction: column;
    background: url("../assets/modal-alert.png");
    background-size: 100%;
    width: 300px;
    height: 240px;
  }
  /deep/.modal_buy_success .modal-content {
    position: relative;
    display: flex;
    flex-direction: column;
    background: url("../assets/modal-purchase-confirm.png");
    background-size: 100%;
    width: 300px;
    height: 240px;
  }
  /deep/.modal_buy .modal-content {
    position: relative;
    display: flex;
    flex-direction: column;
    background: url("../assets/modal-purchase.png");
    background-size: 100%;
    width: 300px;
    height: 240px;
  }
  /deep/ .modal.show {
    display: flex !important;
    flex-direction: row;
    justify-content: center;
    align-content: center;
    align-items: flex-start;
  }
}

/* Absolute Center Spinner */
.loading {
  position: fixed;
  z-index: 2000;
  height: 2em;
  width: 2em;
  overflow: show;
  margin: auto;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
}

/* Transparent Overlay */
.loading:before {
  content: "";
  display: block;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: radial-gradient(rgba(20, 20, 20, 0.8), rgba(0, 0, 0, 0.8));

  background: -webkit-radial-gradient(
    rgba(20, 20, 20, 0.8),
    rgba(0, 0, 0, 0.8)
  );
}

/* :not(:required) hides these rules from IE9 and below */
.loading:not(:required) {
  /* hide "loading..." text */
  font: 0/0 a;
  color: transparent;
  text-shadow: none;
  background-color: transparent;
  border: 0;
}

.loading:not(:required):after {
  content: "";
  display: block;
  font-size: 10px;
  width: 1em;
  height: 1em;
  margin-top: -0.5em;
  -webkit-animation: spinner 150ms infinite linear;
  -moz-animation: spinner 150ms infinite linear;
  -ms-animation: spinner 150ms infinite linear;
  -o-animation: spinner 150ms infinite linear;
  animation: spinner 150ms infinite linear;
  border-radius: 0.5em;
  -webkit-box-shadow: rgba(255, 255, 255, 0.75) 1.5em 0 0 0,
    rgba(255, 255, 255, 0.75) 1.1em 1.1em 0 0,
    rgba(255, 255, 255, 0.75) 0 1.5em 0 0,
    rgba(255, 255, 255, 0.75) -1.1em 1.1em 0 0,
    rgba(255, 255, 255, 0.75) -1.5em 0 0 0,
    rgba(255, 255, 255, 0.75) -1.1em -1.1em 0 0,
    rgba(255, 255, 255, 0.75) 0 -1.5em 0 0,
    rgba(255, 255, 255, 0.75) 1.1em -1.1em 0 0;
  box-shadow: rgba(255, 255, 255, 0.75) 1.5em 0 0 0,
    rgba(255, 255, 255, 0.75) 1.1em 1.1em 0 0,
    rgba(255, 255, 255, 0.75) 0 1.5em 0 0,
    rgba(255, 255, 255, 0.75) -1.1em 1.1em 0 0,
    rgba(255, 255, 255, 0.75) -1.5em 0 0 0,
    rgba(255, 255, 255, 0.75) -1.1em -1.1em 0 0,
    rgba(255, 255, 255, 0.75) 0 -1.5em 0 0,
    rgba(255, 255, 255, 0.75) 1.1em -1.1em 0 0;
}

/* Animation */

@-webkit-keyframes spinner {
  0% {
    -webkit-transform: rotate(0deg);
    -moz-transform: rotate(0deg);
    -ms-transform: rotate(0deg);
    -o-transform: rotate(0deg);
    transform: rotate(0deg);
  }
  100% {
    -webkit-transform: rotate(360deg);
    -moz-transform: rotate(360deg);
    -ms-transform: rotate(360deg);
    -o-transform: rotate(360deg);
    transform: rotate(360deg);
  }
}
@-moz-keyframes spinner {
  0% {
    -webkit-transform: rotate(0deg);
    -moz-transform: rotate(0deg);
    -ms-transform: rotate(0deg);
    -o-transform: rotate(0deg);
    transform: rotate(0deg);
  }
  100% {
    -webkit-transform: rotate(360deg);
    -moz-transform: rotate(360deg);
    -ms-transform: rotate(360deg);
    -o-transform: rotate(360deg);
    transform: rotate(360deg);
  }
}
@-o-keyframes spinner {
  0% {
    -webkit-transform: rotate(0deg);
    -moz-transform: rotate(0deg);
    -ms-transform: rotate(0deg);
    -o-transform: rotate(0deg);
    transform: rotate(0deg);
  }
  100% {
    -webkit-transform: rotate(360deg);
    -moz-transform: rotate(360deg);
    -ms-transform: rotate(360deg);
    -o-transform: rotate(360deg);
    transform: rotate(360deg);
  }
}
@keyframes spinner {
  0% {
    -webkit-transform: rotate(0deg);
    -moz-transform: rotate(0deg);
    -ms-transform: rotate(0deg);
    -o-transform: rotate(0deg);
    transform: rotate(0deg);
  }
  100% {
    -webkit-transform: rotate(360deg);
    -moz-transform: rotate(360deg);
    -ms-transform: rotate(360deg);
    -o-transform: rotate(360deg);
    transform: rotate(360deg);
  }
}
</style>
