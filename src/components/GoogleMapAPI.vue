<template>
    <GoogleMap
    api-key="AIzaSyAYIhhJ30QsV0PtqhEiw2200KorkbsPsp0"
    style="width: 100%; height: 500px"
    :center="center"
    :zoom="8"
    id="map"
    >
    <MarkerCluster>
        <Marker v-for="(location, i) in markers" :options="{ position: location }" :key="i" />
    </MarkerCluster>
    </GoogleMap>

    <button @click="push">push</button>
    <button @click="getUserLocation">Get Current Location</button>
    <input @keyup.enter="fetchData" v-model="searchLocation">
</template>

<script>
import { GoogleMap, Marker, MarkerCluster } from 'vue3-google-map'
import axios from 'axios';

export default {
    components: { GoogleMap, Marker, MarkerCluster, axios},
    data() {
        return { 
            center:{ lat: 40.689247, lng: -74.044502 },
            searchLocation:"",
            userLocation:{value:""},
            markers:[
            { lat: 40.689247, lng: -74.044502 }
            ],
            data:""
        }
    },
    methods:{
        getUserLocation() {
            // Use the Geolocation API to get the user's current location
            // console.log(navigator);
            if ("geolocation" in navigator) {
                navigator.geolocation.getCurrentPosition(position => {
                    this.userLocation.value = {
                        lat: position.coords.latitude,
                        lng: position.coords.longitude
                    };
                    this.addMarker(this.userLocation.value, 'You are here');
                    // map.setCenter(this.userLocation.value);
                });
            }
        },
        addMarker() {
            const map = new google.maps.Map(document.getElementById("map"), {
                zoom: 8,
                center: this.userLocation.value,
            });
            // console.log(map);
            const marker = new google.maps.Marker({
                position: this.userLocation.value,
                map,
                title: "title"
            });
            // console.log(this.userLocation.value);
            this.markers.push(this.userLocation.value);
            // console.log(marker);
            // this.markers.value.push(marker);
            console.log(this.markers);
        },
        async fetchData() {
            const url = "https://maps.googleapis.com/maps/api/geocode/json?address="+this.searchLocation+"&key=AIzaSyAYIhhJ30QsV0PtqhEiw2200KorkbsPsp0"
            try {
            const response = await axios.get(url); // Replace with your API endpoint
            this.data = response.data;
            console.log(this.data.results[0].geometry.location);
            this.markers.push(this.data.results[0].geometry.location);
            } catch (error) {
            console.error('Error fetching data:', error);
      }
        },
        push(){
            this.markers.push({ lat: 43, lng: -80 });
            console.log(this.markers);
        },
        callbackFunction(place) {
            console.log(place);
        }
    }
}
</script>