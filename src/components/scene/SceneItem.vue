<template>

    <div style="padding: 0px;" ref="dddViewPanel" >

      <v-row style="margin: 0px;">
        <v-col style="padding: 0px; pointer-events: auto;" sm="6" offset-sm="6" md="5" offset-md="7" lg="4" offset-lg="8" >

            <div style="background-color: white;">

                <v-card class="">

                    <DDDSceneInsert />

                    <v-btn style="position: absolute; z-index: 5; right: 5px; margin-top: 15px;" to="/3d" class="mx-2" fab dark x-small color="primary"><v-icon dark>mdi-close</v-icon></v-btn>

                    <v-card-title style="text-align: left; word-break: break-word; width: 95%;">{{ nodeName }}</v-card-title>

                    <div v-if="loading" style="text-align: center; margin-top: 20px;"><v-icon>mdi-loading</v-icon> <i>Loading...</i></div>

                    <OSMImage v-if="imageHref" :imageUrl="imageHref" />

                    <v-card-text v-if="showJSON" class="text-left">
                        <pre style="font-size: 80%; line-height: 100%; width: 100%; overflow-x: scroll; overflow-y: hidden;" v-html="jsonMetadataHtml"></pre>
                        <div v-if="!loading" style="text-align: right;">(metadata: ~{{ jsonMetadataHtml.length }} bytes)</div>
                        <div v-if="!loading" style="text-align: right;"><a @click="showJSON = !showJSON;">Keys View</a></div>
                    </v-card-text>

                    <v-card-text v-if="!showJSON" class="text-left">
                        <div>
                            <!-- <h3>Attributes</h3> -->

                            <v-simple-table dense>
                            <tbody>
                            <tr v-for="key in sortedMetadata" :key="key">
                                <td style="max-width: 160px; overflow: hidden; white-space: nowrap; text-overflow: ellipsis;" :title="key"><b :style="[key.indexOf('osm:') !== 0 ? {'color': 'gray'} : {}]">{{key}}</b></td>
                                <td style="white-space: nowrap;">
                                    <div v-if="key === 'osm:id'">
                                        {{ metadata[key] }}  <v-icon small>mdi-link-box-variant</v-icon> <a :href="osmLink" target="_blank">OpenStreetMap</a>
                                    </div>
                                    <div v-else-if="key === 'osm:changeset'">
                                        {{ metadata[key] }}  <v-icon small>mdi-link-box-variant</v-icon> <a :href="osmchaLink" target="_blank">OSMCha</a>
                                    </div>
                                    <div v-else-if="key === 'osm:wikidata'">
                                        {{ metadata[key] }}  <v-icon small>mdi-link-box-variant</v-icon> <a :href="wikidataLink" target="_blank">WikiData</a>
                                    </div>
                                    <div v-else-if="key === 'osm:image'">
                                        {{ metadata[key] }}  <v-icon small>mdi-link-box-variant</v-icon> <a :href="metadata[key]" target="_blank">Link</a>
                                    </div>
                                    <div v-else-if="key === 'osm:mapillary'">
                                        {{ metadata[key] }}  <v-icon small>mdi-link-box-variant</v-icon> <a :href="mapillaryLink" target="_blank">Mapillary</a>
                                    </div>
                                    <div v-else-if="metadata[key] && metadata[key].indexOf && (metadata[key].indexOf('http://') === 0 || metadata[key].indexOf('https://') === 0)" >
                                        <a :href="metadata[key]" target="_blank">{{ metadata[key] }}</a>
                                    </div>
                                    <div v-else>
                                        {{ metadata[key] }}
                                    </div>
                                </td>
                            </tr>
                            </tbody>
                            </v-simple-table>

                            <div v-if="!loading" style="text-align: right;">(metadata: {{ jsonMetadataHtml.length }} bytes)</div>
                            <div v-if="!loading" style="text-align: right;"><a @click="showJSON = !showJSON;">JSON View</a></div>
                        </div>
                    </v-card-text>

                    <v-card-text v-if="!loading" class="text-left">
                        <div>
                            <h3>Links</h3>
                            <div><a v-if="osmLink" :href="osmLink" target="_blank">OpenStreetMap Object</a></div>
                            <div><a v-if="osmchaLink" :href="osmchaLink" target="_blank">OSMCha (Change Analyzer)</a></div>
                            <div><a v-if="mapillaryLink" :href="mapillaryLink" target="_blank">Mapillary Picture</a></div>
                            <div v-if="viewerState.dddConfig.showDevelLinks"><a :href="sceneLinkGoogleMaps" target="_blank">Google Maps View</a></div>
                        </div>
                    </v-card-text>

                    <v-card-text v-if="!loading && viewerState.sceneViewModeShow" class="text-left">
                        <h3>View</h3>
                        <v-btn @click="selectCameraOrbit" class="mx-2" dark color="primary"><v-icon dark>mdi-rotate-orbit</v-icon> Orbit Item</v-btn>
                        <v-btn @click="selectCameraFree" class="mx-2" dark color="primary"><v-icon dark>mdi-axis-arrow</v-icon> Free</v-btn>
                        <!-- <v-btn @click="selectCameraWalk" class="mx-2" dark color="primary"><v-icon dark>mdi-walk</v-icon> Walk</v-btn> -->
                    </v-card-text>


                    <v-card-text v-if="!loading" class="text-left">
                        <div>
                            <h3>Node Tree</h3>
                            <!--<NodeHierarchy :nodeGetter="nodeGetter" :viewerState="viewerState" depth="1"></NodeHierarchy>-->
                            <NodeHierarchy :objectRef="targetObjectRef" :viewerState="viewerState" depth="1"></NodeHierarchy>
                        </div>
                    </v-card-text>

                    <v-card-text v-if="!loading" class="text-left">
                        <h3>Node Actions</h3>
                        <v-btn @click="removeNode" class="mx-2" dark color="primary"><v-icon dark>mdi-delete</v-icon> Remove Node</v-btn>
                    </v-card-text>

                </v-card>

            </div>

        </v-col>
    </v-row>

  </div>

</template>

<style>
tbody tr:nth-of-type(odd) {
   background-color: rgba(0, 0, 0, .05);
}
/*
.v-card__subtitle, .v-card__text, .v-card__title {
    padding: 2px;
}
*/

.v-treeview-node__content, .v-treeview-node__label {
  flex-shrink: 1;
  white-space: normal;
}
.v-treeview-node__root {
  height: auto;
}
</style>


<script>
import DDDScene from '@/components/ddd/DDDScene.vue';
import DDDSceneInsert from '@/components/ddd/DDDSceneInsert.vue';
import OSMImage from '@/components/ddd/OSMImage.vue';
import NodeHierarchy from '@/components/scene/NodeHierarchy.vue';
import { DDDObjectRef } from 'ddd-viewer';

export default {
  mounted() {

    this.$emit('dddViewerMode', 'scene');

    this.metadata = {};

    if (this.getSceneViewer()) {

        //this.setMesh(this.getSceneViewer().selectedMesh);
        this.setTargetObject(this.getSceneViewer().selectedObject);

        /*
        if (! (this.getSceneViewer().selectedMesh)) {
            let urlNodeId = this.$route.params.id;
            //this.getSceneViewer().viewerState.sceneSelectedMeshId = urlNodeId;
            this.getSceneViewer().selectMeshById(urlNodeId);
        }
        */

    }

    window.addEventListener('resize', this.resize);
    this.resize();

  },

  beforeDestroy() {
    this.getSceneViewer().deselectMesh();
  },

  metaInfo() {
    return {
      title: this.$store.getters.appTitle,
      titleTemplate: `${this.$t('sceneItem.TITLE')} - %s`
    }
  },
  properties: [
      'viewerState',
  ],
  inject: [
    'getSceneViewer',
  ],
  data() {
    return {
      loading: true,

      nodeId: null,  // currently this is the nodeId in the URL (last part with # replaced)
      targetObjectRef: null,
      //nodeGetter: () => { return (this.viewerState.sceneSelectedMeshId ? this.getSceneViewer().selectedMesh : null); },
      nodeName: null,
      metadata: {},

      showJSON: false,
    }
  },
  computed: {
    jsonMetadataHtml: function () {
      this.viewerState.sceneSelectedMeshId;
      this.$route;  // force dependency on property

      //relation-12685963
      let result = JSON.stringify(this.metadata, null, 2);
      result = result.replaceAll("<", "&lt;");
      result = result.replaceAll(">", "&gt;");
      result = this.parseLinkOSMObjects(result);
      return result;
    },
    sortedMetadata: function() {
      this.viewerState.sceneSelectedMeshId;
      this.$route;  // force dependency on property
      let keys = Object.keys(this.metadata);
      keys = keys.filter((key) => { return ! (key.indexOf('_') === 0); });
      //keys = keys.filter((key) => { return (key.indexOf('osm:') === 0); });
      keys.sort();
      keys.sort((a, b) => { return ((b.indexOf('osm:') === 0 ? 1 : 0) - (a.indexOf('osm:') === 0 ? 1 : 0));});
      return keys; // Do your custom sorting here
    },
    sceneLinkGoogleMaps: function() {
        this.$route;  // force dependency on property
        this.viewerState.sceneSelectedMeshId;
        let url = null;
        if (this.getSceneViewer()) {
            url = 'https://www.google.com/maps/' +  this.getSceneViewer().positionString() + '/data=!3m1!1e3';  // ?hl=es-ES
        }
        return url;
    },
    imageHref: function() {
        this.$route;  // force dependency on property
        this.viewerState.sceneSelectedMeshId;
        let url = null;
        if (this.metadata['osm:image']) {
            url = this.metadata['osm:image'];
        } else if (this.metadata['osm:mapillary']) {
            let code = this.mapillaryCode(this.metadata['osm:mapillary']);
            url = "https://images.mapillary.com/" + code + "/thumb-1024.jpg";
        }
        return url;
    },
    osmchaLink: function() {
        this.$route;  // force dependency on property
        this.viewerState.sceneSelectedMeshId;
        let url = null;
        if (this.metadata['osm:changeset']) {
            url = 'https://osmcha.org/changesets/' + this.metadata['osm:changeset'] + '/';
        }
        return url;
    },
    osmLink: function() {
        this.$route;  // force dependency on property
        this.viewerState.sceneSelectedMeshId;
        let url = null;
        if (this.metadata['osm:id']) {
            let element = this.metadata['osm:element'];
            let id = this.metadata['osm:id'].split("-")[1];
            url = 'https://www.openstreetmap.org/' + element + '/' + id;
        }
        return url;
    },
    wikidataLink: function() {
        this.$route;  // force dependency on property
        this.viewerState.sceneSelectedMeshId;
        let url = null;
        if (this.metadata['osm:wikidata']) {
            let wikidata = this.metadata['osm:wikidata'];
            url = 'https://www.wikidata.org/wiki/' + wikidata;
        }
        return url;
    },
    mapillaryLink: function() {
        this.$route;  // force dependency on property
        this.viewerState.sceneSelectedMeshId;
        let url = null;
        // Ex: https://www.mapillary.com/app/?lat=42.23703499999999&lng=-8.726672000000008&z=17&pKey=L_VNKSomHYS2OemeuQ4h6g

        if (this.metadata['osm:mapillary']) {
            let code = this.mapillaryCode(this.metadata['osm:mapillary']);
            url = "https://www.mapillary.com/app/?lat=" + this.viewerState.positionWGS84[0].toFixed(14) + "&lng=" + this.viewerState.positionWGS84[1].toFixed(14) + "&z=17&pKey=" + code;
        }
        return url;
    },
    f4mapLink: function() {
        // https://demo.f4map.com/#lat=47.4312176&lon=19.0443469&zoom=18&camera.theta=76.276&camera.phi=99.351
        return null;
    }

  },
  props: [
      'viewerState',
  ],
  watch: {
    '$route' () {
        let urlNodeId = this.$route.params.id;
        if (DDDObjectRef.urlId(urlNodeId) !== this.nodeId) {
            //this.loading = true;
            let objectRef = this.getSceneViewer().findObjectById(urlNodeId);
            //console.debug("Found object: ", urlNodeId, objectRef)

            if (objectRef) {
                const highlight = ! this.getSceneViewer().sequencer.playing;  // avoids highlighting during sequence playback (TODO: make this an option not hardcoded here)
                this.getSceneViewer().selectObject(objectRef, highlight);
                this.setTargetObject(objectRef);
            } else {
                this.setTargetObject(null);
            }

        }
    },
    'viewerState.sceneSelectedMeshId' () {
        this.$forceUpdate();
        //this.setMesh(this.getSceneViewer().selectedMesh);
        //if (! this.metadata['_updated']) {this.metadata['_updated'] = 0;}
        //this.metadata['_updated']++;
    }
  },

  components: {
    DDDScene,
    DDDSceneInsert,
    OSMImage,
    NodeHierarchy,
  },

    methods: {

        parseLinkOSMObjects(val) {
            const relationRegexp = /relation-(\d+)/g;
            val = val.replaceAll(relationRegexp, '<a href="https://www.openstreetmap.org/relation/$1" target="_blank">relation-$1</a>');
            const wayRegexp = /way-(\d+)/g;
            val = val.replaceAll(wayRegexp, '<a href="https://www.openstreetmap.org/way/$1" target="_blank">way-$1</a>');
            const nodeRegexp = /node-(\d+)/g;
            val = val.replaceAll(nodeRegexp, '<a href="https://www.openstreetmap.org/node/$1" target="_blank">node-$1</a>');
            return val;
        },

        /*
        setNodeId(nodeId) {
            if (!nodeId) return;
            this.nodeId = nodeId;
            this.loading = false;

            this.metadata = this.getSceneViewer().nodeMetadata(nodeId);

            this.nodeName =
        }
        */

      /*
      setMesh(mesh, subindex=null) {
          //this.mesh = mesh;
          if (!mesh) { return; }
          this.nodeId = this.getSceneViewer().viewerState.sceneSelectedMeshId.split("/").pop().replaceAll('#', '_');
          this.loading = false;

          this.nodeName = mesh.id.split("/").pop().replaceAll("_", " ");
          if (mesh.metadata && mesh.metadata.gltf && mesh.metadata.gltf.extras) {
              this.metadata = mesh.metadata.gltf.extras;
              if (this.metadata['osm:name']) {
                  this.nodeName = this.metadata['osm:name'];
              }
          }
          this.nodeGetter = () => { return this.getSceneViewer().selectedMesh; };
          //console.debug("Scene Item setMesh called.");

          this.resize();
          setTimeout(() => { window.dispatchEvent(new Event('resize')) }, 200);

      },
      */

        setTargetObject(targetObjectRef) {
            //console.debug("Setting target object: ", targetObject);

            if (!targetObjectRef) { return; }

            this.nodeId = targetObjectRef.getUrlId();
            this.nodeName = decodeURIComponent(targetObjectRef.getUrlId());
            this.loading = false;

            this.metadata = targetObjectRef.getMetadata() || {};
            if (this.metadata && 'osm:name' in this.metadata) {
                this.nodeName = this.metadata['osm:name'];
            } else if (this.metadata && 'ddd:rpath' in this.metadata) {
                this.nodeName = this.metadata['ddd:rpath'].split("/").slice(-1)[0];
            }

            //this.nodeGetter = () => { return targetObject; };
            Object.freeze(targetObjectRef);  // This is CRITICAL to avoid Vue proxying every node and Babylon object
            this.targetObjectRef = targetObjectRef;

            // Force redraw, so canvases are resized according to right pane size and scrollbar
            this.resize();
            setTimeout(() => { window.dispatchEvent(new Event('resize')) }, 200);
        },

      resize() {
        let el = this.$el.querySelector('.v-card');
        //this.$el.style.height = '' + (window.innerHeight - 40) + 'px';
        el.style.minHeight = '' + (window.innerHeight - 38) + 'px';
      },

      mapillaryCode(value) {
        // Sometimes seen: https://www.mapillary.com/map/im/eiHLqwNJWAgjrp1tEDduUA
          if (!value) { return null; }
          let code = value;
          //console.debug(code);
          if (value.indexOf("/map/im/") > 0) {
              const regex = /^.*\/map\/im\/([^\/?]+).*$/gm;
              const m = regex.exec(value);
              //console.debug(m);
              code = m[1];
          }
          return code;
      },

      selectCameraOrbit() {
          this.getSceneViewer().selectCameraOrbit();
      },
      selectCameraFree() {
          this.getSceneViewer().selectCameraFree();
      },
      selectCameraWalk() {
          this.getSceneViewer().selectCameraWalk();
      },

      removeNode() {
         //this.getViewerState().sceneViewer.selectMesh(node);
         let objectRef = this.getSceneViewer().selectedObject;
         this.getSceneViewer().deselectObject();

         let mesh = objectRef.mesh;
         mesh.setParent(null);
         mesh.dispose();
      }

  },

}
</script>

