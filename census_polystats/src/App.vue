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
    </v-app-bar>



    <v-main class="grey lighten-3">
      <v-container>
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
        <v-tabs
          v-model="tab"
          align-with-title
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
  
  <v-tabs-items v-model="tab">
      <v-tab-item :key="analysis">
        <v-row>
          <v-col cols="6">
            <v-sheet rounded="lg">
              <v-form
                ref="form"
                v-model="valid"
                lazy-validation
              >
              <v-list color="transparent">
                <v-list-item
                  link
                  color="grey lighten-4"
                >
                  <v-list-item-content>
                    <v-list-item-title>
                      Add a boundary shapefile
                    </v-list-item-title>
                      <v-file-input
                      show-size
                      :rules="rulesUpload"
                      accept=".zip,.geojson"
                      placeholder="Pick an zipped shapefile"
                      prepend-icon="mdi-map"
                      label="Shapefile"
                      v-model="chosenFile"
                      @change="onAddFiles"
                      required
                    ></v-file-input>
                  </v-list-item-content>
                </v-list-item>

                 <v-list-item
                  link
                  color="grey lighten-4"
                >
                  <v-list-item-content>
                    <v-list-item-title>
                      Select summary attribute
                    </v-list-item-title>
                   
                      <v-select
                        v-model="selectedAttribute"
                        :items="attributes"
                        label="Summary Attribute"
                        outlined
                        :rules="rules"
                        required
                      ></v-select>
                   
                  </v-list-item-content>
                </v-list-item>
                
               
               

                <v-divider class="my-2"></v-divider>

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
          </v-col>

          <v-col cols = "6">
            <v-sheet
              min-height="70vh"
              rounded="lg"
            >
              <LeafletMap :name="geojson" :selectedAttribute="selectedAttribute"
              > </LeafletMap> 
               
            </v-sheet>
          </v-col>
         
        </v-row>
         </v-tab-item>
          <v-tab-item :key="results">
               <v-col cols="8" v-if= "showResults">
                 <LeafletMapResults :name="geojson" :resultsData="resultsData" :selectedAttribute="selectedAttribute"
              > </LeafletMapResults> 
               <v-card
              
              max-width="344"
              outlined
            >
              <v-list-item three-line>
                <v-list-item-content>
                  <div class="text-overline mb-4">
                    OVERLINE
                  </div>
                  <v-list-item-title class="text-h5 mb-1">
                    Headline 5
                  </v-list-item-title>
                  <v-list-item-subtitle>Greyhound divisely hello coldly fonwderfully</v-list-item-subtitle>
                </v-list-item-content>

                <v-list-item-avatar
                  tile
                  size="80"
                  color="grey"
                ></v-list-item-avatar>
              </v-list-item>

              <v-card-actions>
                <v-btn
                  outlined
                  rounded
                  text
                >
                  Button
                </v-btn>
              </v-card-actions>
             
            </v-card>
          </v-col>
          </v-tab-item> 
        </v-tabs-items>
        
      </v-container>
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
      //this.tab = 1
      //this.getTigerPolygons()
      let bbox = turf.bbox(this.geojson);
      
      const path = 'http://localhost:5000/basicAnalysis';
      let payload = {"layer": this.geojson, "bbox":bbox, "summary_attribute": this.selectedAttribute}
      console.log(payload)
      axios.post(path, payload)
        .then((res) => {
           this.tab = 1
           this.showResultsTab =true
           this.resultsData = res.data.data
           this.loading = false
           

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
