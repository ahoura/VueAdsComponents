<template>
  <!-- Composite Start -->
  <div :ref="'revcontentWrapper'+widgetID" :id="wrapperID"></div>
  <!-- Composite End -->
</template>
<script>
  /* ============
   * Revcontent Component
   * ============
   *
   */

  export default {
    /**
     * The name of the component.
     */
    name: 'ads-revcontent',

    /**
     * The mixins that the component can use.
     */
    mixins: [
    ],
    mounted() {
      let referer = '';
      try {
        referer = document.referrer;
        if (typeof referer === 'undefined' || referer === '') {
          throw String('undefined');
        }
      } catch (exception) {
        referer = document.location.href;
        if (referer === '' || typeof referer === 'undefined') {
          referer = document.URL;
        }
      }
      referer = referer.substr(0, 700);
      const rcds = document.getElementById(this.wrapperID);
      const rcel = document.createElement('script');
      rcel.id = `rc_${Math.floor(Math.random() * 1000)}`;
      rcel.type = 'text/javascript';
      rcel.src = `https://trends.revcontent.com/serve.js.php?w=${this.widgetID}&t=${rcel.id}&c=${(new Date()).getTime()}&width=${(window.outerWidth || document.documentElement.clientWidth)}&referer=${encodeURIComponent(referer)}`;
      rcel.async = true;
      rcds.appendChild(rcel);
    },
    /**
     * The properties that the component accepts.
     */
    props: {
      wrapperID: {
        type: String,
        required: false,
        default: 'rcjsload_d2e206',
      },
      widgetID: {
        type: String,
        required: true,
        // default: '104314',
      },
    },

    /**
     * The computed properties that the component can use.
     */
    computed: {},
  };
</script>
<style lang="scss">
</style>
