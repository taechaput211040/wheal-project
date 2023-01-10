<template>
  <section class="vue-winwheel-component">
    <div class="mobile-container">
      <!-- <h1>Vue-Winwheel</h1> -->

      <div class="wheel-wrapper">
        <div class="canvas-wrapper">
          <!-- width 320 && height 770 **** very small\old phone-->
          <canvas id="canvas" width="300" height="300" v-if="smallerThanIPAD  && verySmallPhone">
          </canvas>
          <div class="spin_button" v-if="smallerThanIPAD  && verySmallPhone">
            <img
              :src="require('~/assets/BTN-spin.png')"
              alt='spin button'
              v-if="freespin != 0"
              @click="startSpin"
            />
            <img :src="require('~/assets/BTN-non-spin.png')" style="width=100%; height:auto;" alt='spin button' v-if="freespin == 0"/>
          </div>
          <!-- 375 > width > 768-->
          <canvas id="canvas" width="320" height="320" v-if="smallerThanIPAD && !verySmallPhone">
          </canvas>
          <div class="spin_button" v-if="smallerThanIPAD && !verySmallPhone">
            <img
              :src="require('~/assets/BTN-spin.png')"
              alt='spin button'
              v-if="freespin != 0"
              @click="startSpin"
            />
            <img :src="require('~/assets/BTN-non-spin.png')" style="width=100%; height:auto;" alt='spin button' v-if="freespin == 0"/>
          </div>
          <!-- ipad or larger screen -->
          <canvas id="canvas" width="450" height="450" v-if="!smallerThanIPAD">
          </canvas>
          <div class="spin_button" v-if="!smallerThanIPAD">
            <img
              :src="require('~/assets/BTN-spin.png')"
              alt='spin button'
              v-if="freespin != 0"
              @click="startSpin"
            />
            <img :src="require('~/assets/BTN-non-spin.png')" alt='spin button' v-if="freespin == 0"/>
          </div>
          <!-- <div style="transform: translate(0px, -250px);">middle</div> /-->
        </div>
        <div class="button-wrapper">
          <div class="buyspin" v-if="freespin == 0">
            ซื้อฟรีสปิน {{bycredit_amount}} เครดิต
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
          <a
            class="btn px-0 py-0"
            href="#"
            @click.prevent="startSpin()"
            v-if="freespin == 0"
          >
            <img :src="require('~/assets/BTN-bye.png')" alt="" />
          </a>
          <a class="btn px-0 py-0" :href="this.redirex" v-if="freespin !== 0">
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
      modal-class="modal-bg modal_reward "
      id=""
    >
      <div class="bg_modal my-auto">
        <div class="d-block text-center">
          <h3>{{ this.reward_title }}</h3>
          <div class="pointer" @click="onReceive()">
            <img :src="require('~/assets/btn_reward.png')" alt="" />
          </div>
        </div>
      </div>
    </b-modal>

    <b-modal
      ref="modal_before"
      hide-footer
      hide-header
      modal-class="modal-bg modal_before modal-dialog-centered"
      id=""
    >
      <div class="bg_modal">
        <div class="d-block text-center">
          <h3 class="px-4">
            กดคลิกที่ปุ่ม SPIN! เพื่อเริ่มปั่นกงล้อ รับเครดิตฟรี 24 ชม.
          </h3>
          <!--  -->
          <b-button variant="success" @click="hideModal">OK</b-button>
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
      type: Array,
    },
    score: {
      default() {
        return 30;
      },
    },

    url: {
      default() {
        return "www.hotmail.com";
      },
    },
    dataPost: {
      default() {
        return "";
      },
    },
    reward: {
      default() {
        return "";
      },
    },
    status: {
      default() {
        return 0;
      },
    },
    redirex: "",
    freespin: {
      default() {
        return "";
      },
    },

    beforeSpin: {
      type: Function,
      required: true,
    },
    bycredit_amount: {
      tyoe: Number
    }
  },
  data() {
    return {
      loadingPrize: false,
      theWheel: null,
      modalPrize: false,
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
        animation: {},
      },
      audio: null,
      wheelImage: new Image(),
      isMobile: isMobile,
      sound_bg: new Audio("https://s3.ap-southeast-1.amazonaws.com/image-storage-betkub/luckydraw/20210829153536b82885fea83a4f81aec70eb5f97c0840.mp3"),
      sound_reward: new Audio("https://s3.ap-southeast-1.amazonaws.com/image-storage-betkub/luckydraw/20210829153634d5d55b635d7848c4b8ec0b98dacae384.mp3"),
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
      this.modalPrize = false;
    },
    beforeSpinDefault() {
      return this.beforeSpin();
    },
    async saveData() {
      const data = await this.$axios.$get(this.url);
      // console.log(data);
    },
    startSpin() {
      // this.beforeSpinDefault().then((result) => {
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
            soundTrigger: "pin",
          },
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
      console.log('in reset');
      console.log(window.screen.height);
      window.screen.width < 320 || window.screen.height < 770 ? this.verySmallPhone = true : this.verySmallPhone = false;
      window.screen.width < 768 ? this.smallerThanIPAD = true : this.smallerThanIPAD = false;
      // console.log(this.isMobile);
      // console.log(this.smallerThanIPAD);

      this.audio = new Audio("https://s3.ap-southeast-1.amazonaws.com/image-storage-betkub/luckydraw/202108291537181d53e6b561c84bb4ab33054f80ca2e27.mp3");
      // console.log("resetWheel");
      this.theWheel = new Winwheel.Winwheel({
        ...this.WinWheelOptions,
        numSegments: this.segments.length,
        segments: this.segments,
        textFontSize: this.fontSize,
        textFillStyle: "#fff",
        // drawMode: 'segmentImage',
        drawMode: "image",
        drawText: true,
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
    onReceive() {

      console.log("this.dataPost ",this.dataPost)
      this.isLoading = true;
      this.$axios
        .post(this.url, this.dataPost, {
          auth: {
            username: "ten",
            password: "3900"
          },
        })
        .then(
          (response) => {

            // console.log("this.dataPost ",this.dataPost)
            console.log("close modal")
            console.log("response ",response);
            this.isLoading = false;
            // remove array frist
            this.reward.shift();
            // this.dataPost.shift();
            this.$refs["modal_reward"].hide();
            this.total_spin = this.total_spin + 1;
            // console.log("this.freespin - this.total_spin = ",this.freespin - this.total_spin);
            if (this.freespin - this.total_spin > 0) {
              this.reward_title = this.reward[0].title;
              this.loadingPrize = true;
              this.resetWheel();
              this.loadingPrize = false;
            } else {
              location.reload();
              // this.$nuxt.refresh();
            }
          },
          (error) => {
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
    window.screen.width < 320 || window.screen.height < 770 ? this.verySmallPhone = true : this.verySmallPhone = false;
    window.screen.width < 768 ? this.smallerThanIPAD = true : this.smallerThanIPAD = false;
    if (this.smallerThanIPAD  && this.verySmallPhone) {
      this.wheelImage.src = require("~/assets/FT-wheel-mb-xs.png");
    }else if(this.smallerThanIPAD  && !this.verySmallPhone){
      this.wheelImage.src = require("~/assets/FT-wheel-mb.png");
    }
    else {
      this.fontSize = 26;
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
  },
};
</script>

<style lang="scss" scoped>
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
.vue-winwheel-component #modalSpinwheel.custom-modal .content-wrapper .content h2 {
  text-transform: uppercase;
  color: #b32656;
  margin-bottom: 16px;
  margin-top: 0;
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  font-size: 18px;
  letter-spacing: 1.1px;
  margin: 0;
}
.vue-winwheel-component #modalSpinwheel.custom-modal .content-wrapper .content p {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  font-size: 14px;
  color: black;
  text-align: center;
  line-height: 25px;
}
.vue-winwheel-component #modalSpinwheel.custom-modal .content-wrapper .content p strong {
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

.spin_button{
  transform: translate( 0px, -220px);
}

.spin_button img{
  width: 133px;
  height: auto;
}
@media (min-width: 768px) {
  .vue-winwheel-component .canvas-wrapper:before {
    width: 450px;
    height: 450px;
  }

  .spin_button{
    transform: translate( 0px, -330px);
  }

  .spin_button img{
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
.pointer {
  cursor: pointer;
  margin-top: 50%;
}

@media(max-width: 320px) {
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
@media(max-height: 770px) {
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

// .modal-bg {
//   background-image: url("../assets/bg_modal.png");
// }
</style>


<style scoped>
.buyspin{
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

/deep/.modal_reward .modal-content {
  background-color: inherit;
  border: inherit;
  color: white;
}
.bg_modal {
  background: inherit;
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

/deep/ .modal.show {
  display: flex !important;
  flex-direction: column;
  justify-content: center;
  align-content: center;
  align-items: flex-start;
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
  content: '';
  display: block;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
    background: radial-gradient(rgba(20, 20, 20,.8), rgba(0, 0, 0, .8));

  background: -webkit-radial-gradient(rgba(20, 20, 20,.8), rgba(0, 0, 0,.8));
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
  content: '';
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
  -webkit-box-shadow: rgba(255,255,255, 0.75) 1.5em 0 0 0, rgba(255,255,255, 0.75) 1.1em 1.1em 0 0, rgba(255,255,255, 0.75) 0 1.5em 0 0, rgba(255,255,255, 0.75) -1.1em 1.1em 0 0, rgba(255,255,255, 0.75) -1.5em 0 0 0, rgba(255,255,255, 0.75) -1.1em -1.1em 0 0, rgba(255,255,255, 0.75) 0 -1.5em 0 0, rgba(255,255,255, 0.75) 1.1em -1.1em 0 0;
box-shadow: rgba(255,255,255, 0.75) 1.5em 0 0 0, rgba(255,255,255, 0.75) 1.1em 1.1em 0 0, rgba(255,255,255, 0.75) 0 1.5em 0 0, rgba(255,255,255, 0.75) -1.1em 1.1em 0 0, rgba(255,255,255, 0.75) -1.5em 0 0 0, rgba(255,255,255, 0.75) -1.1em -1.1em 0 0, rgba(255,255,255, 0.75) 0 -1.5em 0 0, rgba(255,255,255, 0.75) 1.1em -1.1em 0 0;
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
