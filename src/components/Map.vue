<script>
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'
import { v4 as uuidv4 } from 'uuid'

export default{

    data() {
        return {
            level: "surface",
            map: null,
            markers: [],
            icons: [
                L.icon({iconUrl: 'totk/icons/korok.png', iconSize:[24, 24]}),
                L.icon({iconUrl: 'totk/icons/treasure.png', iconSize:[24, 24]}),
                L.icon({iconUrl: 'totk/icons/shrine_resurrection.png', iconSize:[24, 24]}),
                L.icon({iconUrl: 'totk/icons/hinox.png', iconSize:[24, 24]}),
                
            ]
        }
    },
    methods: {
        markerPrompt(e) {
            var name = prompt("0:克洛洛\n1:宝箱\n2:洞穴\n3:怪物\n请输入信息："); 
            if (name == null || name == undefined || name.length == 0) return;
            var array = name.split("#")
            this.markers.push({type: array[0], mark: array[1], lat: e.latlng.lat, lng: e.latlng.lng, id: uuidv4(), level: this.level})
            localStorage.setItem("markers", JSON.stringify(this.markers))

            var val = this.markers[this.markers.length-1]
            console.log(val)
            L.marker([val.lat, val.lng], {icon: this.icons[val.type]}).bindPopup(val.mark).addTo(this.map)

            
        },

        changeLevel(level) {
            if (this.level == level) {
                return
            }
            this.level = level
            this.createMap()
        },
        createMap() {

            this.markers =  JSON.parse(localStorage.getItem("markers"))
            if (this.markers == null || this.markers == undefined) {
                this.markers = []
            }

            if (this.map != null || this.map != undefined) {
                this.map.remove()
            }
            

            var bounds = new L.LatLngBounds(
                new L.LatLng(6000, 6000), new L.LatLng(-6000, -6000)
            );

            var level = this.level

            const map = L.map('map',{
                crs: L.CRS.Simple,
                attributionControl: false,
                maxBounds: bounds,
                maxBoundsViscosity: 1.0,
            }).setView([-277.46875, 270.15625], 3);

            L.tileLayer('totk/tiles/'+level+'/{z}/{x}_{y}.jpg', {
                tileSize: 564,
                minZoom: 0,
                maxZoom: 6,
                bounds: bounds,
                noWarp: true,
            }).addTo(map);

            map.on('click', this.markerPrompt);

            this.map = map;

        
            var icons = this.icons
            this.markers.forEach(function(val, idx, arr) {
                if (val.level == level) {
                    L.marker([val.lat, val.lng], {icon: icons[val.type]}).bindPopup(val.mark).addTo(map)
                }
            })
        }
    },
    mounted(){
        this.createMap()
    }
}

</script>

<template>
    <div style="float: left;">
        <button @click="changeLevel('sky')">sky</button>
        <button @click="changeLevel('surface')">surface</button>
        <button @click="changeLevel('depths')">depths</button>
    </div>
    <div id="map"></div>
</template>

<style scoped>
#map  {
        height: 100vh;
        width: 100vw;
    }
</style>
