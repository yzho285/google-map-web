<template>
    <GoogleMap
    :api-key="googleMapAPI"
    style="width: 100%; height: 500px"
    :center="center"
    :zoom="zoom"
    id="map"
    >
    <MarkerCluster>
        <Marker v-for="(position, place_id) in markers" :options="position" :key="place_id" />
    </MarkerCluster>
    </GoogleMap>

    <el-button @click="getUserLocation">Get Current Location</el-button>    
    <el-input @keyup.enter="search" v-model="searchLocation" />
    <el-button @click="search">Search Location</el-button>
    <!-- <Table /> -->
    <el-table 
        :data="markers.slice((currentPage - 1) * pageSize, currentPage * pageSize)" 
        style="width: 100%" 
        :key="markers.place_id"
        @selection-change="selectionLineChangeHandle">
        <el-table-column type="selection" width="55" />
        <el-table-column prop="place_id" label="place_id" width="180" />
        <el-table-column prop="formatted_address" label="formatted_address" />
    </el-table>
    <el-button @click="deleteLocation">Delete</el-button>
    <el-pagination
      v-model:current-page="currentPage"
      v-model:page-size="pageSize"
      :page-sizes="[10, 20, 30, 40]"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />
</template>

<script>
import { GoogleMap, Marker, MarkerCluster } from 'vue3-google-map'
import axios from 'axios';
// import Table from './components/Table.vue'

export default {
    components: { GoogleMap, Marker, MarkerCluster, axios},
    data() {
        return { 
            center:{ lat: 43.761664, lng: -79.3706496 },
            searchLocation:"",
            userLocation:{value:""},
            // markers:[{formatted_address: "Toronto, ON, Canada",
            //         place_id: "ChIJpTvG15DL1IkRd8S0KlBVNTI",
            //         position: {lat: 43.653226, lng: -79.3831843}}],
            markers:[],
            data:"",
            zoom:8,
            googleMapAPI:"YOUR_GOOGLE_API",
            currentPage:1,
            pageSize:10,
            total: 0,
            dataonLineListSelections: [],
    
        }
    },
    created(){
        this.total = this.markers.length;
    },
    methods:{
        getUserLocation() {
            // Use the Geolocation API to get the user's current location
            if ("geolocation" in navigator) {
                navigator.geolocation.getCurrentPosition(position => {
                    this.userLocation.value = {
                        lat: position.coords.latitude,
                        lng: position.coords.longitude
                    };
                    this.zoom = 16;
                    this.addMarker(this.userLocation.value, 'You are here');
                    this.total = this.markers.length;
                });
            }
        },
        addMarker() {
            // const map = new google.maps.Map(document.getElementById("map"), {
            //     zoom: 8,
            //     center: this.userLocation.value,
            // });
            // // console.log(map);
            // const marker = new google.maps.Marker({
            //     position: this.userLocation.value,
            //     map,
            //     title: "title"
            // });
            // console.log(this.userLocation.value);
            this.center = this.userLocation.value;
            const marker = {position:this.userLocation.value}
            this.markers.push(marker);
            // console.log(this.markers);
        },
        async search() {
            const url = "https://maps.googleapis.com/maps/api/geocode/json?address="+this.searchLocation+"&key="+this.googleMapAPI;
            try {
            const response = await axios.get(url);
            const data = response.data;
            // console.log(data);
            this.zoom = 16;
            const cordinator = data.results[0].geometry.location;
            this.center = cordinator;
            const marker = {position:data.results[0].geometry.location, 
                            place_id:data.results[0].place_id,
                            formatted_address:data.results[0].formatted_address};
            const place_ids = this.markers.map(item => item.place_id);
            if(!place_ids.includes(marker.place_id)){
                this.markers.push(marker);
            }
            this.total = this.markers.length;
            // console.log(this.markers);
            } catch (error) {
            console.error('Error fetching data:', error);
            }
        },
        handleSizeChange(val){
            this.pageSize = val;
        },
        handleCurrentChange(val){
            this.currentPage = val;
        },
        selectionLineChangeHandle(val){
            this.selectedArr = val;
        },
        deleteLocation(){
            const selectPlaceID = this.selectedArr.map(item => item.place_id);
            this.markers = this.markers.filter(marker => !selectPlaceID.includes(marker.place_id) )
        }
    }
}
</script>

<!-- <style scoped>
.example-pagination-block + .example-pagination-block {
  margin-top: 10px;
}
.example-pagination-block .example-demonstration {
  margin-bottom: 16px;
}
</style> -->