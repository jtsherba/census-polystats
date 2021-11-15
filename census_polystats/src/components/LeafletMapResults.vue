<template>

  <div>
    <div>
      <span v-if="loading">Loading...</span>
      
    </div>
   <div id="mapContainerResults"></div>
  </div>
</template>

<script>
import "leaflet/dist/leaflet.css";
import L from "leaflet";
import chroma from "chroma-js";
import { latLngBounds } from "leaflet";
import * as turf from '@turf/turf'
export default {
  name: "MapResults",
  props: {name:Object, resultsData:Array, selectedAttribute:String, selectedCensusVariable:String},
  watch: {
    selectedCensusVariable(val){
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
      if(this.setFirstCensusVariable == true){
         
          this.setFirstCensusVariable = false
      }else{
           this.censusVariable = val

           console.log(this.censusVariable)
      this.allValues = []

      Object.entries(this.resultData).forEach(feature => {
      
        if (feature[0] !=="sf"){
          
          this.allValues.push(parseFloat(feature[1][this.censusVariable]))
        }
      })
      
      let max_of_array = Math.max.apply(Math, this.allValues)
      let min_of_array = Math.min.apply(Math, this.allValues)
      let classBreaks = this.equalIntervals(min_of_array, max_of_array, 8)

      //let classBreaks = [1,50,100,250,500,1000,2000,3000,6000,9000];
      let colorHex = ['#deebf7','#08306b'];
      

        this.resultGeojson.features.forEach(feature => {
        
          let feature_id = feature.properties[this.currentAttribute]
        
          feature.properties["data_value"] = this.resultData[feature_id][this.censusVariable]

        
        });
        this.layerGroup.removeLayer(this.geojsonLayer);
      
        
        this.geojsonLayer = L.geoJSON(this.resultGeojson,{style: function (feature) {
    
          return {
      
            weight: 1,
            color: "#ECEFF1",
            opacity: 1,
            fillColor: getColor2(feature.properties.data_value,classBreaks, colorHex),
            fillOpacity: .8
          }
    
      }, onEachFeature:this.onEachFeatureFunction})

       /* let bbox = turf.bbox(this.resultGeojson);
        let latlngbbox = latLngBounds([
          [bbox[1], bbox[0]],
          [bbox[3], bbox[2]]
        ])
        this.bounds=latlngbbox*/
        this.mapResults.fitBounds(this.bounds)


        //this.layerGroup = new L.LayerGroup();
        this.layerGroup.addLayer(this.resultGeojson);
        this.geojsonLayer.addTo(this.mapResults)
        if (this.legend != null){
          this.mapResults.removeControl(this.legend);
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
        this.legend.addTo(this.mapResults);
        
        
      
      }
        
    },
    selectedAttribute(val){
        this.attribute = val
    }, 
    resultsData(allResults){
   
    this.resultData = allResults[0]
 
    let censusVariablesDetail = this.resultData[Object.keys(this.resultData)[0]]
   
    this.censusVariable = Object.keys(censusVariablesDetail)[0]// Object.keys(this.resultData[0])[0]
    
    this.resultGeojson = allResults[1]
    this.currentAttribute = allResults[2]
    this.allValues = []

    Object.entries(this.resultData).forEach(feature => {
     
      if (feature[0] !=="sf"){
        
        this.allValues.push(parseFloat(feature[1][this.censusVariable]))
      }
    })

    let max_of_array = Math.max.apply(Math, this.allValues)
    let min_of_array = Math.min.apply(Math, this.allValues)
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

      this.resultGeojson.features.forEach(feature => {
       
        let feature_id = feature.properties[this.currentAttribute]
       
        feature.properties["data_value"] = this.resultData[feature_id][this.censusVariable]

      
      });
      //this.layerGroup.removeLayer(this.geojsonLayer);
     
      
      this.geojsonLayer = L.geoJSON(this.resultGeojson,{style: function (feature) {
  
        return {
    
          weight: 1,
          color: "#ECEFF1",
          opacity: 1,
          fillColor: getColor2(feature.properties.data_value,classBreaks, colorHex),
          fillOpacity: .8
        }
  
    }, onEachFeature:this.onEachFeatureFunction})

      let bbox = turf.bbox(this.resultGeojson);
      let latlngbbox = latLngBounds([
        [bbox[1], bbox[0]],
        [bbox[3], bbox[2]]
      ])
      this.bounds=latlngbbox
      this.mapResults.fitBounds(this.bounds)


      this.layerGroup = new L.LayerGroup();
      this.layerGroup.addLayer(this.resultGeojson);
      this.geojsonLayer.addTo(this.mapResults)
      if (this.legend != null){
        this.mapResults.removeControl(this.legend);
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
      this.legend.addTo(this.mapResults);
      
      
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
      mapResults:null,
      layerGroup: null,
      geojsonLayer:null,
      geojson: null,
      attribute:null,
      resultData:null,
      censusVariable:null,
      resultGeojson:null,
      currentAttribute:null,
      allValues:[],
      fillColor: "#e4ce7f",
      classBreaks: [1,50,100,250,500,1000,2000,3000,6000,9000],
      colorHex: ['#deebf7','#08306b'],
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      setFirstCensusVariable:true,
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
     this.mapResults = L.map("mapContainerResults").fitBounds(this.bounds)//.setView([51.505, -0.09], 13);
     //const mapDiv = L.map("mapContainer").setView(this.center, 13);
      L.tileLayer('https://{s}.basemaps.cartocdn.com/light_nolabels/{z}/{x}/{y}{r}.png', {
        maxZoom: 20,
        attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors &copy; <a href="https://carto.com/attributions">CARTO</a>',
        //bounds: this.bounds
     }).addTo(this.mapResults);
    this.mapResults.invalidateSize()
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
  #mapContainerResults{height:450px;
                z-index: 1;
                margin-right:20px}

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
</style>
