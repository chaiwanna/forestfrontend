<template>
  <div>
    <div class="row">
      <div class="col-md-1"></div>
      <div class="col-md-10">
        <div class="card">
          <h1 class="card-header">Dashboard</h1>
          <div class="card-body">
            <div class="page-class page-home">
              <div class="row justify-content-md-center">
                <h1>{{date.day}} {{month[date.month]}} {{date.year}}</h1>
              </div>
               <div class="form-group row">
                <label for="inputsubdistricts" class="col-sm-1 col-form-label">พื้นที่ :</label>
                <div class="col-sm-2">
                  <select v-model="form.id" class="form-control">
                    <option
                      v-for="item in forestList"
                      :key="item.id"
                      v-bind:value="item.id"
                    >{{item.name}}</option>
                  </select>
                </div>
              </div>
              <!-- card section -->
              <div class="row">
                <div class="col-xl-3 col-sm-6 mb-3">
                  <div class="card text-white bg-primary o-hidden h-100">
                    <div class="card-body">
                      <div class="card-body-icon">
                        <font-awesome-icon icon="user-circle" />
                      </div>
                      <div class="mr-5">
                        <h3>จำนวนคนเข้าป่า</h3>
                      </div>
                      <div class="mr-5">
                        <h1>{{dashboard.dashboard_access_per_day}} คนทั้งหมด</h1>
                      </div>
                    </div>
                    <router-link
                      class="card-footer text-white clearfix small z-1"
                      :to="{path : '/report'}"
                    >
                      <span class="float-left">View Details</span>
                      <span class="float-right">
                        <i class="fa fa-angle-right"></i>
                      </span>
                    </router-link>
                  </div>
                </div>
                <div class="col-xl-3 col-sm-6 mb-3">
                  <div class="card text-white bg-warning o-hidden h-100">
                    <div class="card-body">
                      <div class="card-body-icon">
                        <font-awesome-icon icon="tree" />
                      </div>
                      <div class="mr-5">
                        <h3>จำนวนป่าที่ดูแล</h3>
                      </div>
                      <div class="mr-5">
                        <h1>{{dashboard.dashboard_forest_care}} พื้นที่</h1>
                      </div>
                    </div>
                    <router-link
                      class="card-footer text-white clearfix small z-1"
                      :to="{path : '/map'}"
                    >
                      <span class="float-left">View Details</span>
                      <span class="float-right">
                        <i class="fa fa-angle-right"></i>
                      </span>
                    </router-link>
                  </div>
                </div>
              </div>
             
              <div class="row">
                <template v-for="(item, index) in dashboard.dashboard_access_forest_detail">
                  <span :key="index" class="col-xl-3 col-sm-6 mb-3">
                    <div>
                      <div class="card text-white bg-success o-hidden h-100">
                        <div class="card-body">
                          <div class="card-body-icon">
                            <font-awesome-icon icon="tree" />
                          </div>
                          <div class="mr-5">
                            <h3>บริเวณโซนหมู่บ้าน {{item.name}}</h3>
                          </div>
                          <div class="mr-5">
                            <h1>{{item.count}} คน</h1>
                          </div>
                        </div>
                        <div
                          class="card-footer text-white clearfix small z-1"
                          v-on:click="changeSelectQR(item.id)"
                          v-b-modal.modal-1
                        >
                          <span class="float-left">ดาวน์โหลด QRcode</span>
                          <span class="float-right">
                            <font-awesome-icon :icon="['fas','qrcode']" />
                          </span>
                        </div>
                      </div>
                    </div>
                  </span>
                </template>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="col-md-1"></div>
    </div>
    <b-modal
      id="modal-1"
      title="QR Code"
      hide-footer
      header-bg-variant="success"
      header-text-variant="light"
    >
      <div class="row justify-content-md-center">
      <qrcode
        :value="`${configService.get('appUrl')}/quick_access?forest_id=${selectQR}`"
        :options="{ width: 200 }"
      ></qrcode>
      </div>
    </b-modal>
  </div>
</template>

<script>
import forestDetailService from '../services/forestDetailService';
import reportService from '../services/reportService';

export default {
  name: 'Home',
  components: {},
  metaInfo: {
    title: 'Home',
    meta: [
      {
        name: 'description',
        content: 'Welcome to Vue.js backend.'
      }
    ]
  },
  data() {
    return {
      dashboard: {
        dashboard_access_per_day: 0,
        dashboard_forest_care: 0,
        dashboard_access_forest_detail: []
      },
      month: [
        'มกราคม',
        'กุมภาพันธ์',
        'มีนาคม',
        'เมษายน',
        'พฤษภาคม',
        'มิถุนายน',
        'กรกฎาคม',
        'สิงหาคม',
        'กันยายน',
        'ตุลาคม',
        'พฤศจิกายน',
        'ธันวาคม'
      ],
      date: {
        day: new Date().getDate(),
        month: new Date().getMonth(),
        year: new Date().getFullYear() + 543
      },
      selectQR: 1,
      forestList: [],
      form: {
        id: 0
      }
    };
  },
  methods: {
    createFilter() {
      const filter = {};
      if (this.form.id && this.form.id !== 0) {
        filter['forest_detail.id'] = this.form.id;
      }
      const returnData = { filter };
      return returnData;
    },
    async loadForestList() {
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
    },
    async getDashboard() {
      const filter = this.createFilter();
      console.log(filter);

      function isEmptyObject(obj) {
        return JSON.stringify(obj) === '{}';
      }
      if (!isEmptyObject(filter.filter)) {
        this.dashboard = (await reportService.getReport(JSON.stringify(this.createFilter()))).data;
        return;
      }
      this.dashboard = (await reportService.getReport()).data;
    },
    changeSelectQR(id) {
      this.selectQR = id;
    }
  },
  mounted() {
    this.loadForestList();
    this.getDashboard();
  },
  watch: {
    'form.id': function() {
      this.getDashboard();
    }
  }
};
</script>

<style>
.card-body-icon {
  position: absolute;
  z-index: 0;
  top: -0px;
  right: 10px;
  font-size: 5rem;
  /* -webkit-transform: rotate(15deg); */
  /* -ms-transform: rotate(15deg); */
  /* transform: rotate(15deg); */
}

@media (min-width: 576px) {
  .card-columns {
    column-count: 1;
  }
}

@media (min-width: 768px) {
  .card-columns {
    column-count: 2;
  }
}

@media (min-width: 1200px) {
  .card-columns {
    column-count: 2;
  }
}
</style>
