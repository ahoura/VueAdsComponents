<template>
  <!-- Composite Start -->
  <div :ref="'adsterraWrapper'+widgetID" :id="'adsterraWrapper'+widgetID">
    <div :id="'container-'+widgetID"></div>
  </div>
  <!-- Composite End -->
</template>
<script>
  /* ============
   * AdsterraNative Component
   * ============
   *
   * A basic component for Adsterra's Native unit.
   *
   */

  export default {
    /**
     * The name of the component.
     */
    name: 'ads-adsterra-native',
    mounted() {
      const refKey = `adsterraWrapper${this.widgetID}`;
      const rcds = document.getElementById(refKey);
      const rcel = document.createElement('script');
      rcel.id = `asn_${Math.floor(Math.random() * 1000)}`;
      rcel.type = 'text/javascript';
      rcel.src = `//wzdzht7am5.com/${this.widgetID}/invoke.js`;
      rcel.async = true;
      rcds.appendChild(rcel);
    },
    /**
     * The properties that the component accepts.
     */
    props: {
      widgetID: {
        type: String,
        required: true,
        // default: '104314',
      },
    },
  };
</script>
<style lang="scss">
</style>
