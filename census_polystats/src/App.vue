<template>
  <v-app id="inspire">
    <v-app-bar
      app
      color="white"
      flat
      
    >
       <v-container class="py-0 fill-height">
        <v-avatar
          class="mr-10"
          color="grey darken-1"
          size="32"
        ></v-avatar>

        <v-btn
          v-for="link in links"
          :key="link"
          text
        >
          {{ link }}
        </v-btn>
        

        <v-spacer></v-spacer>
        
        
      </v-container>
      <v-spacer></v-spacer>
       <v-tabs
          v-model="tab"
          
           right
        >
          <v-tabs-slider color="yellow"></v-tabs-slider>

        <!--  <v-tab
            v-for="item in items"
            :key="item"
            
          >
            {{ item }}
          </v-tab>-->

          <v-tab
           
            :key="analysis"
            
          >
            {{ "analysis" }}
          </v-tab>
          <v-tab
            v-show="showResultsTab"
            :key="results"
            
          >
            {{ "results" }}
          </v-tab>
    </v-tabs>
    </v-app-bar>
      <v-card
    color="grey lighten-4"
    flat
    height="50px"
    tile
    class="toolBarCard"
  >
    <v-toolbar color="grey lighten-4">
      

      <v-toolbar-title>Title</v-toolbar-title>

     


      <v-spacer></v-spacer>

      <v-btn icon>
        <v-icon>mdi-magnify</v-icon>
      </v-btn>

      <v-btn icon>
        <v-icon>mdi-heart</v-icon>
      </v-btn>

      <v-btn icon>
        <v-icon>mdi-dots-vertical</v-icon>
      </v-btn>
    </v-toolbar>
  </v-card>


    <v-main class="grey lighten-3">
     
         <v-snackbar
                v-model="snackbar"
                :timeout="timeout"
                absolute
                bottom
                color="primary"
                left
                text
              >
                {{ text }}

                <template v-slot:action="{ attrs }">
                  <v-btn
                    color="blue"
                    text
                    v-bind="attrs"
                    @click="snackbar = false"
                  >
                    Close
                  </v-btn>
                </template>
              </v-snackbar>
       

  <v-tabs-items v-model="tab" class="v-tab-item-class">
      <v-tab-item :key="analysis">
        <v-row height="100%">
         <!-- <v-col cols="6">
           
          </v-col>-->

          <v-col cols = "12" height="100%">
            <v-sheet
              min-height="70vh"
              rounded="lg"
              height="100%"
            >
              <div class="overlay">
                       <v-sheet rounded="lg">
              <v-form
                ref="form"
                v-model="valid"
                lazy-validation
              >
             
                  <v-card
                    outline
                    class="toolInset"
                  >
                    <v-card-title class="text-h8">
                      Load Boundary File
                    </v-card-title>

                    <!--<v-card-subtitle>Listen to your favorite artists and albums whenever and wherever, online and offline.</v-card-subtitle>-->
                    <div class="spatialFileInput">
                   
                      <v-row>
                        <v-col cols = "12">
                       <v-file-input
                      show-size
                      :rules="rulesUpload"
                      accept=".zip,.geojson"
                      placeholder="Pick an zipped shapefile or GeoJSON File"
                      prepend-icon="mdi-map"
                      label="GeoJSON or SHP"
                      v-model="chosenFile"
                      @change="onAddFiles"
                      required
                      dense
                    ></v-file-input>
                        </v-col>
                        </v-row>
                        <v-row>
                        <v-col cols = "12">
                    <v-select
                        v-model="selectedAttribute"
                        :items="attributes"
                        label="Summary Attribute"
                        outlined
                        :rules="rules"
                        required
                        dense
                      ></v-select>
                       </v-col>
                      </v-row>
                   
                  </div>
                          
                    <v-card-title class="text-h8">
                      Draw Boundary
                    </v-card-title>
            
               
                
                  </v-card>  
                   
                 
                
               
               

                <v-divider class="my-2"></v-divider>
                  <v-list color="transparent">
                <v-list-item
                  link
                  color="grey lighten-4"
                >
                  <v-list-item-content>
                      <div class="text-center">
                        <v-btn
                          class="ma-2"
                          :loading="loading"
                          :disabled="loading"
                          color="secondary"
                          @click="runSummary"
                        >
                          Get Data
                        </v-btn>
                         <v-btn
                          class="ma-2"
                          :loading="loading"
                          :disabled="loading"
                          color="secondary"
                          @click="loader = 'loading'"
                        >
                          Reset
                        </v-btn>
                       </div>
                  </v-list-item-content>
                </v-list-item>
              </v-list>
              </v-form>
            </v-sheet>
              </div>
              <LeafletMap :name="geojson" :selectedAttribute="selectedAttribute"
              > </LeafletMap> 
             
            </v-sheet>
          </v-col>
         
        </v-row>
         </v-tab-item>
          <v-tab-item :key="results">
              <v-dialog
                  v-model="dialog"
                  width="500"
                >
                  <v-container style="height: 400px;background-color:white">
                  <v-row
                    class="fill-height"
                    align-content="center"
                    justify="center"
                  >
                    <v-col
                      class="text-subtitle-1 text-center"
                      cols="12"
                    >
                      Please wait while we prepare census data. This may take several minutes.
                    </v-col>
                    <v-col cols="6">
                      <v-progress-linear
                        color="deep-purple accent-4"
                        indeterminate
                        rounded
                        height="6"
                      ></v-progress-linear>
                    </v-col>
                  </v-row>
                </v-container>
              </v-dialog>
              <v-row>
               <v-col cols="8" v-if= "showResults">
                 <LeafletMapResults :resultsData="resultsData" :selectedAttribute="selectedAttribute" :selectedCensusVariable="selectedCensusVariable"> </LeafletMapResults> 

              </v-col>
              <v-col cols="4" >
                   <v-list-item
                  link
                  color="grey lighten-4"
                >
                  <v-list-item-content>
                    <v-list-item-title>
                      Census Variable
                    </v-list-item-title>
                   
                      <v-select
                        v-model="selectedCensusVariable"
                        :items="censusVariables"
                        label="Census Variables"
                     
                        outlined
                       
                        required
                      ></v-select>
                   
                  </v-list-item-content>
                </v-list-item>
              </v-col>
              </v-row>
              <v-row>
                <v-data-table
                  :headers="tableHeaders"
                  :items="tableValues"
                  :items-per-page="5"
                  class="elevation-1"
                ></v-data-table>
              </v-row>
          </v-tab-item> 
        </v-tabs-items>
        
     
    </v-main>
  </v-app>
</template>

<script>
  import axios from 'axios';
  import shp from 'shpjs';
  import LeafletMap from './components/LeafletMap';
  import LeafletMapResults from './components/LeafletMapResults';
  import * as turf from '@turf/turf'
  export default {
    name: 'app',
    components: {
    LeafletMap,
    LeafletMapResults
    },
    data: () => ({
      rules: [
        v => !!v || 'Name is required'
      ],
     rulesUpload: [
        v => !!v || 'File is required',
        value => !value || value.size < 2000000000 || 'File size should be less than 2 MB!',
      ],
      links: [
        'Profile',
        'Updates',
      ],
      geojson:null,
      geojsonResults:null,
      chosenFile: null,
      msg:null,
      tigerPolygons:null,
      intersections:[],
      loading:false, 
      showResults:true,
      resultsData:null, 
      attributes:[],
      selectedAttribute:null,
      selectedCensusAttribute: null,
      censusAttributes:[],
      selectedCensusGroup: null,
      censusGroups:[],
      groupLookup:null,
      variableIDLookup:null,
      tab: null,
      items: [
          "Analysis", "Results"
        ],
      analysis:"analysis",
      results:"results",
      showResultsTab:true,
      valid: false,
      snackbar: false,
      text: 'Required inputs missing.',
      timeout: 2000,
      dialog:false,
      selectedCensusVariable: null, 
      censusVariables: [],
      tableHeaders: [],
      tableValues: []
    }),
  mounted(){
      //this.populateCensusDropdowns()
  },
  methods:{
    onAddFiles() {
      //for the shapefiles in the files folder called pandr.shp
      
      if (this.chosenFile == null){
        this.geojson = {"empty":"true"}
        this.attributes = []
        return
      }else if(this.chosenFile.name.endsWith(".geojson")){
      
        let reader = new FileReader()
         reader.readAsText(this.chosenFile)
          reader.onload = () => {
            this.geojson = JSON.parse(reader.result);           
            this.updateAttributeSelection()
          }

            //Read the file as text.
  
      }else{
          
          let reader = new FileReader()
          reader.readAsArrayBuffer(this.chosenFile)
          reader.onload = () => {
            this.data = reader.result;
             shp(this.data).then((geojson) => {
            //see bellow for whats here this internally call shp.parseZip()
            this.geojson=geojson
            
            this.updateAttributeSelection()
            });
          }
      }
     },
    updateAttributeSelection(){
       this.attributes = Object.keys(this.geojson.features[0].properties)
       this.selectedAttribute = this.attributes[0]
    }, 

    runSummary(){

      
      if (this.$refs.form.validate() === true){
        console.log(this.valid)
      this.loading = true
      this.dialog = true
      this.tab = 1
           this.showResultsTab =true
           this.geojsonResults = this.geojson
      let bbox = turf.bbox(this.geojson);
      
      const path = 'http://localhost:5000/basicAnalysis';
      let payload = {"layer": this.geojson, "bbox":bbox, "summary_attribute": this.selectedAttribute}
      console.log(payload)
      axios.post(path, payload)
        .then((res) => {
           //this.tab = 1
           //this.showResultsTab =true
           this.geojsonResults = this.geojson
           let resultsData = res.data.data
           this.resultsData = [resultsData,this.geojson, this.selectedAttribute]
           this.censusVariables = Object.keys(resultsData[Object.keys(resultsData)[0]])
           this.selectedCensusVariable = this.censusVariables[0]
           this.loading = false
           this.dialog = false
             this.tableHeaders =  [
          {
            text: 'Census Variable Name',
            align: 'start',
            sortable: false,
            value: 'name',
          },
           { text: 'Census Variable Category', value: 'category' },
          { text: 'Census Value', value: 'value' },
          { text: 'Summary Attribute', value: 'summary_attribute' },
          
           ]

            for (const [key, value] of Object.entries(resultsData)) {
              for (const [subkey, subvalue] of Object.entries(value)) {
                  let tableValue = {
                    name: subkey,
                    category:"None",
                    value: subvalue,
                    summary_attribute:key
                  }
                  this.tableValues.push(tableValue)
              }
            }
          
      
           console.log(resultsData)
           

        })
        .catch((error) => {
          // eslint-disable-next-line
          console.log(error);
         
        });
      }else{
         this.snackbar = true
      }
         }, 

    
       }
    }
</script>
<style>
.overlay {
  height: 100px;
  width: 400px;
  margin-right: -500px;
  padding-left:10px;
  padding-top:10px;
  position: relative;
  float: left;
  z-index: 2;
  pointer-events: auto;

}
.spatialFileInput{
  padding-left:15px;
  padding-right:15px;
}
.v-tab-item-class{
  height: 100%;
}
.toolBarCard{
  padding-top:50px;
}

.toolInset{
  margin:10px;
  top:10px;
}
</style>