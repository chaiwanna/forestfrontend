<template>
  <div>
    <div class="row">
      <div class="col-md-1"></div>
      <div class="col-md-10">
        <div class="card">
          <h1 class="card-header">รายงานจำนวนคนเข้าป่า</h1>
          <div class="card-body">
            <div class="row">
              <div class="col-xl-12">
                <div class="form-group row">
                  <label for="inputsubdistricts" class="col-sm-2 col-form-label">เลือกช่วงเวลา :</label>
                  <div class="col-sm-4">
                    <date-picker v-model="form.time3" range></date-picker>
                  </div>

                
                </div>

                <!-- <div v-if="user.role === 99">
                  <div class="form-group row">
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
                    <div class="col-sm-1"></div>

                   
                  </div>
                </div>-->
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
                    <b-button
                    class="col-sm-1"
                    type="submit"
                    size="sm"
                    variant="success"
                    v-on:click="loadData()"
                  >
                    <font-awesome-icon :icon="['fas', 'search']" class="mr-1" />ค้นหา
                  </b-button>
                </div>
              </div>
            </div>
            <b-button
              class="col-sm-2"
              type="submit"
              size="sm"
              variant="success"
              v-on:click="download()"
            >
              <font-awesome-icon icon="download" class="mr-1" />ดาวโหลด excel
            </b-button>
            <div class="row">
              <div class="col-md-12">
                <GChart
                  v-if="chartData.length > 1"
                  type="ColumnChart"
                  :data="chartData"
                  :options="chartOptions"
                />
                <h2 v-if="chartData.length <= 1">ไม่พบข้อมูล</h2>
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
import 'vue2-datepicker/locale/es/th';
import { GChart } from 'vue-google-charts';
import reportService from '../../services/reportService';
import forestDetailService from '../../services/forestDetailService';
import userService from '../../services/newUserService';

export default {
  components: {
    GChart,
    DatePicker
  },
  computed: {
    ...mapState('auth', ['user'])
  },
  data() {
    return {
      form: {
        search: null,
        time3: null,
        user_id: null,
        id: 0
      },
      userList: [],
      // Array will be automatically processed with visualization.arrayToDataTable function
      date: new Date(),
      chartData: [['เวลา', 'จำนวน']],
      chartOptions: {
        chart: {
          title: 'จำนวนการเข้าป่า'
          // subtitle: 'Sales, Expenses, and Profit: 2014-2017'
        },
        vAxis: { minValue: 0 }
      },
      forestList: []
    };
  },
  mounted() {
    this.loadForestList();
    this.loadData();
    this.loadUserList();
  },
  methods: {
    download() {
      let a;
      if (this.user.role === 99) {
        reportService.getExcelObj(JSON.stringify(this.createFilter())).then(res => {
          window.open(res);
        });
      } else {
        reportService.getExcelObj(JSON.stringify(this.createFilter())).then(res => {
          window.open(res);
        });
      }
      return a;
    },
    async loadData() {
      const data = await reportService.getGraphObj(this.createFilter());
      console.log(data.data);
      this.chartData = [['วัตถุประสงค์', 'จำนวน']];
      data.data.forEach(element => {
        this.chartData.push([element.name, element.count]);
      });
      console.log(this.chartData);
    },
    createFilter() {
      const filter = {};
      if (this.form.time3) {
        const [one, two] = this.form.time3;
        if (one) {
          const date = new Date(one);
          filter.date_from = `${date.getFullYear()}-${date.getMonth() + 1}-${date.getDate()} 00:00:01`;
        }
        if (two) {
          const date = new Date(two);
          filter.date_too = `${date.getFullYear()}-${date.getMonth() + 1}-${date.getDate()} 23:59:59`;
        }
      }
      if (this.form.user_id) {
        filter.user_id = this.form.user_id;
      }
      if(this.form.id && this.form.id !== 0){
        filter['forest_access.forest_detail_id'] = this.form.id;
      }
      const returnData = { filter };
      return returnData;
    },
    async loadUserList() {
      const data = await userService.getAll();
      this.userList = data.data;
    },
    async loadForestList(){
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
  }
};
</script>

<style scoped>
h1 {
  font-size: 18px;
  font-weight: 600;
  text-align: center;
}
.card-header {
  letter-spacing: 5px;
}
.card-body {
  padding-left: 5%;
  padding-right: 5%;
}
</style>