<template>
  
  <div>
   
    <div>
      <span v-if="loading">Loading...</span>
      
    </div>

   <div id="mapContainer"></div>
    

  </div>
  
</template>

<script>
import "leaflet/dist/leaflet.css";
import L from "leaflet";
import 'leaflet-draw';
import chroma from "chroma-js";
import { latLngBounds } from "leaflet";
import * as turf from '@turf/turf'
export default {
  name: "Map",
  props: {name:Object, resultsData:Object, selectedAttribute:String, updateStartDraw:Boolean, updateStartDelete:Boolean, sendRemoveDrawPolygon:Boolean, sendRemoveUploadedFile:Boolean},
  watch: {
    sendRemoveUploadedFile(){
     
      if(this.layerGroup!=null){
      
      this.layerGroup.removeLayer(this.geojsonLayer);
      }
    },
    sendRemoveDrawPolygon(){
      if (this.drawnItems != null) {
      this.startDelete()
      }
    },
    updateStartDraw(){
        this.startDraw()
    },
    updateStartDelete(){
        this.startDelete()
    },
    selectedAttribute(val){
        this.attribute = val
    }, 
    name(val) {
      if ("empty" in val){
     
        
        this.layerGroup.removeLayer(this.geojsonLayer);
        

        
      }else{
      this.loading = true;
      this.geojson = JSON.parse(JSON.stringify(val));
      let bbox = turf.bbox(this.geojson);
      let latlngbbox = latLngBounds([
        [bbox[1], bbox[0]],
        [bbox[3], bbox[2]]
      ])
      this.bounds=latlngbbox
      this.map.fitBounds(this.bounds)
      let offset = this.map.getSize().x*0.15;
      // Then move the map
      this.map.panBy(new L.Point(-offset, 0), {animate: false});
      this.geojsonLayer = L.geoJSON(this.geojson,{style: this.style, onEachFeature:this.onEachFeatureFunction})
      this.layerGroup = new L.LayerGroup();
      this.layerGroup.addTo(this.map);
      this.layerGroup.addLayer(this.geojsonLayer);
      
      
      this.loading = false;
      }
    }, 
    resultsData(val){
    
    let allValues = []
    Object.entries(val).forEach(feature => {
      if (feature[0] !=="sf"){
        allValues.push(parseInt(parseFloat(feature[1].sample_data)))
      }
    })

    let max_of_array = Math.max.apply(Math, allValues)
    let min_of_array = Math.min.apply(Math, allValues)
    let classBreaks = this.equalIntervals(min_of_array, max_of_array, 8)

    //let classBreaks = [1,50,100,250,500,1000,2000,3000,6000,9000];
    let colorHex = ['#deebf7','#08306b'];
    function  getColor2(n, classBreaks, colorHex) {
      
      var mapScale = chroma.scale(colorHex).classes(classBreaks);
      
      let regionColor = '#ffffff';
      if (n === 0) {
          regionColor = '#ffffff';
      } else { 
          regionColor = mapScale(n).hex();
      }
    return regionColor
    }


      this.geojson.features.forEach(feature => {
        
        let feature_id = feature.properties[this.attribute]
       
        feature.properties["data_value"] = val[feature_id].sample_data

      
      });
      this.layerGroup.removeLayer(this.geojsonLayer);
     
      
      this.geojsonLayer = L.geoJSON(this.geojson,{style: function (feature) {
  
        return {
    
          weight: 1,
          color: "#ECEFF1",
          opacity: 1,
          fillColor: getColor2(feature.properties.data_value,classBreaks, colorHex),
          fillOpacity: .8
        }
  
    }, onEachFeature:this.onEachFeatureFunction})
      this.layerGroup.addLayer(this.geojsonLayer);
      this.geojsonLayer.addTo(this.map)
      if (this.legend != null){
        this.map.removeControl(this.legend);
      }
      this.legend = L.control({position: 'topright'});
      this.legend.onAdd = function () {
          var div = L.DomUtil.create('div', 'legend');
          div.innerHTML += '<i style="background: #ffffff;"></i>0';
          //classBreaks.push(999); // add dummy class to extend to get last class color, chroma only returns class.length - 1 colors
          for (var i = 0; i < classBreaks.length; i++) {
              if (i+2 === classBreaks.length) {
                  div.innerHTML += '<i style="background: ' + getColor2(classBreaks[i], classBreaks, colorHex) + ';"></i> ' +
                  classBreaks[i] + '+';
                  break
              } else {
                  div.innerHTML += '<i style="background: ' + getColor2(classBreaks[i], classBreaks, colorHex) + ';"></i> ' +
                  classBreaks[i] + '–' + classBreaks[i+1] + '<br>';
              }
          }
          return div;
      };
      this.legend.addTo(this.map);
      
      
    }
  },  
  mounted(){

    this.setupLeafletMap();
    
  },
  data() {
    return {
      legend:null,
      center: [37,7749, -122,4194],
      loading: false,
      show: true,
      enableTooltip: true,
      zoom: 6,
      //center: [38, -122.2],
      bounds: latLngBounds([
        [49, -66],
        [25, -124]
      ]),
      map:null,
      drawControl:null,
      layerGroup: null,
      drawnItems:null,
      drawnItemGeojson:null, 
      geojsonLayer:null,
      geojson: null,
      attribute:null,
      fillColor: "#e4ce7f",
      classBreaks: [1,50,100,250,500,1000,2000,3000,6000,9000],
      colorHex: ['#deebf7','#08306b'],
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
     
    };
  },
  computed: {
    options() {
      return {
        onEachFeature: this.onEachFeatureFunction
      };
    },
    
   style() {
    return {
     
        fillColor: '#fa9fb5',
        weight: 1,
        opacity: 1,
        color: 'white',
        fillOpacity: 0.8
      
    };
    },
    onEachFeatureFunction() {
   
      return (feature, layer) => {
        let attributeHTML = ""
        for (const [key, value] of Object.entries(feature.properties)) {
          attributeHTML += "<div>"+ key + ": " + value + "</div>"
        }
        layer.bindTooltip(attributeHTML)
      };
    }
  },
  methods: {
   setupLeafletMap: function () {
     this.map = L.map("mapContainer", { zoomControl: false }).fitBounds(this.bounds)//.setView([51.505, -0.09], 13);
     //const mapDiv = L.map("mapContainer").setView(this.center, 13);
      L.tileLayer('https://{s}.basemaps.cartocdn.com/light_nolabels/{z}/{x}/{y}{r}.png', {
        maxZoom: 20,
        attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors &copy; <a href="https://carto.com/attributions">CARTO</a>',
        //bounds: this.bounds
     }).addTo(this.map);

     L.control.zoom({
    position: 'topright'
      }).addTo(this.map);

      let offset = this.map.getSize().x*0.15;
      // Then move the map
      this.map.panBy(new L.Point(-offset, 0), {animate: false});
        },
   
  

    equalIntervals: function(low, high, numberOfIntervals){
        let diff = high-low
        let eachInterval = diff/numberOfIntervals
        let outIntervals = []
        for (let i = 0; i < numberOfIntervals; i++){
          low+=eachInterval
          outIntervals.push(low)
        }

     return outIntervals
    }, 

    startDraw:function(){
          this.drawnItems = new L.FeatureGroup();
          this.map.addLayer(this.drawnItems);
      
          let polygonDrawer = new L.Draw.Polygon(this.map);
          polygonDrawer.enable();
          // Assumming you have a Leaflet map accessible
          //var vm = this
          this.map.on('draw:created', (e) => {
             let layer = e.layer;

            // Do whatever you want with the layer.
            // e.type will be the type of layer that has been draw (polyline, marker, polygon, rectangle, circle)
            // E.g. add it to the map
            
           this.drawnItems.addLayer(layer);
             // layer.editing.enable()
           this.drawnItems.addTo(this.map)
           this.drawnItemGeojson = this.drawnItems.toGeoJSON()
           this.$emit('drawnItemGeojsonCreated', this.drawnItemGeojson)
          
              
              //console.log(this.drawnItemGeojson)
            });
             
         
         
    },
    startDelete:function(){
      var layers = this.drawnItems.getLayers();
      layers[layers.length - 1].remove();
  
    }
 },
  async created() {
    //this.loading = true;
    //const response = await fetch("https://rawgit.com/gregoiredavid/france-geojson/master/regions/pays-de-la-loire/communes-pays-de-la-loire.geojson")
    //const data = await response.json();
    //console.log(data)
    //this.geojson = data;
    //this.loading = false;
  }
};
</script>
<style>
  #mapContainer{
              
                z-index: 1;
              
              height: calc(100vh - 136px) !important;
                position: relative;
                }

  .legend {
	line-height: 18px;
	color: #555;
  background: #ffffff;
}
.legend i {
	width: 18px;
	height: 18px;
	float: left;
	margin-right: 8px;
	opacity: 0.7;
}

.v-main{
  padding-top: 56px !important;
}

</style>
