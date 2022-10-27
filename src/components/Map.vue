<template>
    <form id="sole-form">
        <!-- Origin -->
        <div class="row">             
            <label class="form-label">Your Origin:</label>            
            <div class="col">
                <input type="text" class="form-control" placeholder="Enter longitude" v-model="waypoints[0].lng">
            </div>
            <div class="col">
                <input type="password" class="form-control" placeholder="Enter latitude" v-model="waypoints[0].lat">
            </div>
        </div>
        <hr>
        <!-- Waypoints -->
        <div class="row"> 
            <label class="form-label">Destination 1:</label>    
            <div class="col">
                <input type="text" class="form-control" placeholder="Enter longitude" v-model="waypoints[1].lng">
            </div>
            <div class="col">
                <input type="password" class="form-control" placeholder="Enter latitude" v-model="waypoints[1].lat">
            </div>
        </div>
         <hr>
        <div class="row"> 
            <label class="form-label">Destination 2:</label>    
            <div class="col">
                <input type="text" class="form-control" placeholder="Enter longitude" v-model="waypoints[2].lng">
            </div>
            <div class="col">
                <input type="password" class="form-control" placeholder="Enter latitude" v-model="waypoints[2].lat">
            </div>
        </div>
        <hr>
        <div class="row"> 
            <label class="form-label">Destination 3:</label>
            <div class="col">
                <input type="text" class="form-control" placeholder="Enter longitude" v-model="waypoints[3].lng">
            </div>
            <div class="col">
                <input type="password" class="form-control" placeholder="Enter latitude" v-model="waypoints[3].lat">
            </div>
        </div>
        <hr>
        <div class="row"> 
            <label class="form-label">Destination 4:</label>
            <div class="col">
                <input type="text" class="form-control" placeholder="Enter longitude" v-model="waypoints[4].lng">
            </div>
            <div class="col">
                <input type="password" class="form-control" placeholder="Enter latitude" v-model="waypoints[4].lat">
            </div>
        </div>
         <hr>
        <div class="row"> 
            <label class="form-label">Destination 5:</label>
            <div class="col">
                <input type="text" class="form-control" placeholder="Enter longitude" v-model="waypoints[5].lng">
            </div>
            <div class="col">
                <input type="password" class="form-control" placeholder="Enter latitude" v-model="waypoints[5].lat">
            </div>
        </div>   
        <hr>
        <div class="row"> 
            <label class="form-label">Destination 6:</label>
            <div class="col">
                <input type="text" class="form-control" placeholder="Enter longitude" v-model="waypoints[6].lng">
            </div>
            <div class="col">
                <input type="password" class="form-control" placeholder="Enter latitude" v-model="waypoints[6].lat">
            </div>
        </div>
        <hr>
        <div class="row"> 
            <label class="form-label">Destination 7:</label>
            <div class="col">
                <input type="text" class="form-control" placeholder="Enter longitude" v-model="waypoints[7].lng">
            </div>
            <div class="col">
                <input type="password" class="form-control" placeholder="Enter latitude" v-model="waypoints[7].lat">
            </div>
        </div>
        <button @click="getOptimizedRoutes">Submit</button>       
    </form>

    <!-- Map below -->
	<div id='map' class='map'></div>
</template>

<script>
export default {
  name: "post-request",
  data() {
    return {
            waypoints: [
                { lat: '',  lng : ''},
                { lat: '', lng: '' },
                { lat: '', lng: '' } ,
                { lat: '', lng: '' },    
                { lat: '', lng: '' },    
                { lat: '', lng: '' },    
                { lat: '', lng: '' },    
                { lat: '', lng: '' },                
            ],
    };
  },
  methods: {
    getOptimizedRoutes(e) {
        e.preventDefault();

        const data = {
            waypoints: this.waypoints.map((location) => {                
                return {
                    point: {
                        latitude: Number(location.lat),
                        longitude: Number(location.lng),
                    },
                };
            }),

            options: {
                travelMode: "truck",
                vehicleMaxSpeed: 0,
                vehicleCommercial: true,
                traffic: "live",
                departAt: "now",
                outputExtensions: ["travelTimes", "routeLengths"],
                waypointConstraints: {
                    originIndex: 0,
                    destinationIndex: 0,
                },
            },
        };

        const requestOptions = {
            method: "POST",
            headers: { "Content-type": "application/json;charset=UTF-8" },
            body: JSON.stringify(data),
        };

        fetch(`https://api.tomtom.com/routing/waypointoptimization/1?key=${import.meta.env.VITE_APP_API_KEY}`, requestOptions)
            .then(response => response.json())
            .then((data) => {
                // For the markers
                let markers = []                                 
                let solution = data.optimizedOrder

                // Get user origin
                var centerLocation = this.waypoints[0]

                // Hide the form
                document.getElementById('sole-form').remove()

                let map = tt.map({
                key:import.meta.env.VITE_APP_API_KEY,
                container: 'map',
                center: centerLocation,
                bearing: 0,
                maxZoom: 21,
                minZoom: 1,
                pitch: 60,
                zoom: 14,
                });        

                map.addControl(new tt.FullscreenControl()); 
                map.addControl(new tt.NavigationControl()); 

                // Add markers to map
                this.waypoints.forEach(function (location, index) {                    
                    markers.push(new tt.Marker().setLngLat(location).addTo(map))                                                                                                                                       
                })                

                
                let optimizedLocs = solution.map((order, index) => {  
                        if(order === 0) {
                            const popup = new tt.Popup({ offset: 50 }).setText("Origin")
                            markers[order].setPopup(popup).togglePopup()
                        } else {
                            const popup = new tt.Popup({ offset: 50 }).setText("Destination #" + index)
                            markers[order].setPopup(popup).togglePopup()                    
                        }                                    
                                                
                        return this.waypoints[order]
                })                                                         
                
            })
    },      
  },
  created() {
    console.log('Loaded!');
  }
};
	
</script>

<!-- Use preprocessors via the lang attribute! e.g. <style lang="scss"> -->
<style>
#app {
	font-family: Avenir, Helvetica, Arial, sans-serif;
	text-align: center;
	color: #2c3e50;
	margin-top: 60px;
}

a,
button {
	color: #4fc08d;
}

.map {
    width: 80vw;
    height: 70vh;
}


button {
	background: none;
	border: solid 1px;
    margin-top: 1rem;
	border-radius: 2em;
	font: inherit;
	padding: 0.75em 2em;
}
</style>
