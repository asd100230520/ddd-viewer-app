<template>

    <div class="ddd-view-mode-selector unselectable" style="text-align: right;">

        <div style="margin-top: 4px;">
            <small style="color: white;"><v-icon small dark>mdi-white-balance-sunny</v-icon> {{ positionDateFormatted }}</small>
        </div>

        <div v-if="dddConfig.geolocation" style="margin-top: 4px;">
            <v-btn @click="switchGeolocationPosition" class="" :color="viewerState.geolocationEnabled ? 'success': 'primary'"><small><v-icon dark>mdi-crosshairs-gps</v-icon> GPS</small></v-btn>
        </div>

        <div style="margin-top: 4px;">
            <v-btn @click="selectCameraOrbit" class="" dark color="primary"><small><v-icon dark>mdi-rotate-orbit</v-icon> Orbit</small></v-btn>
        </div>
        <div style="margin-top: 4px;">
            <v-btn @click="selectCameraFree" class="" dark color="primary"><small><v-icon dark>mdi-axis-arrow</v-icon> Free</small></v-btn>
        </div>
        <div style="margin-top: 4px;">
            <v-btn @click="selectCameraWalk" class="" dark color="primary"><small><v-icon dark>mdi-walk</v-icon> Walk</small></v-btn>
        </div>

        <div style="margin-top: 8px;">
            <v-btn @click="cycleMoveSpeed" class="" dark color="info"><small><v-icon dark>mdi-fast</v-icon> Move Speed {{ viewerState.sceneMoveSpeed }}</small></v-btn>
        </div>

    </div>

</template>

<style>
.ddd-view-mode-selector {
    position: absolute;
    /*z-index: 2;*/
    bottom: 10px;
    right: 10px;
    /*//opacity: .75;*/
    cursor: pointer;
}
.ddd-view-mode-selector button {
    pointer-events: auto;
}
*.unselectable {
   -moz-user-select: none;
   -khtml-user-select: none;
   -webkit-user-select: none;

   /*
     Introduced in Internet Explorer 10.
     See http://ie.microsoft.com/testdrive/HTML5/msUserSelect/
   */
   -ms-user-select: none;
   user-select: none;
}
</style>

<script>
import 'ol/ol.css';
import * as olProj from 'ol/proj';
import Map from 'ol/Map';
import OSM from 'ol/source/OSM';
import TileDebug from 'ol/source/TileDebug';
import TileLayer from 'ol/layer/Tile';
import View from 'ol/View';
import {createXYZ, extentFromProjection} from 'ol/tilegrid.js';
import {ScaleLine, defaults as defaultControls} from 'ol/control';

export default {

  metaInfo() {
    return {
      title: this.$store.getters.appTitle,
      titleTemplate: `${this.$t('home.TITLE')} - %s`
    }
  },

  data() {
    return {
    }
  },

  props: [
      'viewerState',
  ],
  inject: [
    'getSceneViewer',
  ],
  components: {
  },

  computed: {
      positionDateFormatted() {
          this.viewerState.positionDateSeconds; // Force update
          let hours = this.viewerState.positionDate.getHours();
          let minutes = this.viewerState.positionDate.getMinutes();
          return hours + ":" + ((minutes < 10) ? "0" : "") + minutes;
      }

  },

  methods: {

      selectCameraOrbit() {
          this.$root.dddViewerComponent.selectCameraOrbit();
      },

      selectCameraFree() {
          this.$root.dddViewerComponent.selectCameraFree();
      },

      selectCameraWalk() {
          this.$root.dddViewerComponent.selectCameraWalk();
      },

      switchGeolocationPosition() {
            this.getSceneViewer().geolocationPosition(!this.viewerState.geolocationEnabled);
      },

      cycleMoveSpeed() {
          this.$emit('cycleMoveSpeed');
      }

  },

}

</script>
