<template>
  <div id="app">
    <div class="map-content">
      <button class="show-detail-btn">Open Detail</button>

      <div class="detail-wrapper" :class="detailClass">
        <div class="item-list">
          <div class="item" v-for="i in 5" :key="i">
            <div class="img-place"></div>
            <div class="infos">
              <div class="txt-place"></div>
              <div class="txt-place w-60"></div>
              <div class="txt-place w-50"></div>
            </div>
          </div>
        </div>
      </div>
      </div>
  </div>
</template>

<script>
import Hammer from 'hammerjs';

export default {
  name: 'App',
  data() {
    return {
      detailClass: 'open-detail'
    }
  },
  mounted() {
    this.bindtap();
    this.bindPan();
  },
  methods: {
    // 绑定打开详情面板btn 的单击事件
    bindtap() {
      const manager = new Hammer.Manager(document.querySelector('.show-detail-btn'));

      manager.add(new Hammer.Tap({
        taps: 1
      }))

      manager.on('tap', () => {
        const $target = document.querySelector('.detail-wrapper');

        this.detailClass = 'open-detail';
        $target.style.transform = 'translate3d(0, 30vh, 0)';
        $target.style.webkitTransform = 'translate3d(0, 30vh, 0)';
        $target.style.mozTransform = 'translate3d(0, 30vh, 0)';
      });
    },

    bindPan() {
      const reqAnimationFrame = (function () {
        // requestAnimationFrame 执行动画的api（重点）
          return window[Hammer.prefixed(window, 'requestAnimationFrame')] || function (callback) {
              window.setTimeout(callback, 1000 / 60);
          };
      })();

      const $target = document.querySelector('.detail-wrapper');
      const manager = new Hammer.Manager($target);
      const ORIGIN_Y = this.parseTranslateValue($target).y;
      const MOVE_THRESHOLD = 30; // 自定义回弹效果阈值
      const TOP = 0, BOTTOM = $target.offsetHeight;

      console.log('ORIGIN_Y', ORIGIN_Y)
      console.log('BOTTOM', BOTTOM)
      
      let START_Y = ORIGIN_Y;
      let ticking = false;
      let translateY = 0;
      let direction_move;

      const pan = new Hammer.Pan({
        direction: Hammer.DIRECTION_VERTICAL,
        pointers: 0,
        threshold: 0
      });

      manager.add(pan);

      manager.on('panstart panmove panend pancancel', (ev) => {
        const type = ev.type;

        switch( type ) {
          case 'panstart':
            START_Y = this.parseTranslateValue($target).y;
            break;
          case 'panmove':
            translateY = START_Y + ev.deltaY;
            this.detailClass = 'panmoving';
            direction_move = ev.direction;

            if( translateY <= 1 ) {
              // 触顶
              translateY = TOP;
              this.detailClass = 'panmoving slide-up-sticky-detail';
            }else if( translateY >= BOTTOM ) {
              // 触底
              translateY = BOTTOM;
            }

            break;
          case 'panend':

            if( translateY >= ORIGIN_Y + MOVE_THRESHOLD ) {
              // console.log('下黏贴', translateY)
              if( direction_move === Hammer.DIRECTION_UP ) {
                this.detailClass = 'open-detail';
                translateY = ORIGIN_Y;
              }else {
                this.detailClass = 'slide-down-hide-detail';
                translateY = BOTTOM;
              }

            }else if( ORIGIN_Y <= translateY && translateY < ORIGIN_Y + MOVE_THRESHOLD ) {
              // console.log('中间黏贴1', translateY)
              
              this.detailClass = 'open-detail';
              translateY = ORIGIN_Y;

            }else if( ORIGIN_Y - MOVE_THRESHOLD <= translateY && translateY < ORIGIN_Y ) {
              // console.log('中间黏贴2', translateY)

              this.detailClass = 'open-detail';
              translateY = ORIGIN_Y;

            }else if( MOVE_THRESHOLD <= translateY && translateY < ORIGIN_Y - MOVE_THRESHOLD  ) {
              // console.log('上黏贴1', translateY)
              if( direction_move === Hammer.DIRECTION_DOWN ) {
                this.detailClass = 'open-detail';
                translateY = ORIGIN_Y;
              }else {
                this.detailClass = 'slide-up-sticky-detail';
                translateY = TOP;
              }

            }else if( 0 <= translateY && translateY < MOVE_THRESHOLD  ) {
              // console.log('上黏贴2', translateY)
              translateY = TOP;
              this.detailClass = 'slide-up-sticky-detail';

            }
            break;
        }

        // console.log(ev)
        requestElementUpdate();
      });

      function updateTransform() {
        const value = `translate3d(0, ${translateY}px, 0)`;

        $target.style.transform = value;
        $target.style.webkitTransform = value;
        $target.style.mozTransform = value;
        ticking = false;
      }

      function requestElementUpdate() {
        if(!ticking) {
          reqAnimationFrame(updateTransform);
          ticking = true;
        }
      }

    },

    parseTranslateValue(ele) {
      const temp = getComputedStyle(ele).transform.substring(7).split(',');

      return {
        x: Math.round( temp[4] ),
        y: Math.round( temp[5].replace(/\)?$/, '') )
      }
    }
  }
}
</script>

<style lang="scss" src="./assets/scss/main.scss"></style>