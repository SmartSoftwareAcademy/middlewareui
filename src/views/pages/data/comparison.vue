<script>
import Layout from "../../layouts/main";
import PageHeader from "@/components/page-header";
import appConfig from "@/app.config";
import Swal from "sweetalert2";
import mapping from "../../../components/mapping/mapping.vue";
//import VueGoogleAutocomplete from "vue-google-autocomplete";
import axios from "../../../Axiosconfig";
import countTo from "vue-count-to";
import DatePicker from "vue2-datepicker";
import reportdet from "@/components/report/header";
//import Multiselect from "vue-multiselect";
//import vue2Dropzone from "vue2-dropzone";
import "vue2-dropzone/dist/vue2Dropzone.min.css";
/**
 * Invoice-list component
 */
const newheaders = window.$headers;
newheaders["Content-Type"] = "multipart/form-data";
export default {
  page: {
    title: "Data Sync",
    meta: [
      {
        name: "description",
        content: appConfig.description,
      },
    ],
  },
  components: {
    Layout,
    PageHeader,
    countTo,
    DatePicker,
    reportdet,
    mapping,
    //Multiselect,
    //vueDropzone: vue2Dropzone,
  },
  data() {
    return {
      stoped: true,
      showme: true,
      synched: 0,
      waiting: 0,
      failed: 0,
      scheduled: 0,
      title: "KHIS Data",
      items: [
        {
          text: "KHIS Data",
        },
        {
          text: "Data Sync",
          active: true,
        },
      ],
      /**
       * When the location found
       *        */
      //end
      contid: "",
      id: "",
      myindex: "",
      concodance: 0,
      from: new Date(),
      to: new Date(),
      time: new Date().getTime(),
      date_range: "",
      exported: 0,
      status: "0",
      totalrecords: 0,
      records: [],
      series: [
        {
          data: [25, 66, 41, 89, 63, 25, 44, 20, 36, 40, 54],
        },
      ],
      chartOptions: {
        fill: {
          colors: ["#5b73e8"],
        },
        chart: {
          type: "bar",
          sparkline: {
            enabled: true,
          },
        },
        plotOptions: {
          bar: {
            columnWidth: "50%",
          },
        },
        labels: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11],
        xaxis: {
          crosshairs: {
            width: 1,
          },
        },
        tooltip: {
          fixed: {
            enabled: false,
          },
          x: {
            show: false,
          },
          y: {
            title: {
              formatter: function () {
                return "";
              },
            },
          },
          marker: {
            show: false,
          },
        },
      },
      growthChartOptions: {
        fill: {
          colors: ["#f1b44c"],
        },
        chart: {
          type: "bar",
          sparkline: {
            enabled: true,
          },
        },
        plotOptions: {
          bar: {
            columnWidth: "50%",
          },
        },
        labels: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11],
        xaxis: {
          crosshairs: {
            width: 1,
          },
        },
        tooltip: {
          fixed: {
            enabled: false,
          },
          x: {
            show: false,
          },
          y: {
            title: {
              formatter: function () {
                return "";
              },
            },
          },
          marker: {
            show: false,
          },
        },
      },
      orderseries: [70],
      orderRadial: {
        fill: {
          colors: ["#34c38f"],
        },
        chart: {
          sparkline: {
            enabled: true,
          },
        },
        dataLabels: {
          enabled: false,
        },
        plotOptions: {
          radialBar: {
            hollow: {
              margin: 0,
              size: "60%",
            },
            track: {
              margin: 0,
            },
            dataLabels: {
              show: false,
            },
          },
        },
      },
      customerseries: [55],
      customerRadial: {
        fill: {
          colors: ["#5b73e8"],
        },
        chart: {
          sparkline: {
            enabled: true,
          },
        },
        dataLabels: {
          enabled: false,
        },
        plotOptions: {
          radialBar: {
            hollow: {
              margin: 0,
              size: "60%",
            },
            track: {
              margin: 0,
            },
            dataLabels: {
              show: false,
            },
          },
        },
      },
      totalRows: 1,
      total: 0,
      currentPage: 1,
      perPage: 100,
      pageOptions: [
        10, 25, 50, 100, 500, 1000, 1500, 2000, 2500, 3000, 3500, 4000, 4500,
        5000, 5500, 6000,
      ],
      showsync: true,
      filter: null,
      filterOn: [],
      sortBy: "id",
      sortDesc: false,
      editmode: false,
      mappeddata: [],
      index: 1,
      count: 0,
      fields: [
        {
          key: "check",
          label: "#",
        },
        {
          key: "MOH_Indicator_ID",
          sortable: true,
        },
        {
          key: "MOH_Indicator_Name",
          sortable: true,
        },
        {
          key: "shortName",
          sortable: true,
        },
        {
          key: "lastUpdated",
          sortable: true,
        },
        {
          key: "created",
          sortable: true,
        },

        {
          key: "action",
        },
      ],
    };
  },
  created() {
    setInterval(() => {
      if (this.stoped && !this.showsync) {
        this.upadtearray();
      }
    }, 10000);
    //this.upadtearray();
  },
  computed: {
    /**
     * Total no. of records
     */
    rows() {
      return this.records.length;
    },
  },
  watch: {},
  mounted() {
    // Set the initial number of items
    this.upadtearray();
    this.getTotalRecords();
    this.updateDataSync();
    this.totalRows = this.items.length;
  },
  methods: {
    onFiltered(filteredItems) {
      // Trigger pagination to update the number of buttons/pages due to filtering
      this.totalRows = filteredItems.length;
      this.currentPage = 1;
    },
    printpdf(pl) {
      this.pl = pl;
      const data = this.records.map((row) => ({
        "S/NO": this.index++,
        MOH_Indicator_ID: row.MOH_Indicator_ID,
        MOH_Indicator_Name: row.MOH_Indicator_Name,
      }));

      //get headers
      const headers = Object.keys(data[0]);
      const cars = [];
      Object.entries(data).forEach((index, val) => {
        const [key] = index;
        console.log(key, val);
        cars.push(Object.values(data[key]));
      });

      const uniqueCars = Array.from(new Set(cars));
      this.headers = headers;
      this.uniqueCars = uniqueCars;
      this.records = data;
      //alert(headers);
    },
    getrpt() {
      const d = new Date();
      const year = d.getFullYear();
      const month = d.getMonth();
      const date = d.getDate();

      const hour = d.getHours();
      const min = d.getMinutes();
      const sec = d.getSeconds();
      const msec = d.getMilliseconds();
      const filename =
        year +
        "-" +
        month +
        "-" +
        date +
        "-" +
        hour +
        "-" +
        min +
        "-" +
        sec +
        "-" +
        msec;
      //alert(filename);
      const data = this.records.map((row) => ({
        MOH_Indicator_Name: row.name,
        MOH_Indicator_ID: row.id,
        MOH_Disag_Name: "",
        MOH_Disag_ID: "",
        "Disaggregation Type": "Coarse",
      }));
      //alert("");
      const csvRows = [];
      //get headers
      const headers = Object.keys(data[0]);

      csvRows.push(headers.join(","));
      //alert(csvRows);
      //loop over the headers
      for (const row of data) {
        const values = headers.map((header) => {
          const escaped = ("" + row[header]).replace(/"/g, '\\"');
          // alert(escaped);
          return '"' + escaped + '"'; //'" + escaped + "';
        });
        csvRows.push(values.join(","));
      }
      //alert(csvData);
      const csvData = csvRows.join("\n");
      //alert(csvData);

      const blob = new Blob([csvData], { type: "textcsv" });
      const url = window.URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.setAttribute("hidden", "");
      a.setAttribute("href", url);
      a.setAttribute("download", this.title + filename + ".csv");
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
    },
    getmonth(d) {
      const monthNames = [
        "Jan",
        "Feb",
        "Mar",
        "Apr",
        "May",
        "Jun",
        "Jul",
        "Aug",
        "Sep",
        "Oct",
        "Nov",
        "Dec",
      ];
      return monthNames[d];
    },
    getcurrentdate(mydate) {
      let d = new Date(mydate);
      let year = d.getFullYear();
      let month = this.getmonth(d.getMonth());
      let date = d.getDate();
      date = this.getv(date);
      //month = this.getv(month);

      let hour = d.getHours();
      let min = d.getMinutes();
      let sec = d.getSeconds();
      hour = this.getv(hour);
      min = this.getv(min);
      sec = this.getv(sec);

      //const msec = d.getMilliseconds();
      const datetime =
        date + "/" + month + "/" + year + " " + hour + ":" + min + ":" + sec;
      return datetime;
    },
    getv(val) {
      if (val < 10) {
        val = "0" + val;
      }
      return val;
    },
    getTotalRecords() {
      axios
        .get("total_records_count/")
        .then((res) => {
          console.log(res.data);
          this.totalrecords = Number(res.data.totalrecords);
          this.waiting = Number(this.totalrecords - this.synched);
        })
        .catch((e) => {
          console.log(e);
          Swal.fire({
            position: "center",
            icon: "warning",
            title: "Error!",
            html: "" + e,
            showConfirmButton: true,
            timer: 3000,
          });
        });
    },
    upadtearray() {
      axios
        .get("indicators/" + this.perPage)
        .then((res) => {
          console.log(res.data);
          this.records = res.data;
        })
        .then(() => {
          axios
            .get("indicatorscount/")
            .then((res) => {
              console.log(res.data);
              this.synched = Number(res.data.total);
              this.waiting = Number(this.totalrecords - this.synched);
            });
        })
        .catch((e) => {
          console.log(e);
          Swal.fire({
            position: "center",
            icon: "warning",
            title: "Error!",
            html: "" + e,
            showConfirmButton: true,
            timer: 3000,
          });
        });
    },
    updateDataSync() {
      axios
        .get("listmiddleware_settings/1/")
        .then((res) => {
          console.log(res.data);
          this.showsync = res.data.syncdata;
        })
        .catch((e) => {
          console.log(e);
          Swal.fire({
            position: "center",
            icon: "warning",
            title: "Error!",
            html: "" + e,
            showConfirmButton: true,
            timer: 3000,
          });
        });
    },
    formatDate(date) {
      var d = new Date(date),
        month = "" + (d.getMonth() + 1),
        day = "" + d.getDate(),
        year = d.getFullYear();

      if (month.length < 2) month = "0" + month;
      if (day.length < 2) day = "0" + day;

      return [year, month, day].join("-");
    },
    Sync() {
      this.stoped = true;
      this.showsync = false;
      axios
        .put("listmiddleware_settings/1/",
          {
            id: 1,
            syncdata: true,
            client_url: "https://test.hiskenya.org/dhiske/",
          })
        .then((res) => {
          console.log(res.data);
          Swal.fire({
            position: "center",
            icon: "info",
            title: "Success!",
            html: "Data Sync triggered!",
            showConfirmButton: false,
            timer: 2000,
          });
        })
        .catch((e) => {
          console.log(e);
          Swal.fire({
            position: "center",
            icon: "warning",
            title: "Error!",
            html: "" + e,
            showConfirmButton: true,
            timer: 3000,
          });
        });
    },
    Stop() {
      this.stoped = false;
      this.showsync = true;
      axios
        .put("listmiddleware_settings/1/",
          {
            id: 1,
            syncdata: false,
            client_url: "https://test.hiskenya.org/dhiske/",
          })
        .then((res) => {
          console.log(res.data);
          Swal.fire({
            position: "center",
            icon: "info",
            title: "Success!",
            html: "Data Sync stopped!",
            showConfirmButton: false,
            timer: 2000,
          });
        })
        .catch((e) => {
          console.log(e);
          Swal.fire({
            position: "center",
            icon: "warning",
            title: "Error!",
            html: "" + e,
            showConfirmButton: true,
            timer: 3000,
          });
        });
    },

    handleSubmit() {
      console.log("Error on submit");
    },
    increaseCount(n) {
      this.count += n;
      console.log(this.count);
    },
    myRecs({ records, uniqueCars, headers, title, pl, concodance }) {
      console.log(records);
      this.records = records;
      console.log(uniqueCars);
      this.uniqueCars = uniqueCars;
      console.log(headers);
      this.headers = headers;
      this.title = title;
      this.pl = pl;
      this.concodance = concodance;
    },
    search(fromdate, todate) {
      fromdate = this.formatDate(this.from);
      todate = this.formatDate(this.to);
      axios
        .get("indicators/filter/" +
          fromdate +
          "/" +
          todate +
          "/" +
          this.perPage)
        .then((res) => {
          console.log(res.data);
          this.records = res.data;
          this.synched = Number(res.data.length);
          this.stoped = true;
          this.getTotalRecords();
        })
        .catch((e) => {
          console.log(e);
          Swal.fire({
            position: "center",
            icon: "warning",
            title: "Error!",
            html: "" + e,
            showConfirmButton: true,
            timer: 3000,
          });
        });
    },
  },
  middleware: "authentication",
  //    capacity = models.IntegerField()
  //     location = models.PointField(blank=True, null=True)
  //     name = models.CharField(max_length=120)
  //     category = models.CharField(max_length=20,choices=CHOICES_TYPE, default=1)
};
</script>

<template>
  <Layout>
    <PageHeader :title="title" :items="items" />

    <div class="row">
      <div class="col-md-12">
        <div class="row">
          <div class="col-md-6 col-xl-3">
            <div class="card">
              <div class="card-body">
                <div class="float-end mt-2">
                  <apexchart class="apex-charts" dir="ltr" width="70" height="40" :options="chartOptions"
                    :series="series"></apexchart>
                </div>
                <div>
                  <h4 class="mb-1 mt-1">
                    <span data-plugin="counterup">
                      <countTo :startVal="1" :endVal="synched" :duration="1"></countTo>
                    </span>
                  </h4>
                  <p class="text-muted mb-0">Synced</p>
                </div>
                <p class="text-muted mt-3 mb-0">
                  <span class="text-success me-1" :class="{
                    'text-danger': waiting > synched,
                  }">
                    <i class="mdi mdi-arrow-down-bold me-1" :class="{
                      'mdi mdi-arrow-up-bold': waiting < synched,
                    }"></i>{{ ((synched / totalrecords) * 100).toFixed(2) }}%
                  </span>
                  since last week
                </p>
              </div>
            </div>
          </div>
          <div class="col-md-6 col-xl-3">
            <div class="card">
              <div class="card-body">
                <div class="float-end mt-2">
                  <apexchart class="apex-charts" type="radialBar" dir="ltr" width="45" height="45" :options="orderRadial"
                    :series="orderseries"></apexchart>
                </div>
                <div>
                  <h4 class="mb-1 mt-1">
                    <span data-plugin="counterup">
                      <countTo :startVal="1" :endVal="waiting" :duration="1"></countTo>
                    </span>
                  </h4>
                  <p class="text-muted mb-0">Waiting</p>
                </div>
                <p class="text-muted mt-3 mb-0">
                  <span class="text-success me-1" :class="{
                    'text-danger': waiting < synched,
                  }">
                    <i class="mdi mdi-arrow-down-bold me-1" :class="{
                      'mdi mdi-arrow-up-bold': waiting > synched,
                    }"></i>{{ ((waiting / totalrecords) * 100).toFixed(2) }}%
                  </span>
                  since last week
                </p>
              </div>
            </div>
          </div>
          <!-- end col-->

          <div class="col-md-6 col-xl-3">
            <div class="card">
              <div class="card-body">
                <div class="float-end mt-2">
                  <apexchart class="apex-charts" type="radialBar" dir="ltr" width="45" height="45"
                    :options="customerRadial" :series="customerseries"></apexchart>
                </div>
                <div>
                  <h4 class="mb-1 mt-1">
                    <span data-plugin="counterup">
                      <countTo :startVal="1000" :endVal="0" :duration="2000"></countTo>
                    </span>
                  </h4>
                  <p class="text-muted mb-0">Failed</p>
                </div>
                <p class="text-muted mt-3 mb-0">
                  <span class="text-danger me-1">
                    <i class="mdi mdi-arrow-down-bold me-1"></i>0.00%
                  </span>
                  since last week
                </p>
              </div>
            </div>
          </div>
          <div class="col-md-6 col-xl-3">
            <div class="card">
              <div class="card-body">
                <div class="float-end mt-2">
                  <apexchart class="apex-charts" dir="ltr" width="70" height="40" :options="growthChartOptions"
                    :series="series"></apexchart>
                </div>
                <div>
                  <h4 class="mb-1 mt-1">
                    <span data-plugin="counterup">
                      <countTo :startVal="1000" :endVal="0" :duration="2000"></countTo>
                    </span>
                  </h4>
                  <p class="text-muted mb-0">Scheduled</p>
                </div>
                <p class="text-muted mt-3 mb-0">
                  <span class="text-danger me-1">
                    <i class="mdi mdi-arrow-down-bold me-1"></i>0.00%
                  </span>
                  since last week
                </p>
              </div>
            </div>
          </div>
          <!-- end col-->
        </div>
      </div>
    </div>
    <div class="row">
      <div class="col-sm-12 col-md-6">
        <div id="tickets-table_length" class="dataTables_length">
          <label class="d-inline-flex align-items-center fw-normal">
            Show&nbsp;
            <b-form-select v-model="perPage" size="sm" :options="pageOptions"></b-form-select>&nbsp;entries
          </label>
        </div>
      </div>
      <!-- Search -->
      <div class="col-sm-12 col-md-6">
        <div id="tickets-table_filter" class="dataTables_filter text-md-end">
          <label class="d-inline-flex align-items-center fw-normal">
            Search:
            <b-form-input v-model="filter" type="search" class="form-control form-control-sm ms-2"></b-form-input>
          </label>
        </div>
      </div>
      <!-- End search -->
    </div>

    <div class="row">
      <div class="col-sm-6 col-md-2">
        <div id="tickets-table-date-picker" class="dataTables_length">
          <label class="d-inline-flex align-items-center fw-normal">
            From&nbsp;
            <date-picker v-model="from" type="date"></date-picker>
          </label>
        </div>
      </div>
      <div class="col-sm-6 col-md-2">
        <div id="tickets-table-date-picker" class="dataTables_length">
          <label class="d-inline-flex align-items-center fw-normal">
            To&nbsp;
            <date-picker v-model="to" type="date"></date-picker>
          </label>
        </div>
      </div>
      <div class="col-sm-6 col-md-2">
        <div id="tickets-table-date-picker" class="dataTables_length">
          <label class="d-inline-flex align-items-center fw-normal">
            Range&nbsp;
            <date-picker v-model="time_range" range></date-picker>
          </label>
        </div>
      </div>
      <div class="col-sm-6 col-md-2">
        <div id="tickets-table-date-picker" class="dataTables_length">
          <label class="d-inline-flex align-items-center fw-normal">
            <button class="btn btn-secondary waves-effect waves-light uil-search-alt text-white outline-dark"
              @click="search()">
              Search
            </button>
          </label>
        </div>
      </div>
      <div class="col-sm-6 col-md-2">
        <b-button variant="outline-dark bg-secondary text-white uil uil-sync" @click="Sync()" v-show="showsync">Sync
          Data</b-button>
        <b-button variant="outline-dark bg-danger text-white uil uil-stop-circle" @click="Stop()" v-show="!showsync">Stop
          Sync</b-button>
      </div>
    </div>
    <div ref="content">
      <div class="card">
        <div class="card-body">
          <form @submit.prevent="handleSubmit">
            <div class="card">
              <div class="card-body">
                <div class="row justify-content-between">
                  <div class="col-sm-6">
                    <button class="btn btn-secondary waves-effect waves-light uil-export btn-outline-dark"
                      @click="getrpt()">
                      Export to CSV
                    </button>
                  </div>

                  <div class="col-sm-2">
                    <button @click="printpdf('p')" v-b-modal.modal-Print
                      class="btn btn-secondary waves-effect waves-light uil-file-alt outline-dark">
                      Print PDF
                    </button>
                  </div>
                  <div class="col-sm-2">
                    <button v-b-modal.data-mapping
                      class="btn btn-secondary waves-effect waves-light uil-database-alt outline-dark"
                      @click="clearvalues()">
                      Indicator Mapping
                    </button>
                  </div>
                </div>
              </div>
            </div>
            <div class="row">
              <div class="col-sm-12">
                <div class="card">
                  <div class="card-body changebg">
                    <div class="row" id="print">
                      <div class="col-12">
                        <div>
                          <div class="float-end">
                            <form class="d-inline-flex mb-3"></form>
                          </div>
                        </div>
                        <div
                          class="table table-centered datatable dt-responsive nowrap table-card-list dataTable no-footer dtr-inline">
                          <div class="row">
                            <div class="col-sm-12 col-md-6">
                              <div id="tickets-table_length" class="dataTables_length">
                                <label class="d-inline-flex align-items-center fw-normal">
                                  Show&nbsp;
                                  <b-form-select v-model="perPage" size="sm"
                                    :options="pageOptions"></b-form-select>&nbsp;entries
                                </label>
                              </div>
                            </div>
                            <!-- Search -->
                            <div class="col-sm-12 col-md-6">
                              <div id="tickets-table_filter" class="dataTables_filter text-md-end">
                                <label class="d-inline-flex align-items-center fw-normal">
                                  Search:
                                  <b-form-input v-model="filter" type="search" placeholder="Search..."
                                    class="form-control form-control-sm ms-2"></b-form-input>
                                </label>
                              </div>
                            </div>
                            <!-- End search -->
                          </div>
                          <!-- Table -->

                          <b-table table-class="table table-centered datatable table-card-list"
                            thead-tr-class="bg-transparent" :items="records" :fields="fields" responsive="sm"
                            :per-page="perPage" :current-page="currentPage" :sort-by.sync="sortBy"
                            :sort-desc.sync="sortDesc" :filter="filter" :filter-included-fields="filterOn"
                            @filtered="onFiltered">
                            <template v-slot:cell(action)>
                              <ul class="list-inline mb-0">
                                <li class="list-inline-item">
                                  <a href="javascript:void(0);" class="px-2 text-primary" v-b-tooltip.hover title="Edit"
                                    v-b-modal.modal-Edit>
                                    <i class="uil uil-pen font-size-18"></i>
                                  </a>
                                </li>
                                <li class="list-inline-item">
                                  <a href="javascript:void(0);" class="px-2 text-danger" v-b-tooltip.hover title="Delete">
                                    <i class="uil uil-trash-alt font-size-18"></i>
                                  </a>
                                </li>
                              </ul>
                            </template>
                          </b-table>
                        </div>
                        <div class="row">
                          <div class="col">
                            <div class="dataTables_paginate paging_simple_numbers float-end">
                              <ul class="pagination pagination-rounded">
                                <!-- pagination -->
                                <b-pagination v-model="currentPage" :total-rows="rows" :per-page="perPage"></b-pagination>
                              </ul>
                            </div>
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </form>
        </div>
      </div>
    </div>
    <!--modals-->
    <b-modal id="modal-Print" title="Print PDF" hide-footer size="bg" centered>
      <reportdet :title="title" :records="records" :pl="pl" :headers="headers" :uniqueCars="uniqueCars" :shome="showme"
        v-show="showme" :concodance="concodance"></reportdet>
    </b-modal>
    <b-modal id="data-mapping" title="Data Mapping" header-class="bg-secondary bg-gradient bg-opacity-50" hide-footer
      modal-class="" size="xl" centered>
      <mapping @myrecords="myRecs" />
    </b-modal>
    <!--end modals-->
    <div class="row">
      <div class="col">
        <div class="dataTables_paginate paging_simple_numbers float-end">
          <ul class="pagination pagination-rounded">
            <!-- pagination -->
            <b-pagination v-model="currentPage" :total-rows="rows" :per-page="perPage"></b-pagination>
          </ul>
        </div>
      </div>
    </div>
  </Layout>
</template>
