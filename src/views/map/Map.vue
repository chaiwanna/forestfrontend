<template>
  <div>
    <div class="row">
      <div class="col-md-1"></div>
      <div class="col-md-10">
        <div class="card">
          <h1 class="card-header">แผนที่</h1>
          <div class="card-body">
            <div>
               <div class="row">
              <div class="col-xl-12">

                <div class="form-group row">
                  <label for="inputsubdistricts" class="col-sm-2 col-form-label">เลือกช่วงเวลา :</label>
                  <div class="col-sm-4">
                    <date-picker v-model="form.time3" range></date-picker>
                  </div>
                  
                </div>

                <!-- <div class="form-group row">
                  <label for="inputsubdistricts" class="col-sm-2 col-form-label">ค้นหาตามบุคคล :</label>
                  <div class="col-sm-3">
                    <select v-model="form.user_id" class="form-control">
                      <option
                        v-for="item in userList"
                        :key="item.id"
                        v-bind:value="item.id"
                      >{{item.first_name}} {{item.last_name}}</option>
                    </select>
                  </div>
                  
                </div> -->

                <div class="form-group row">
                  <label for="inputsubdistricts" class="col-sm-2 col-form-label">พื้นที่ :</label>
                  <div class="col-sm-3">
                    <select v-model="form.id" class="form-control">
                      <option
                        v-for="item in forestList"
                        :key="item.id"
                        v-bind:value="item.id"
                      >{{item.name}}</option>
                    </select>
                  </div>
                  <b-button class="col-sm-2" type="submit" size="sm" variant="success" v-on:click="getForest()">
                  <font-awesome-icon :icon="['fas', 'search']" class="mr-1" />ค้นหา
                </b-button>
                  
                </div>

              </div>
              
            </div>
              <div class="container">
                <div class="row justify-content-md-center">
                  <div class="col-xl-12">
                    <div id="app">
                      <longdo-map :location="center" :zoom="10" :lastView="false">
                        <longdo-map-marker
                          v-for="(item, i) in markers"
                          :key="i"
                          :location="item.location"
                          :title="item.title"
                          :detail="item.detail"
                        />
                      </longdo-map>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="col-md-1"></div>
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex';
import DatePicker from 'vue2-datepicker';
import 'vue2-datepicker/index.css';
import 'vue2-datepicker/locale/es/th';
import { LongdoMap, LongdoMapMarker } from 'longdo-map-vue';
import reportService from '../../services/reportService';
import userService from '../../services/newUserService';
import forestDetailService from '../../services/forestDetailService';


export default {
  name: 'map',
  component: {
    'longdo-map': LongdoMap,
    LongdoMapMarker
  },
  components:{
    DatePicker
  },
  computed: {
    ...mapState('auth', ['user'])
  },
  data() {
    return {
      markers: [],
      center: {
        lon: 100,
        lat: 17
      },
      form: {
        search: null,
        time3: null,
        user_id:null,
        id:0
      },
      userList : [],
      forestList : []

    };
  },
  methods: {
    createFilter() {
      const filter = {};
      if (this.form.time3) {
        const [one, two] = this.form.time3;
        if (one) {
          const date = new Date(one);
          filter.date_from = `${date.getFullYear()}-${date.getMonth()+1}-${date.getDate()} 00:00:01`;
        }
        if (two) {
          const date =  new Date(two);
          filter.date_too = `${date.getFullYear()}-${date.getMonth()+1}-${date.getDate()} 23:59:59`;
        }
      }
      if (this.form.user_id) {
          filter.user_id = this.form.user_id;
        
      }
      if(this.form.id && this.form.id !== 0){
        filter.id = this.form.id
      }
      const returnData = { filter };
      return returnData;
    },
    async getForest() {
      this.markers = []
      const map = (await reportService.getMapDetail(this.createFilter())).data;
      map.forEach(element => {
        this.markers.push({
          location: { lon: element.longitude, lat: element.latitude },
          title: element.name,
          detail: `จำนวนคนที่เข้าใช้ ${element.count}`
        });
         this.center= {
        lon: element.longitude,
        lat: element.latitude
      }
      console.log(this.map);
      
      
      });
    },async loadUserList(){
      const data = await userService.getAll();
      this.userList = data.data
    },async loadForestList(){
      const data = await forestDetailService.getAll();
      this.forestList = [];
      this.forestList.push({
        area: null,
        id: 0,
        latitude: 19.198,
        longitude: 100.203,
        name: 'ทั้งหมด'
      });
      (data.data).forEach(element => {
        this.forestList.push(element)
      });
    }
  },
  mounted() {
    this.getForest();
    this.loadForestList();
    this.loadUserList();
  }
};
</script>

<style>
#app {
  width: 100%;
  height: 600px;
}
</style>