/* eslint no-use-before-define: 0 */
<template>
  <div>
    <!-- table locations -->
    <table style="margin: 10px 0; padding: 2px">
      <thead>
        <tr>
          <th>Name</th>
          <th>Latitude</th>
          <th>Longitude</th>
          <th>InfoWindow</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="marker in markers"
          :key="marker.id"
          @click="clickTableRow(marker)"
        >
          <td>{{ marker.title }}</td>
          <td>{{ marker.position.lat }}</td>
          <td>{{ marker.position.lng }}</td>
          <td>{{ marker.content }}</td>
        </tr>
      </tbody>
    </table>

    <!-- create button -->
    <v-btn @click="enableAddMarker = !enableAddMarker"> Create </v-btn>
    <v-btn
      @click="
        (markerVisible = !markerVisible) && (animation = (animation + 1) % 3)
      "
    >
      Show/Hide Marker
    </v-btn>
    <div>
      <br />
    </div>
    <!-- modal -->
    <v-row justify="center" data-app>
      <v-dialog v-model="dialog" persistent max-width="600px">
        <v-card>
          <v-card-title>
            <span class="headline">Location information</span>
          </v-card-title>
          <v-card-text>
            <v-container>
              <v-row>
                <v-col cols="12">
                  <v-text-field
                    v-model="addressOfMarker"
                    label="Địa chỉ*"
                    required
                  ></v-text-field>
                  <v-text-field
                    v-model="utilityType"
                    label="Loại tiện ích"
                    placeholder="parking, library, info, restaurant, inn, school"
                    required
                  ></v-text-field>
                  <v-text-field
                    v-model="infoWindowContent"
                    label="InfoWindow"
                    placeholder="<h1>ohaha</h1>"
                    required
                  ></v-text-field>
                </v-col>
              </v-row>
            </v-container>
            <small>*ám chỉ bắt cmn buộc</small>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" text @click="dialog = false">
              Close
            </v-btn>
            <v-btn color="blue darken-1" text @click="saveMarker"> Save </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-row>
    <hr />
    <div>
      <gmap-map
        :center="currentCenter"
        :zoom="15"
        style="width: auto; height: 400px; border: 3px solid green"
        @click="createNewMarker"
      >
        <gmap-info-window
          :options="infoWindowOptions"
          :position="infoWindowPosition"
          :opened="infoWindowOpen"
          @closeclick="closeInfoWindow"
        />
        <gmap-marker
          :visible="markerVisible"
          :animation="animation"
          v-for="marker in markers"
          :key="marker.id"
          :position="marker.position"
          :icon="marker.icon"
          :clickable="true"
          :title="marker.title"
          @click="toggleInfoWindow(marker)"
        />
      </gmap-map> 
    </div>
  </div>
</template>

<script>
import { locations, getIcon, icons } from "./mockup";

export default {
  data() {
    return {
      infoWindowOptions: {
        content: "",
      },
      infoWindowPosition: null,
      infoWindowOpen: false,
      currentMarkerId: -1,
      markers: locations,
      enableAddMarker: false,
      dialog: false,
      positionOfMarker: {},
      addressOfMarker: "",
      currentCenter: locations[0].position,
      infoWindowContent: "",
      utilityType: "",
      markerVisible: true,
      animation: 0, // 1 = Bounce, 2 = Drop
    };
  },
  methods: {
    toggleInfoWindow: function (marker) {
      this.infoWindowPosition = marker.position;
      this.infoWindowOptions.content = marker.content;

      if (this.currentMarkerId == marker.id) {
        this.infoWindowOpen = !this.infoWindowOpen;
      } else {
        this.infoWindowOpen = true;
        this.currentMarkerId = marker.id;
      }
    },
    closeInfoWindow() {
      // nếu không có đóng sau khi thoát thì lúc sau mở nó sẽ mở cái đóng, tức là k nhìn thấy gì luôn vì nó bị đóng rồi
      this.infoWindowOpen = false;
    },
    clickTableRow: function (marker) {
      this.currentCenter = marker.position;
    },
    createNewMarker(event) {
      if (!this.enableAddMarker) return;
      this.positionOfMarker = {
        lat: event.latLng.lat(),
        lng: event.latLng.lng(),
      };
      this.dialog = true;
      this.enableAddMarker = false; // reset về k cho create
    },

    saveMarker() {
      this.dialog = false;
      this.markers.push({
        id: this.markers[this.markers.length - 1] + 1,
        position: this.positionOfMarker,
        title: this.addressOfMarker,
        content: this.infoWindowContent,
        icon: getIcon((icons[this.utilityType] && this.utilityType) || "info"),
      });
      // this.currentCenter = this.markers[this.markers.length - 1].position; // neu muon center lai sau khi add marker
    },
  },
};
</script>

<style scoped>
table,
th,
td {
  border: 2px solid blue;
  border-collapse: collapse;
  padding: 2px;
}
</style>