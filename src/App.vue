
<template>
  <v-app>

    <Toolbar :viewerState="viewerState" />

    <v-main class="text-center">

        <div class="ddd-render-back">

            <DDDMap v-if="viewerState.mapVisible" :viewerState="viewerState" />
            <DDDScene v-if="viewerState.sceneVisible" :viewerState="viewerState" />

        </div>

        <div class="ddd-front">

          <loading />

          <v-container fluid style="padding: 0px;">
              <router-view :viewerState="viewerState" @dddViewerMode="dddViewerMode" />
            <!-- <transition name="fade" mode="out-in">
            </transition> -->
          </v-container>
        </div>

    </v-main>

    <!--<Footer />-->

    <DDDServer />

  </v-app>
</template>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Rubik:ital,wght@0,300;0,400;0,500;0,600;0,700;0,800;0,900;1,300;1,400;1,500;1,600;1,700;1,800;1,900&display=swap');

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.1s;
}

.fade-enter,
.fade-leave-to {
  opacity: 0;
}

header.v-app-bar {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    z-index: 20;
}

div.ol-zoom {
    left: 5px;
    top: 10px;
}


div.ol-rotate {
    left: 5px;
    top: 70px;
    right: inherit;
}

div.ol-attribution.ol-uncollapsible {
    left: 8px;
    bottom: 5px;
    right: auto;
    font-size: 80%;
}

div.ol-scale-line {
    bottom: 35px;
}

aside.v-navigation-drawer--absolute {
    z-index: 30;
}

div.ol-control button {
    background-color: #473d54; // $primary;
}

div.ol-control button:hover, div.ol-control button:focus {
    background-color: #5e5964; // $primary;
}

.ddd-render-back {
    position: fixed;
    left: 0;
    right: 0;
    top: 40px;
    /* z-index: 0; */
}

.ddd-front {
    /*position: fixed;*/
    margin-top: 38px;
    /*top: 38px;*/
    right: 0px;
    z-index: 5;
    /*width: 400px;*/
    bottom: 0;
    max-height: 100%;
    /*height: 100%;*/
    overflow: auto;
    width: 100%;
    pointer-events: none;
}

</style>

<script>
import {createXYZ, extentFromProjection} from 'ol/tilegrid.js';
import proj4 from 'proj4';
//import {register} from 'ol/proj/proj4';
import * as olProj from 'ol/proj';
import * as extent from 'ol/extent';

import Toolbar from '@/components/core/Toolbar.vue'
import Loading from '@/components/core/Loading.vue'
//import Footer from '@/components/core/Footer.vue'

import DDDViewerAppState from '@/DDDViewerAppState';

import DDDMap from '@/components/ddd/DDDMap.vue';
import DDDScene from '@/components/ddd/DDDScene.vue';
import SceneViewMode from '@/components/scene/SceneViewMode.vue';
import DDDServer from '@/dddserver/components/DDDServer.vue';


export default {
  name: 'App',

  metaInfo() {
    return {
      title: this.appTitle,
      htmlAttrs: {
        lang: this.$i18n.locale
      },
      meta: [
        { name: 'msapplication-TileColor', content: '#ffc40d' },
        { name: 'theme-color', content: '#ffffff' },
        {
          name: 'apple-mobile-web-app-title',
          content: this.appTitle
        },
        { name: 'application-name', content: this.appTitle }
      ],
      link: [
        {
          rel: 'apple-touch-icon',
          sizes: '180x180',
          href: '/apple-icon-180x180.png'
        },
        {
          rel: 'icon',
          type: 'image/png',
          sizes: '32x32',
          href: '/favicon-32x32.png'
        },
        {
          rel: 'icon',
          type: 'image/png',
          sizes: '16x16',
          href: '/favicon-16x16.png'
        },
        { rel: 'manifest', href: '/site.webmanifest' },
        { rel: 'mask-icon', color: '#5bbad5', href: '/safari-pinned-tab.svg' },
        { rel: 'favicon', href: '/favicon.ico' }
      ]
    }
  },
  components: {
    Toolbar,
    Loading,
    //Footer
    DDDMap,
    DDDScene,
    SceneViewMode,
    DDDServer
  },
  computed: {
    appTitle() {
      return this.$store.getters.appTitle
    },
  },
  inject: [
  ],
  provide: function () {
      return {
        getSceneViewer: () => { return this.sceneViewer; },
        setSceneViewer: (sceneViewer) => { this.sceneViewer = sceneViewer; },
      }
  },
  data() {
    return {
      //name: this.$store.state.auth.user.name,
      //showVerifyDialog: !this.$store.state.verify.emailVerified
      //viewer: dddViewer,
      viewerState: this.$root.viewerAppState,
    }
  },
  props: {
    //mapVisible: Boolean,
    //sceneVisible: Boolean,
    //viewerState: Object,
  },
  beforeCreate() {
      //this._viewerState = new ViewerState();
      //this.mapVisible = true;
      //this.$set(this, 'mapVisible', true);
      //Object.freeze(this.viewerState);
  },
  created() {
    this.parseHref();
  },
  mounted() {
  },
  methods: {

      dddViewerMode(mode) {
        //console.debug("Received Viewer Mode change event to: " + mode);
        if (mode !== null) {
            this.$root.viewerAppState.mapVisible = mode === 'map';
            this.$root.viewerAppState.sceneVisible = mode === 'scene';
        }
        setTimeout(() => {
            window.dispatchEvent(new Event('resize'));
        }, 200);
      },

      dddPosition(coords, zoom) {
          //console.debug("Received Viewer coords: " + coords);
          this.viewerState.positionWGS84 = coords;
          this.viewerState.positionTileZoomLevel = zoom;
      },

      dddScenePosition(coords) {
          //console.debug("Received Viewer coords: " + coords);
          this.viewerState.positionScene = coords;
      },

      resize() {

      },

      parseHref() {
          //console.debug("Route: " + window.location.href);

          try {
                // Parse at location
                //http://localhost:8080/maps/@42.1354407,-0.4126472,17.0z
                let href = window.location.href;
                const regexp = /.*@([0-9.\-]+),([0-9.\-]+)((,(([0-9.\-]+)[ayhtz]))*).*/;
                let matches = href.match(regexp);
                const regexpTile = /.*@([0-9\-]+),([0-9\-]+)t((,(([0-9.\-]+)[ayhtz]))*).*/;
                let matchesTile = href.match(regexpTile);

                let tileCoords = null;
                //console.debug(matchesTile);

                if ((! matches || matches.length < 3) || (matchesTile)) {
                    // Try tile

                    if (matchesTile && matchesTile.length >= 3) {
                        //console.debug("Loading position from tile grid coords.");
                        matches = matchesTile;
                        tileCoords = [17, parseInt(matches[1]), parseInt(matches[2])];
                    } else {
                        // Try last
                        const dddLastPositionString = localStorage.getItem('dddLastPositionString');
                        //console.debug("Using last known position in this viewer: " + dddLastPositionString);
                        matches = dddLastPositionString.match(regexp);
                    }
                }

                if (!matches) { return; }

                if (matches.length >= 3 && tileCoords !== null) {
                    //console.debug(tileCoords);
                    this.$root.viewerAppState.positionWGS84 = this.positionFromTile(tileCoords);
                    this.$root.viewerAppState.positionTileZoomLevel = tileCoords[0];
                } else if (matches.length >= 3) {
                    this.$root.viewerAppState.positionWGS84 = [parseFloat(matches[2]),parseFloat(matches[1])];
                }

                if (matches.length >= 4) {
                    for (let match of matches[3].split(",")) {
                        if (match === "") { continue; }
                        let value = parseFloat(match.slice(0, -1));
                        let code = match.slice(-1);
                        if (code === 'z') {
                            this.$root.viewerAppState.positionTileZoomLevel = value;
                        } else if (code === 'a') {
                            this.$root.viewerAppState.positionGroundHeight = value;
                        } else if (code === 'h') {
                            this.$root.viewerAppState.positionHeading = value;
                        } else if (code === 't') {
                            this.$root.viewerAppState.positionTilt = value;
                        }
                        //console.debug(value, code);
                    }

                }
            } catch(e) {
                // eslint-disable-next-line no-console
                console.error("Error parsing location from href: " + e);
            }

          //let positionWgs84 = this.getViewerState().positionWGS84;
      },

      positionFromTile(tileCoords) {
        const tileGrid = createXYZ({
              extent: extentFromProjection('EPSG:3857'),
        });
        let tileExtent = tileGrid.getTileCoordExtent(tileCoords);
        let tileCenter = extent.getCenter(tileExtent);
        let tileCenterWGS84 = olProj.transform(tileCenter, 'EPSG:3857', 'EPSG:4326');
        return tileCenterWGS84;
      }
  }
}
</script>
