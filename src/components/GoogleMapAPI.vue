<template>
    <GoogleMap
    :api-key="googleMapAPI"
    style="width: 100%; height: 500px"
    :center="center"
    :zoom="zoom"
    id="map"
    >
    <MarkerCluster>
        <Marker v-for="(location, i) in markers" :options="{ position: location }" :key="i" />
    </MarkerCluster>
    </GoogleMap>

    <button @click="getUserLocation">Get Current Location</button>
    <input @keyup.enter="search" v-model="searchLocation">
    <button @click="search">Search Location</button>
</template>

<script>
import { GoogleMap, Marker, MarkerCluster } from 'vue3-google-map'
import axios from 'axios';

export default {
    components: { GoogleMap, Marker, MarkerCluster, axios},
    data() {
        return { 
            center:{ lat: 43.761664, lng: -79.3706496 },
            searchLocation:"",
            userLocation:{value:""},
            markers:[],
            data:"",
            zoom:8,
            googleMapAPI:"YOUR_GOOGLE_MAP_API"
        }
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
            this.markers.push(this.userLocation.value);
            console.log(this.markers);
        },
        async search() {
            const url = "https://maps.googleapis.com/maps/api/geocode/json?address="+this.searchLocation+"&key="+this.googleMapAPI;
            try {
            const response = await axios.get(url);
            this.data = response.data;
            this.zoom = 16;
            const cordinator = this.data.results[0].geometry.location;
            this.center = cordinator;
            // console.log(this.data.results[0].geometry.location);
            this.markers.push(cordinator);
            } catch (error) {
            console.error('Error fetching data:', error);
      }
        },
    }
}
</script>