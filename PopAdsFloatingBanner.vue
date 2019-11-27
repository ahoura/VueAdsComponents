<template>
  <!-- Composite Start -->
  <div v-if="visible" :ref="'fbwrapper'" id="c_window_xEucqIjg" class="c_window_xEucqIjg" :style="mainWrapperSize" :class="addPositionClass">
    <div style="height:30px;">
      <span id="a_title_nEYjMupI">
        Ads by <a :href="adby.url" target="_blank">{{adby.name}}</a>
      </span>
      <span v-if="!adtimeout" @click.prevent="dismissAd" class="a_close_nEYjMupI">
        <a href="#" @click.prevent="dismissAd" data-alink="data-alink" style="text-decoration: none!important; color: rgba(0,0,0,0.3);">
          &times;
        </a>
      </span>
      <span v-else id="a_timer_oYvwGmQc">{{adtimeout}}</span>
    </div>
    <div @click.prevent="handleOverlayClick" id="alink_overlay_EPXdyaUf" alink="alink"></div>
    <iframe
      v-if="adurl"
      :src="adurl"
      name="a_iframe_DwTGCjTm"
      id="a_iframe_DwTGCjTm"
      frameBorder="0"
      scrolling="no"
      sandbox="allow-forms allow-scripts"></iframe>
  </div>
  <!-- Composite End -->
</template>
<script>

import detectZoom from 'detect-zoom';


  export default {
    /**
     * The name of the component.
     */
    name: 'ads-popads-floating-banner',

    /**
     * The mixins that the component can use.
     */
    mixins: [],
    mounted() {
      this.start();
    },
    beforeDestroy() {
      clearInterval(this.initialDocCheck);
      clearInterval(this.checkInventoryInterval);
      clearTimeout(this.libLoadTimeout);
      clearInterval(this.adtimer);
      clearTimeout(this.delayedFallbackChecker);
    },
    data() {
      return {
        visible: false,
        initialDocCheck: 0,
        libLoadTimeout: 0,
        checkInventoryInterval: 0,
        adby: {
          name: 'PopAds.net',
          url: 'https://www.popads.net',
        },
        adposition: null,
        adurl: null,
        adclickurl: null,
        adtimeout: 5,
        adtimer: null,
        delayedFallbackChecker: null,
        mainTop: null,
        floatingBannerCalled: false,
        frameSize: {
          width: 0,
          height: 0,
        },
        mainWrapperSize: {
          width: 0,
          height: 0,
        },
        overlaySize: {
          width: 0,
          height: 0,
        },
        titleSize: {
          width: 0,
          height: 0,
        },
      };
    },
    /**
     * The properties that the component accepts.
     */
    props: {
      siteId: {
        type: Number,
        required: true,
      },
      minBid: {
        type: Number,
        required: false,
        default: 0,
      },
      popundersPerIP: {
        type: Number,
        required: false,
        default: 0,
      },
      delayBetween: {
        type: Number,
        required: false,
        default: 0,
      },
      default: {
        type: [String, Boolean],
        required: false,
        default: false,
      },
      defaultPerDay: {
        type: Number,
        required: false,
        default: 0,
      },
      topmostLayer: {
        type: Boolean,
        required: false,
        default: false,
      },
      blockedCountries: {
        type: String,
        required: false,
        default: '',
      },
    },
    methods: {
      adscoreDeploy() {
        const doc = window.document;
        this.initialDocCheck = setInterval(() => {
          if (doc.body) {
            clearInterval(this.initialDocCheck);
            const g = doc.createElement('script');
            g.src = '//c.adsco.re/';
            try {
              g.onerror = () => {
                if (g.src === '//c.adsco.re/') {
                  g.src = '//162.252.214.5/a.js';
                } else {
                  clearTimeout(this.libLoadTimeout);
                  this.checkInventory('e1');
                }
              };
            } catch (err) {
              // nada
            }
            g.onload = () => {
              clearTimeout(this.libLoadTimeout);
              try {
                window.AdscoreInit('QpUJAAAAAAAAGu98Hdz1l_lcSZ2rY60Ajjk9U1c', {
                  sub_id: this.siteId,
                  callback: (b) => {
                    this.checkInventory(b.signature || `e2 ${b.error}`);
                  },
                });
              } catch (err) {
                this.checkInventory(`e4 ${err.message}`);
              }
            };
            document.body.appendChild(g);
            this.libLoadTimeout = setTimeout(() => {
              this.checkInventory('e3');
            }, 6E3);
          }
        }, 100);
      },
      getScreenData() {
        try {
          const b = window.screen;
          const c = detectZoom.zoom();
          return [b.width, b.height, c, b.width * c, b.height * c].join();
        } catch (err) {
          return '';
        }
      },
      checkInventory(hash) {
        this.checkInventoryInterval = setInterval(() => {
          let endpoint = '//serve.popads.net/c';
          if (document.body) {
            clearInterval(this.checkInventoryInterval);
            const params = {
              _: encodeURIComponent(hash),
              v: 4,
              siteId: this.siteId,
              minBid: this.minBid,
              popundersPerIP: this.popundersPerIP,
              blockedCountries: this.blockedCountries,
              documentRef: encodeURIComponent(document.referrer),
              s: this.getScreenData(),
            };
            const paramkeys = Object.keys(params);
            paramkeys.forEach((l) => {
              endpoint += `${(endpoint.indexOf('?') === -1 ? '?' : '&')}${l}=${(params[l] || '')}`;
            });
            const r = document.createElement('script');
            r.src = endpoint;
            r.onerror = () => {
              // window.alert('err');
              // this.runFallback();
            };
            r.onload = () => {
              this.delayedFallbackChecker = setTimeout(() => {
                this.runFallback();
              }, 1000);
            };
            document.body.appendChild(r);
          }
        }, 100);
      },
      runFallback() {
        // fallback check
        if (!this.floatingBannerCalled) {
          const fallbackAd = {
            url: 'https://videosupers.pro/download?h=waWQiOjEwMDI1MDMsInNpZCI6MTAwNzIyNywid2lkIjo5ODM5LCJzcmMiOjJ9eyJ&bbr=1&si1=&si2=',
            position: 'bottom-left',
            width: '400px',
            height: '40%',
            clickurl: 'https://videosupers.pro/download?h=waWQiOjEwMDI1MDMsInNpZCI6MTAwNzIyNywid2lkIjo5ODM5LCJzcmMiOjJ9eyJ&bbr=1&si1=&si2=',
            by: {
              name: 'Evadav',
              url: 'http://refadav.com/?ref=DWi9g8eCp74rEXYVRDhLC5yRlmUV5sHw',
            },
            countdown: 0,
          };

          const chance = Math.random();
          if (chance < 0.5) {
            fallbackAd.url = 'https://videosupers.pro/play?h=waWQiOjEwMDI1MDMsInNpZCI6MTAwNzIyNywid2lkIjo5ODM2LCJzcmMiOjJ9eyJ&si1=&si2=';
            fallbackAd.clickurl = 'https://videosupers.pro/play?h=waWQiOjEwMDI1MDMsInNpZCI6MTAwNzIyNywid2lkIjo5ODM2LCJzcmMiOjJ9eyJ&si1=&si2=';
          }
          this.parseFloatingBanner(fallbackAd);
        }
      },
      // a.url, a.position, a.width, a.height, a.clickurl
      enableFloatingBanner(url, position, width, height, clickurl, by, countdown = 5) {
        this.adposition = position;
        this.adby.name = by.name;
        this.adby.url = by.url;
        // check width and height
        const isWidthPx = width.search(/px/i);
        const isHeightPx = height.search(/px/i);
        const x = (Math.max(document.documentElement.clientWidth, window.innerWidth || 0),
          Math.max(document.documentElement.clientHeight, window.innerHeight || 0));

        this.mainWrapperSize.width = width;
        if (isWidthPx === -1) {
          this.overlaySize.width = '100%';
          this.frameSize.width = '100%';
          this.titleSize.width = '100%';
        } else {
          this.overlaySize.width = width;
          this.frameSize.width = width;
          this.titleSize.width = width;
        }

        if (isHeightPx === -1) {
          this.frameSize.height = '100%';
          this.overlaySize.height = '96%';
          this.mainWrapperSize.height = height;
        } else {
          const heightInt = height.split('px')[0];
          const heightMax = Number(heightInt) + 30;
          this.overlaySize.height = height;
          this.frameSize.height = height;
          this.mainWrapperSize.height = `${heightMax}px`;

          const widthInt = width.split('px')[0];
          const widthMax = Number(widthInt) + 40;

          const n = a => a.matches;
          const k = () => {
            // Math.max(document.documentElement.clientWidth, window.innerWidth || 0);
            if (Math.max(document.documentElement.clientHeight, window.innerHeight
              || 0) < heightMax) {
              this.mainTop = '0';
            }
          };

          window.onresize = k;
          const h = document.createElement('style');
          h.innerHTML = `@media all and (max-width: ${widthMax}px){#c_window_xEucqIjg{position:fixed;top:0!important;left:0;right:0;width:90%!important;margin:10px auto auto!important;text-align:center}.bottom-center_vUTDnibMkZJIvuTH,.top-center_vUTDnibMkZJIvuTH{left:0!important;right:0!important;transform:none!important}#a_iframe_DwTGCjTm{width:100%!important;}#alink_overlay_EPXdyaUf{width:90%!important;height:90%!important},.bottom-right_vUTDnibMkZJIvuTH {top:0px!important;} }`;
          document.head.appendChild(h);
          if (matchMedia) {
            const b = window.matchMedia(`(min-width: ${widthInt}px)`);
            b.addListener(n);
            n(b);
          }
          if (window.matchMedia('(orientation: landscape)').matches && x < heightMax) {
            document.getElementById('c_window_xEucqIjg').style.top = '0';
          }
          window.addEventListener('orientationchange', () => {
            if (window.matchMedia('(orientation:landscape)').matches || x < heightMax) {
              document.getElementById('c_window_xEucqIjg').style.top = '0';
            }
          });
        }
        this.visible = true;
        this.$nextTick(() => {
          this.$popunder.ignoreTo(this.$refs.fbwrapper);
        });
        this.adurl = url;
        this.adclickurl = clickurl;
        this.adtimeout = countdown;
        if (this.adtimeout) {
          this.adtimer = setInterval(() => {
            this.adtimeout -= 1;
            if (this.adtimeout <= 0) {
              clearInterval(this.adtimer);
            }
          }, 1000);
        }
      },
      dismissAd() {
        this.visible = false;
        // event.preventDefault();
      },
      handleOverlayClick(/* event */) {
        // const target = (event || window.event).target || a.srcElement;
        const win = window.open(this.adclickurl, '_blank');
        if (win) {
          win.focus();
        }
      },
      parseFloatingBanner(ad) {
        this.enableFloatingBanner(ad.url,
          ad.position,
          ad.width,
          ad.height,
          ad.clickurl,
          ad.by,
          ad.countdown,
        );
      },
      monitorWindow() {
        // nada yet since we dont use popup
      },
      setupJSONP() {
        // eslint-disable-next-line no-underscore-dangle,
        window._pao = {
          parse: () => {
            // z._parseInventory(a)
            // do nothing with pops
          },
          fbparse: (ad) => {
            this.floatingBannerCalled = true;
            ad.by = {
              name: 'PopAds.net',
              url: 'https://www.popads.net',
            };
            this.parseFloatingBanner(ad);
          },
        };
      },
      start() {
        // this.$popunder.ignoreTo(this.adHolder);
        this.setupJSONP();
        this.adscoreDeploy();
        this.monitorWindow();
      },
    },

    /**
     * The computed properties that the component can use.
     */
    computed: {
      // widgetElmId() {
      //   return `rn_ad_native_${Math.random().toString(36).substring(7)}`;
      // },
      addPositionClass() {
        return `${this.adposition}_vUTDnibMkZJIvuTH`;
      },
      adStyle() {
        return (this.mainTop) ? `top: ${this.mainTop}; ` : '';
      },
    },
  };
</script>
<style lang="scss" scoped>
  #a_timer_oYvwGmQc,
  #a_title_nEYjMupI,
  .a_close_nEYjMupI{
      top:0;
      right:0;
      height:30px;
      line-height:30px;
      text-align:center
  }
  .top-left_vUTDnibMkZJIvuTH{
      position:fixed;
      top:0;
      left:0
  }
  .bottom-left_vUTDnibMkZJIvuTH{
      position:fixed;
      bottom:0;
      left:0
  }
  .top-right_vUTDnibMkZJIvuTH{
      position:fixed;
      top:0;
      right:0
  }
  .bottom-right_vUTDnibMkZJIvuTH{
      position:fixed;
      bottom:0;
      right:0
  }
  .top-center_vUTDnibMkZJIvuTH{
      position:fixed;
      top:0;
      left:50%;
      transform:translateX(-50%)
  }
  .bottom-center_vUTDnibMkZJIvuTH{
      position:fixed;
      bottom:0;
      left:50%;
      transform:translateX(-50%)
  }
  .c_window_xEucqIjg{
      display: block;
      z-index:9999999;
      overflow:hidden;
      position:fixed;
      background-color:#FFF;
      margin:20px;
      padding:0;
      // border:1px solid #ccc;
      // border-radius:5px;
      -webkit-box-shadow:0 0 5px 1px rgba(153,153,153,.5);
      -moz-box-shadow:0 0 5px 1px rgba(153,153,153,.5);
      box-shadow:0 0 5px 1px rgba(153,153,153,.5);
      max-width: 60vw;
      max-height: 50vh;
  }
  #alink_overlay_EPXdyaUf{
      width: 100%;
      height: 100%;
      position:absolute;
      z-index:1;
      background:rgba(0,0,0,0);
      cursor:pointer
  }
  #a_iframe_DwTGCjTm{
    width: 100%;
    height: 100%;
    z-index:-1;
    padding:0!important;
  }
  .a_close_nEYjMupI{
      position:absolute;
      color:rgba(0,0,0,.3);
      width:30px;
      font-size:30px
  }
  #a_title_nEYjMupI{
      position:absolute;
      width: 100%;
      font-size: 1rem;
      text-align: left;
      padding-left: 0.5rem;
      color:rgba(0,0,0,1);
      // font-size:18px
  }
  .a_close_nEYjMupI a{
      text-decoration:none!important
  }
  #a_timer_oYvwGmQc{
      position:absolute;
      color:rgba(0,0,0,.3);
      width:30px;
      font-size:30px
  }
  .a_close_nEYjMupI:focus,.a_close_nEYjMupI:hover{
      color:#000;
      cursor:pointer
  }
  .a_hide_qkasklrO{
      display:none
  }
</style>
