<template>
  <div>
    <v-card>
      <div>
        <v-card flat>
          <v-row>
            <v-col class="float-left" cols="2">
              <div class="ml-5">
                <v-menu offset-y>
                  <template v-slot:activator="{ on, attrs }">
                    <div>
                      <v-btn
                        @click="isEmpty(todelivered)"
                        class="float-left"
                        outlined
                        color="purple"
                        v-bind="attrs"
                        v-on="on"
                      >
                        <v-icon>mdi-download</v-icon>Export
                      </v-btn>
                    </div>
                  </template>
                  <v-list v-show="is_empty === false">
                    <v-col>
                      <OrderToDeliverPdf :headers="headers" :todelivered="todelivered"></OrderToDeliverPdf>
                      <div>
                        <download-csv :data="todelivered" name="Deliveries.csv">
                          <v-btn text small>Export as CSV</v-btn>
                        </download-csv>
                      </div>
                    </v-col>
                  </v-list>
                </v-menu>
              </div>
            </v-col>
          </v-row>
        </v-card>
      </div>
      <br>
      <v-data-table
        :headers="headers"
        :items="todelivered"
        :items-per-page="10"
        class="elevation-1"
      >
        <template v-slot:item.preferred_delivery_date="{ item }">
          <span>{{new Date(item.preferred_delivery_date).toISOString().substring(0,10)}}</span>
        </template>
         <template v-slot:item.distance="{ item }">
              <span>{{item.distance+'km'}}</span>
            </template>
        <template v-slot:item.contact_number="{ item }">
          <span>{{'0'+item.contact_number}}</span>
        </template>
        <template v-slot:item.order_status="{ item }">
          <v-chip :color="getColor(item.order_status)" dark>{{ item.order_status }}</v-chip>
        </template>
        <!-- <template v-slot:item.action="{ item }">
          <div v-if="isAdmin() === true">
            <div v-if="isCanceled(item) === true || isDelivered(item) === true">
              <v-icon
                disabled
                normal
                class="mr-2"
                title="Delivered"
                @click="alertDelivered(item)"
              >mdi-truck-check-outline</v-icon>
              <v-icon
                disabled
                @click="alertCancel(item)"
                normal
                class="mr-2"
                title="Cancel"
              >mdi-cancel</v-icon>
            </div>
            <div v-else>
              <v-icon
                disabled
                normal
                class="mr-2"
                title="Delivered"
                @click="alertDelivered(item)"
              >mdi-truck-check-outline</v-icon>
              <v-icon @click="alertCancel(item)" normal class="mr-2" title="Cancel">mdi-cancel</v-icon>
            </div>
          </div>
          <div v-if="isRider() === true">
            <div v-if="isCanceled(item) === true || isDelivered(item) === true">
              <v-icon
                disabled
                normal
                class="mr-2"
                title="Delivered"
                @click="alertDelivered(item)"
              >mdi-truck-check-outline</v-icon>
              <v-icon
                disabled
                @click="alertCancel(item)"
                normal
                class="mr-2"
                title="Cancel"
              >mdi-cancel</v-icon>
            </div>
            <div v-else>
              <v-icon
                normal
                class="mr-2"
                title="Delivered"
                @click="alertDelivered(item)"
              >mdi-truck-check-outline</v-icon>
              <v-icon
                disabled
                @click="alertCancel(item)"
                normal
                class="mr-2"
                title="Cancel"
              >mdi-cancel</v-icon>
            </div>
          </div>
        </template>-->
      </v-data-table>
    </v-card>
  </div>
</template>

<script>
import Vue from "vue";
import JsonCSV from "vue-json-csv";
import axios from "axios";
import Swal from "sweetalert2";
import OrderToDeliverPdf from "./OrderToDeliverPdf.vue";

Vue.component("downloadCsv", JsonCSV);

export default {
  components: { OrderToDeliverPdf },
  data() {
    return {
      todelivered: [],
      // deliveriesExcel: [],
      is_empty: false,
      headers: [
        {
          text: "Receiver Name",
          align: "start",
          sortable: false,
          value: "receiver_name"
        },
        { text: "Address", value: "customer_address", sortable: false },
        { text: "Mobile Number", value: "contact_number", sortable: false },
        { text: "Distance", value: "distance" },
        {
          text: "Delivery Date",
          value: "preferred_delivery_date",
          sortable: false
        },
        {
          text: "Ube Halaya Jar(Quantity)",
          value: "ubehalayajar_qty",
          sortable: false
        },
        {
          text: "Ube Halaya Tub(Quantity)",
          value: "ubehalayatub_qty",
          sortable: false
        },
        {
          text: "Total Item",
          value: "total_item",
          sortable: false
        },
        {
          text: "Total Payment",
          value: "total_payment",
          sortable: false
        },
        // { text: "Actions", value: "action", sortable: false },
        { text: "Status", value: "order_status" }
      ]
    };
  },
  beforeCreate() {
    let config = {};
    config.headers = {
      Authorization: "Bearer " + localStorage.getItem("token"),
      "Access-Control-Allow-Origin": "*"
    };
    this.config = config;
  },
  created() {
    this.getToDelivered();
    // this.getDeliveries();
  },
  methods: {
    getColor(status) {
      if (status === "Canceled") return "orange";
      else if (status === "On order") return "blue";
      else return "green";
    },
    getToDelivered() {
      axios
        .get(this.url + "/api/posts/delivery", this.config)
        .then(response => {
          this.todelivered = response.data.data;
          this.todelivered.forEach((order, index) => {
            let {
              building_or_street,
              barangay,
              city_or_municipality,
              province,
              ubehalayajar_qty,
              ubehalayatub_qty
            } = order;
            var place = building_or_street
              .toString()
              .concat(" ", barangay, " ", city_or_municipality, " ", province);
            this.todelivered[index]["customer_address"] = place;
            this.todelivered[index]["total_item"] =
              ubehalayatub_qty + ubehalayajar_qty;
          });
          console.log("-->>>this,todelivered", this.todelivered);
        });
    },
    // getDeliveries() {
    //   axios
    //     .get(this.url + "/api/getDeliveries", this.config)
    //     .then(response => {
    //       setTimeout(() => {
    //         this.$vloading.hide();
    //       }, 1000);
    //       this.deliveriesExcel = response.data.data;
    //     });
    // },
    isEmpty(todelivered) {
      if (todelivered.length == 0) {
        this.is_empty = true;
        Swal.fire({
          position: "center",
          icon: "warning",
          title: "Cannot be Downloaded. No Data Available",
          showConfirmButton: true
        });
      } else {
        this.is_empty = false;
      }
    }
    // isAdmin() {
    //   if (localStorage.getItem("role") === "admin") {
    //     return true;
    //   }
    // },
    // isRider() {
    //   if (localStorage.getItem("role") === "driver") {
    //     return true;
    //   }
    // },
    // isCanceled(item) {
    //   if (item.order_status == "Canceled") {
    //     return true;
    //   }
    // },
    // isDelivered(item) {
    //   if (item.order_status == "Delivered") {
    //     return true;
    //   }
    // },
    // alertDelivered(item) {
    //   Swal.fire({
    //     title: "Are you sure item is being delivered?",
    //     icon: "warning",
    //     showCancelButton: true,
    //     confirmButtonColor: "#3085d6",
    //     cancelButtonColor: "#CFD8D",
    //     cancelButtonText: "No",
    //     confirmButtonText: "Yes",
    //     reverseButtons: true
    //   }).then(result => {
    //     if (result.value) {
    //       this.deliveredItem(item);
    //       this.getToDelivered();
    //     }
    //   });
    // },

    // deliveredItem(item) {
    //   axios
    //     .post(this.url + "/api/post/updateStat/" + item.id, {}, this.config)
    //     .then(response => {
    //       Swal.fire({
    //         title: "Order has been delivered",
    //         icon: "success",
    //         showConfirmButton: false,
    //         timer: 1500
    //       });
    //       this.getToDelivered();
    //     });
    // },
    // alertCancel(item) {
    //   Swal.fire({
    //     title: "Are you sure?",
    //     icon: "warning",
    //     showCancelButton: true,
    //     confirmButtonColor: "#3085d6",
    //     cancelButtonColor: "#CFD8D",
    //     cancelButtonText: "No",
    //     confirmButtonText: "Yes",
    //     reverseButtons: true
    //   }).then(result => {
    //     if (result.value) {
    //       this.deleteItem(item);
    //       this.getToDelivered();
    //     }
    //   });
    // },
    // deleteItem(item) {
    //   axios
    //     .post(
    //       this.url + "/api/post/updateCanceledStat/" + item.id,
    //       {},
    //       this.config
    //     )
    //     .then(response => {
    //       Swal.fire({
    //         title: "Canceled!",
    //         text: "Order has been canceled",
    //         icon: "success",
    //         showConfirmButton: false,
    //         timer: 1500
    //       });
    //       this.getToDelivered();
    //     });
    // },
  }
};
</script>
