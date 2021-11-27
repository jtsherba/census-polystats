<template>
  <v-app id="inspire">
    <v-app-bar
      app
      color="white"
      flat
      height="64"
      
    >
       <v-container class="py-0 fill-height">
        <v-avatar
          class="mr-10"
          color="grey darken-1"
          size="32"
        ></v-avatar>

       <v-toolbar-title>Census Polystats</v-toolbar-title>
        

        <v-spacer></v-spacer>
        
        
      </v-container>
      <v-spacer></v-spacer>
       <v-btn
          depressed
          class="ma-2"
          outlined
          color="indigo"
          v-on:click="setAboutTab()"
        >
          About
        </v-btn>
      
    </v-app-bar>

  <v-card
    color="grey lighten-4"
    flat
    tile
    class="toolBarCard"
   
  >
    <v-toolbar  height="80" color="grey lighten-4">
        <v-tabs
          v-model="tab"
          color="deep-purple accent-4"
           :class="{visible: tabsVisible}"
           left
        >
          <v-tabs-slider ></v-tabs-slider>

      

          <v-tab
           
            :key="analysis"
            href='#tab-1'
          >
            {{ "Run Analysis" }}
          </v-tab>
          <v-tab
       
            v-show="showResultsTab"
            :key="results"
            href='#tab-2'
          >
            {{ "results" }}
          </v-tab>
          
    </v-tabs>
    
      <v-spacer></v-spacer>

       <v-btn v-if="analysisRun" @click="changeTab()">{{ changeTabText }}</v-btn>          
     
     
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
      <v-tab-item :key="analysis" value='tab-1'>
        <v-row height="100%">
         <!-- <v-col cols="6">
           
          </v-col>-->

          <v-col cols = "12" height="100%">
            <v-sheet
              min-height="70vh"
              rounded="lg"
              
            >
              <div class="overlay">
              <v-sheet rounded="lg" height="100%" style="padding-bottom: 18px; background:rgba(63, 81, 181,0);">
              <v-form
                ref="form"
                v-model="valid"
                lazy-validation
                height="100%"
              >
            
                  <v-card
                   height="100%"
                    outline
                    class="toolInset"
                  >
                  <v-card-title class="text-h8">
                      Set Analysis Area
                    </v-card-title>
                    <div class="spatialFileInput">
                   
                      <v-row>
                        <v-col cols = "12">
                     <v-btn
                        depressed
                        class="ma-2"
                        outlined
                        color="indigo"
                        v-on:click="updateDrawPolygon()"
                      >
                        Draw Polygon
                      </v-btn>
                      <v-btn
                        depressed
                        class="ma-2"
                        outlined
                        color="secondary"
                        v-on:click="deleteDrawPolygon()"
                      >
                        Delete
                      </v-btn>
                        </v-col>
                      </v-row>
                    </div>  
                  <v-expansion-panels
                    flat
                    class="loadFileExpansion"
                  >
                  <v-expansion-panel>
                  <v-expansion-panel-header>Or load a file</v-expansion-panel-header>
                  <v-expansion-panel-content>
                    

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
                  </v-expansion-panel-content>
                </v-expansion-panel>
                  </v-expansion-panels>
                  
                          
                    <v-divider class="runControlDivider"></v-divider>
                   
  

                    <v-btn
                      text
                      color="primary"
                      right 
                      small
                      @click="expandOptions"
                    >
                    Advanced Options
                   <v-icon
                    right
                   
                  >
                  mdi-chevron-right
                  </v-icon>
                   </v-btn>
                <div class="runButtons">
                  
                  <v-btn
                 class="ma-1"
                    outlined
                  color="indigo"
                  :loading="loading"
                  :disabled="runButtonDisabled"
                  right
                  @click="runSummary"
                >
                  Run
                </v-btn>
                 <v-btn
                  class="ma-2"
                  outlined
                  :loading="loading"
                  :disabled="loading"
                  color="secondary"
                  @click="resetAll()"
                  right
                >
                  Reset
                </v-btn>
                
                </div>
              
                  </v-card>  
                 
               
              </v-form>
            </v-sheet>
              </div>

              <div class="overlay2" v-if="showAdvancedOptions">
              <v-sheet rounded="lg" height="100%" style="padding-bottom: 18px; background:rgba(63, 81, 181,0);">
                    <v-card
                    height="100%"
                    outline
                   class="toolInset" 
                  >
                  <div class="runOptions">
                    <v-row>
                      
                    <v-select
                        class="optionsSelect ma-1"
                        dense
                        v-model= "selectedCensusStatistics"
                        :items="censusStatistics"
                        label="Census Statistics"
                        outlined
                        v-on:change="changeCensusType"
                      ></v-select>
                    <v-file-input
                    class="loadCustom ma-1"
                    v-if="showCensusStatisticsFileInput"
                    accept="*.json"
                    label="File input"
                    ></v-file-input>
                    <v-select
                        class="optionsSelect ma-1"
                        dense
                        v-model= "selectedSummaryMethod"
                        :items="summaryMethod"
                        label="Summary Method"
                        outlined
                      ></v-select>
                      <v-switch
                        class="ma-1"
                        v-model="switch1"
                        label="Inlcude unsummarized layer"
                      ></v-switch>
                      <v-select
                      class="ma-1"
                      dense
                        v-model= "selectedYears"
                        :items="years"
                        label="Years"
                        multiple
                        outlined
                      ></v-select>
                      </v-row>


                </div>
                  </v-card>  
               </v-sheet>
              </div>
              <LeafletMap @drawnItemGeojsonCreated="drawnItemGeojsonCreated" :name="geojson" :selectedAttribute="selectedAttribute" :updateStartDraw="startDraw" :updateStartDelete="startDelete" :sendRemoveDrawPolygon="sendRemoveDrawPolygon" :sendRemoveUploadedFile="sendRemoveUploadedFile"
              > </LeafletMap> 
             
            </v-sheet>
          </v-col>
         
        </v-row>
         </v-tab-item>
          <v-tab-item :key="results" value="tab-2">
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
                 <LeafletMapResults :resultsData="resultsData" :selectedAttribute="selectedAttribute" :selectedCensusVariable="selectedCensusVariable" > </LeafletMapResults> 

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
        //v => !!v || 'File is required',
        value => !value || value.size < 2000000000 || 'File size should be less than 2 MB!',
      ],
      links: [
        'Profile',
        'Updates',
      ],
      geojson:null,
      drawnItemGeojson:null,
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
      startDraw:null,
      startDelete:null,
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
      tableValues: [],
      sendRemoveDrawPolygon: false, 
      sendRemoveUploadedFile:false,
      runButtonDisabled:true, 
      tabsVisible:true,
      changeTabText:"Back to Analysis",
      analysisRun:false,
      summaryMethod: ["Apportion by Area", "Contains", "Fully Contains"],
      selectedSummaryMethod: "Apportion by Area",
      censusStatistics: ["Basic Socio-economic Profile", "Load Custom"],
      selectedCensusStatistics: "Basic Socio-economic Profile",
      showCensusStatisticsFileInput:false,
      selectedYears: ["2019"],
      years:["2015", "2016", "2017", "2018", "2019"],
      showAdvancedOptions:false,

      
    }),
  mounted(){
      //this.populateCensusDropdowns()
  },
  methods:{
    expandOptions(){
        this.showAdvancedOptions = !this.showAdvancedOptions
    },
    changeCensusType(){
        console.log(this.selectedCensusStatistics)
        if (this.selectedCensusStatistics == "Load Custom"){
            this.showCensusStatisticsFileInput=true
        }else{
          this.showCensusStatisticsFileInput=false
        }
    },
    changeTab(){
    if (this.tab == "tab-1"){
        this.changeTabText = "Back to Analysis"
        this.tab="tab-2"
    }else{
     this.changeTabText = "Back to Results"
     this.tab='tab-1'

    }
     
    },
    resetAll(){
      this.runButtonDisabled = true
       this.sendRemoveDrawPolygon = !this.sendRemoveDrawPolygon
        this.sendRemoveUploadedFile = !this.sendRemoveUploadedFile
        this.geojson = {"empty":"true"}
        this.chosenFile = null
        this.attributes = []
    },
    drawnItemGeojsonCreated(value) {
      this.drawnItemGeojson = value // someValue
      this.runButtonDisabled = false
    },
    deleteDrawPolygon(){
      this.startDelete = !this.startDelete
      this.runButtonDisabled = true
    },
    updateDrawPolygon(){
      this.chosenFile = null
      this.attributes = []
         this.sendRemoveDrawPolygon = !this.sendRemoveDrawPolygon
        this.sendRemoveUploadedFile = !this.sendRemoveUploadedFile
        this.startDraw = !this.startDraw
    },
    onAddFiles() {
      //for the shapefiles in the files folder called pandr.shp
      this.sendRemoveDrawPolygon = !this.sendRemoveDrawPolygon
      if (this.chosenFile == null){
        this.geojson = {"empty":"true"}
        this.runButtonDisabled = true
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
      this.runButtonDisabled = false
     },
    updateAttributeSelection(){
       this.attributes = Object.keys(this.geojson.features[0].properties)
       this.selectedAttribute = this.attributes[0]
    }, 

    runSummary(){
      if(this.drawnItemGeojson != null){
        this.drawnItemGeojson.features[0].properties['id'] = "1"
          this.loading = true
      this.dialog = true
      this.tab = "tab-2"
           this.showResultsTab =true
           this.geojsonResults = this.drawnItemGeojson
      let bbox = turf.bbox(this.drawnItemGeojson);
      this.selectedAttribute = "id"
      const path = 'http://localhost:5000/basicAnalysis';
      let payload = {"layer": this.drawnItemGeojson, "bbox":bbox, "summary_attribute": this.selectedAttribute}
      console.log(payload)
      axios.post(path, payload)
        .then((res) => {
           //this.tab = 1
           //this.showResultsTab =true
           //this.geojsonResults = this.geojson
        
           let resultsData = res.data.data
           this.resultsData = [resultsData, this.drawnItemGeojson, this.selectedAttribute]
           this.censusVariables = Object.keys(resultsData[Object.keys(resultsData)[0]])
           this.selectedCensusVariable = this.censusVariables[0]
           this.loading = false
           this.dialog = false
           this.analysisRun=true
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
          
      
      
           

        })
        .catch((error) => {
          // eslint-disable-next-line
          console.log(error);
         
        });

      }else{
      
      if (this.$refs.form.validate() === true){
        console.log(this.geojson)
        console.log(this.drawnItemGeojson)
      this.loading = true
      this.dialog = true
      this.tab = "tab-2"
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
            this.analysisRun=true
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
          
      
      
           

        })
        .catch((error) => {
          // eslint-disable-next-line
          console.log(error);
         
        });
      }else{
         this.snackbar = true
      }
      }
         }, 

    
       }
    }
</script>
<style>
.overlay {
  width: 350px;
  margin-right: -350px;
  padding-left:10px;
  padding-top:10px;
  position: relative;
  float: left;
  z-index: 2;
  pointer-events: auto;

}
.overlay2 {
 width: 500px;
margin-right: -850px;
margin-left: 340px;
padding-left: 10px;
padding-top: 10px;
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
 
  position: sticky;
  position: -webkit-sticky; /* for Safari */
  top: 56px;
  z-index: 2;

}
.runButtons{
  padding-top:10px;
  padding-left: 15px;
    padding-right: 15px;
}
.toolInset{
  margin:5px;
  top:5px;
  border: 1px solid rgba(63, 81, 181, .5) !important;
  padding-bottom: 15px;
}
.visible{
  display:none
}
.runOptions{
  padding-top:35px;
}
.optionsSelect{
  width:290px;
  margin-left:10px;
  margin-right:10px;
}
.loadCustom {
  width:150px;
}
.runControlDivider{
  padding-top:20px;
}
.loadFileExpansion{
    padding-bottom:20px;
}
.advancedOptions{
 width:200px;
}
</style>