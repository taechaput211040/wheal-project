<template>
  <section class="vue-winwheel">
    <div class="mobile-container">
      <!-- <h1>Vue-Winwheel</h1> -->

      <div class="wheel-wrapper">
        <div class="canvas-wrapper">
          <canvas id="canvas" width="300" height="300" v-if="isMobile">
          </canvas>
          <canvas id="canvas" width="450" height="450" v-if="!isMobile">
          </canvas>
        </div>
        <div class="button-wrapper">
          <a
            class="btn px-0 py-0"
            href="#"
            @click.prevent="startSpin()"
            v-if="!loadingPrize && !wheelSpinning && status == 0"
          >
            <img :src="require('~/assets/btn_spin.png')" alt="" />
          </a>

          <a class="btn px-0 py-0" :href="this.redirex" v-if="status == 1">
            <img :src="require('~/assets/btn_back.png')" alt="" />
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

    <div class="d-none">
        <audio controls  id="sound" >
        <source :src="require('~/assets/horn.mp3')" type="audio/mpeg">

      </audio>
      </div>
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
      modal-class="modal-bg modal_reward"
      id=""
    >
      <div class="bg_modal">
        <div class="d-block text-center">
          <h3 v-html="reward.title"></h3>
          <a href="">
            <img :src="require('~/assets/btn_back.png')" alt="" />
          </a>
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
      <div class="bg_modal ">
        <div class="d-block text-center">
          <h3 class="px-4">
            กดคลิกที่ปุ่ม SPIN! เพื่อเริ่มปั่นกงล้อ รับเครดิตฟรี 24 ชม.
          </h3>
          <!--  -->
          <b-button variant="success" 
          @click="hideModal"
          
          >Button</b-button>
          <!-- @click.prevent="playSound('http://localhost:8001/static/alberta.mp3')" -->

        </div>
      </div>
      <!-- <b-button class="mt-3" block @click="$bvModal.hide('bv-modal-example')"
        >Close Me</b-button> -->
    </b-modal>
  </section>
</template>


<script>
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
    reward: {},
    status: {
      default() {
        return 0;
      },
    },
    redirex: "",

    beforeSpin: {
      type: Function,
      required: true,
    },
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
        outterRadius: 410,
        innerRadius: 25,
        lineWidth: 8,
        drawText: true,
        animation: {},
      },
      audio: null,
      wheelImage: new Image(),
      isMobile: isMobile,
      sound_bg: new Audio("http://localhost:8001/static/alberta.mp3"),
      sound_reward: new Audio("http://localhost:8001/static/horn.mp3"),
      fontSize: 14,
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
        this.theWheel.animation.stopAngle = this.score;
        this.theWheel.startAnimation();
        // this.wheelSpinning = false;
        this.wheelSpinning = true;
        this.$axios.post(this.url, this.dataPost).then(
          (response) => {
            console.log(response);
          },
          (error) => {
            console.log(error);
          }
        );
      }
      // });
    },
    resetWheel() {
      // require("~/assets/tickticktick.mp3")
      
      this.audio = new Audio("http://localhost:8001/static/tickticktick.mp3");
      // console.log("resetWheel");
      this.theWheel = new Winwheel.Winwheel({
        ...this.WinWheelOptions,
        numSegments: this.segments.length,
        segments: this.segments,
        textFontSize: this.fontSize,
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
    },
    onFinishSpin(indicatedSegment) {
      this.audio.pause();
      this.playSoundReward();
      this.prizeName = indicatedSegment.text;
      this.showPrize();
      this.$refs["modal_reward"].show();
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
      this.sound_bg.setAttribute('crossorigin', 'anonymous'); 
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
    playSound (sound) {
      if(sound) {
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
    if (isMobile) {
      this.wheelImage.src = require("~/assets/wheel_2.png");
    } else {
      this.fontSize = 26;
      this.wheelImage.src = require("~/assets/wheel_m.png");
    }

    // console.log("status = " + this.status);

    this.initSpin();

    this.$refs["modal_before"].show();
    // this.resetWheel()
    // this.beforeSpinDefault().then((result) => {
    // setTimeout(() => this.resetWheel(), 300);
    // });
  },

  created() {},
};
</script>

<style lang="scss" scoped>
.vue-winwheel {
  text-align: center;
  /* background-image: url("../assets/wheel.png"); */
  /* background-image: url("../assets/wing.mp4"); */

  background-size: cover;
  background-position: center bottom;
  background-repeat: no-repeat;
}
.vue-winwheel h1 {
  color: #b32656;
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  font-size: 36px;
  line-height: 90px;
  letter-spacing: 4px;
  margin: 0;
}
.vue-winwheel h2 {
  margin: 0;
}
.vue-winwheel #modalSpinwheel.custom-modal .content-wrapper .content {
  width: calc(100vw - 30px);
  padding-top: 52px;
}
.vue-winwheel #modalSpinwheel.custom-modal .content-wrapper .content h2 {
  text-transform: uppercase;
  color: #b32656;
  margin-bottom: 16px;
  margin-top: 0;
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  font-size: 18px;
  letter-spacing: 1.1px;
  margin: 0;
}
.vue-winwheel #modalSpinwheel.custom-modal .content-wrapper .content p {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  font-size: 14px;
  color: black;
  text-align: center;
  line-height: 25px;
}
.vue-winwheel #modalSpinwheel.custom-modal .content-wrapper .content p strong {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
}
.vue-winwheel
  #modalSpinwheel.custom-modal
  .content-wrapper
  .content
  .modal-dismiss {
  top: 12px;
  right: 12px;
}
.vue-winwheel
  #modalSpinwheel.custom-modal
  .content-wrapper
  .content
  .modal-dismiss
  i.icon_close {
  font-size: 30px;
  color: #da2a52;
}
.vue-winwheel canvas#canvas {
  position: relative;
}
.vue-winwheel .canvas-wrapper {
  position: relative;
}
.vue-winwheel .canvas-wrapper:after {
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
.vue-winwheel .canvas-wrapper:before {
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
@media (min-width: 980px) {
  .vue-winwheel .canvas-wrapper:before {
    width: 450px;
    height: 450px;
  }
  .vue-winwheel {
    margin-top: 14vw;
  }
}

.vue-winwheel .wheel-wrapper {
  position: relative;
}
.vue-winwheel .wheel-wrapper:before {
  content: "";
  width: 62px;
  height: 47px;
  position: absolute;
  top: -10px;
  left: calc(50% - 31px);
  right: 0;
  display: block;
  z-index: 999;
  background-image: url("../assets/triangle.png");
  /* background-image: url("~/node_modules/vue-winwheel/spinner-marker.svg"); */
  background-repeat: no-repeat;
  background-size: contain;
  background-position: center;
}
.vue-winwheel .wheel-wrapper .button-wrapper {
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 231px;
  height: 118px;
}
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
.btn {
  img {
    width: 170px;
  }
}
.modal_reward {
  .bg_modal {
    h3 {
      margin-top: 30vw;
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

// .modal-bg {
//   background-image: url("../assets/bg_modal.png");
// }
</style>


<style scoped>
/deep/.modal_reward .modal-dialog {
  background-image: url("../assets/bg_modal.png");
  background-size: 100%;
  z-index: 100000;
  position: unset;
  width: 100%;
  margin: 0;
}

/deep/.modal_reward .modal-content {
  background-color: inherit;
  border: inherit;
  color: white;
  height: 100vh;
}
.bg_modal {
  background: inherit;
}

@media (min-width: 980px) {
  /deep/.modal_reward .modal-dialog {
    margin: auto;
  }
  /deep/.modal_reward .bg_modal h3 {
    margin-top: 9vw;
  }
  .bg_modal a {
    bottom: 10vw;
  }
}

 /deep/ .modal.show  {
  display:flex!important;
  flex-direction:column;
  justify-content:center;
  align-content:center;
  align-items: flex-start;
}
</style>