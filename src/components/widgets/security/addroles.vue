<script>
import Swal from "sweetalert2";
import axios from "../../../Axiosconfig";
//import html2canvas from "html2canvas";
//import rptheader from "@/components/report/header.js";

export default {
    props: {
        selectedlists: Array,
        rname: String,
        editmode: Boolean,
        id: Number,
    },
    components: {},
    data() {
        return {
            title: "",
            items: [
                {
                    text: "USER: " + JSON.parse(localStorage.user).username,
                },
                {
                    text: "Role List",
                    active: true,
                },
            ],
            addroleslog: false,
            username: "titus.owuor@tdbsoft.co.ke",
            filter: null,
            filterOn: [],
            sortBy: "id",
            sortDesc: true,
            name: "",
            myindex: "",
            modaltitle: "Add",
            lists: [
                "Dashboard",
                "DataAlignment",
                "IndicatorMappingRules",
                "IndicatorComparison",
                "FacilityMapping",
                "DataQuality",
                "Indicators",
                "AllIndicators",
                "Categories",
                "Security",
                "DataPullSchedule",
                "Roles",
                "Users",
                "PasswordPolicy",
                "Reports",
            ],
        };
    },
    watch: {
        addroleslog(newValue) {
            var curentuser = JSON.parse(localStorage.user).email
            var currentdate = new Date();
            const data = {
                datetime: currentdate,
                useremail: curentuser,
                application: window.navigator.userAgent,
                details: `Add Roles:${newValue},\nDetails(role name:${this.rname},\ndescription:${this.description},\nscreens[${this.selectedlists}])`,
                computer: window.localStorage.clientip
            }
            axios
                .post(window.$http + "AuditLogs", data, { headers: window.$headers })
                .then(response => {
                    console.log(response.data)
                })
                .catch(e => {
                    console.log(e)
                })
        }
    },
    computed: {
        /**
         * Total no. of records
         */
        rows() {
            return this.orderData.length;
        },
    },
    mounted() {
        // Set the initial number of items
        this.focusInput();
        this.totalRows = this.items.length;
        this.fetchscreens();
    },
    methods: {
        focusInput() {
            this.$refs.rname.focus();
        },
        gettime() {
            //2021-06-11T09:05:53.07
            //2021-5-11T12:54.48
            var DateNow = new Date();
            var DateNowString;
            DateNow.setDate(DateNow.getDate() + 20);
            DateNowString =
                DateNow.getFullYear() +
                "-" +
                ("0" + (DateNow.getMonth() + 1)).slice(-2) +
                "-" +
                ("0" + DateNow.getDate()).slice(-2) +
                "T" +
                ("0" + DateNow.getHours()).slice(-2) +
                ":" +
                ("0" + DateNow.getMinutes()).slice(-2) +
                ":" +
                ("0" + DateNow.getSeconds()).slice(-2) +
                "." +
                ("0" + DateNow.getMilliseconds()).slice(-2);
            return DateNowString;
        },

        /**
         * Search the table data with search input
         */
        onFiltered(filteredItems) {
            // Trigger pagination to update the number of buttons/pages due to filtering
            this.totalRows = filteredItems.length;
            this.currentPage = 1;
        },
        handleSubmit() {
            //alert(this.rname);
            //console.log("Error on submit");
        },
        add() {
            //var timenow = this.gettime();
            //console.log(timenow);
            if (this.rname.trim() == "") {
                Swal.fire("Please enter Role Name!", {
                    className: "alert-custom",
                });
                return;
            }
            axios
                .post("listroles/", {
                    id: 0,
                    role_name: this.rname,
                    screens: this.selectedlists.toString(),
                    faclities: this.faclities,
                })
                .then((response) => {
                    //this.screenlist = response;
                    console.log(response.data);
                    this.addroleslog = true;
                    Swal.fire({
                        position: "center",
                        icon: "success",
                        title: "Your work has been saved",
                        showConfirmButton: false,
                        timer: 1500,
                    }).then((result) => {
                        result;
                        this.clearvalues();
                        var orderid = this.orderData.length + 1;
                        this.orderData.push({
                            id: orderid,
                            name: this.rname,
                            screens: this.selectedlists,
                        });
                    });
                })
                .catch(function (error) {
                    Swal.fire({
                        position: "center",
                        icon: "error",
                        title: "Error on submmission. Check Servers.",
                        showConfirmButton: false,
                        timer: 1500,
                    }).then((result) => {
                        result;
                        this.clearvalues();
                    });
                    console.log(error);
                });
        },
        editrec() {
            if (this.selectedlist.trim() == "") {
                Swal.fire("Please select screens!");
                return;
            }
            axios
                .put("listroles/" + this.id + "/", {
                    role_id: this.id,
                    screens: this.selectedlists.toString(),
                    faclities: this.faclities,
                })
                .then((response) => {
                    //this.screenlist = response;
                    this.addroleslog = true;
                    Swal.fire({
                        position: "center",
                        icon: "success",
                        title: "Your work has been saved",
                        showConfirmButton: false,
                        timer: 3000,
                    }).then((result) => {
                        result;
                        this.clearvalues();
                    });
                    console.log(response);
                })
                .catch(function (error) {
                    Swal.fire({
                        position: "center",
                        icon: "error",
                        title: "Error on submmission. Check Servers.",
                        showConfirmButton: false,
                        timer: 1500,
                    }).then((result) => {
                        result;
                        this.clearvalues();
                    });
                    console.log(error);
                });
            Swal.fire({
                position: "center",
                icon: "success",
                title: "Your work has been saved",
                showConfirmButton: false,
                timer: 1500,
            });
        },
        deleterec(index, id, name) {
            //alert(rolename);
            this.id = id;
            this.name = name;
            Swal.fire({
                title: "Are you sure, you want to delete " + this.name + "?",
                text: "You won't be able to revert this!",
                icon: "warning",
                showCancelButton: true,
                confirmButtonColor: "#000000",
                cancelButtonColor: "#f46a6a",
                confirmButtonText: "Yes, delete it!",
            }).then((result) => {
                if (result.value) {
                    this.orderData.splice(index, 1);
                    //this.$delete(this.orderData, this.id - 1);
                    Swal.fire("Deleted!", this.name + " has been deleted.", "success");
                }
            });
        },
        clearvalues() {
            this.rname = "";
            this.allToLeft();
            this.selectedlists = [];
        },
        oneToRight() {
            // if (this.rname.trim() == "") {
            //   Swal.fire("Please enter Role Name!", {
            //     className: "alert-custom",
            //   });
            //   return;
            // }
            //console.log("allToRight");

            var select = document.getElementById("lists").value;
            //alert(select);
            //var res = select.split("-");

            //select = res[1];
            //var indexselect = res[0];
            if (select == undefined) {
                return;
            }

            //alert(select);
            if (select != "") {
                this.selectedlists.push(select);
                var del = this.lists.indexOf(select);
                //alert(del);
                this.lists.splice(del, 1);
            }
        },
        oneToLeft() {
            var select = document.getElementById("selectedlists").value;
            //var res = select.split("-");
            //select = res[1];
            //var indexselect = res[0];
            //alert("" + select);
            if (select == undefined) {
                return;
            }
            if (select != "") {
                this.lists.push(select);
                //this.lists.push(select);
                var del = this.selectedlists.indexOf(select);
                this.selectedlists.splice(del, 1);
                //this.selectedlists.splice(indexselect, 1);
            }
        },
        allToRight() {
            // alert(this.lists.length);
            //console.log("allToRight");
            var del = this.lists.length;
            if (this.lists[0] == undefined) {
                return;
            }
            //alert(del + "");
            for (var i = 0; i <= this.lists.length; i++) {
                //alert(i + "");
                this.selectedlists.push(this.lists[i]);
                //var del = this.lists.indexOf(this.lists[i]);
            }
            this.lists.splice(0, del);
        },
        allToLeft() {
            if (this.selectedlists[0] == undefined) {
                return;
            }

            var del = this.selectedlists.length;
            for (var i = 0; i < this.selectedlists.length; i++) {
                this.lists.push(this.selectedlists[i]);
                //  var del = this.selectedlists.indexOf(this.selectedlists[i]);
                //this.selectedlists.splice(del, 1);
            }
            this.selectedlists.splice(0, del);
        },
        populate() {
            //alert("");
            if (this.selectedlists.length <= 0) {
                Swal.fire({ title: "Please select a screen!" });
            }
            if (this.rname == "") {
                Swal.fire({ title: "Please enter role name!" });
            }

            if (this.description == "") {
                Swal.fire({ title: "Please enter description!" });
            }

            Swal.fire({
                position: "center",
                icon: "success",
                title: "Role name added" + this.rname,
                showConfirmButton: false,
                timer: 3500,
            });
            //this.selectedlists = this.lists;
        },
    },
};
</script>

<template>
    <Layout>
        <PageHeader :title="[editmode ? 'Edit Roles' : 'Add Roles']" :items="items" />
        <div>
            <form @submit.prevent="handleSubmit()">
                <div class="card">
                    <div class="card-body">
                        <div class="card">
                            <div class="card-body">
                                <div class="col-sm-12">
                                    <div class="input-group">
                                        <div class="input-group-text col-sm-4">Role Name:</div>
                                        <input :disabled="editmode" ref="rname" class="form-control" placeholder="role Name"
                                            v-model="rname" />
                                    </div>
                                </div>

                                <div class="mt-3"></div>
                                <div class="row">
                                    <div class="col-sm">
                                        <div class="">Select Screens / Windows / Page:</div>

                                        <select class="form-control" size="15" id="lists" multiple>
                                            <option :id="index" :value="list" v-for="(list, index) in lists" :key="list">
                                                {{ list }}
                                            </option>
                                        </select>
                                    </div>
                                    <div class="col-sm">
                                        <div class="row mb-4"></div>
                                        <div class="row mb-4"></div>
                                        <div class="row mb-4"></div>
                                        <div class="row mb-2">
                                            <button class="btn btn-secondary" @click="allToRight()">
                                                Add All &raquo;
                                            </button>
                                        </div>

                                        <div class="row mb-4">
                                            <button class="btn btn-secondary" @click="oneToRight()">
                                                Add Selected&rsaquo;
                                            </button>
                                        </div>
                                        <div class="row mb-4"></div>
                                        <div class="row mb-2">
                                            <button class="btn btn-danger" @click="oneToLeft()">
                                                &lsaquo; Remove Selected
                                            </button>
                                        </div>
                                        <div class="row">
                                            <button class="btn btn-danger" @click="allToLeft()">
                                                &laquo; Remove All
                                            </button>
                                        </div>
                                    </div>

                                    <div class="col-sm">
                                        <div>Selected Screens / Windows / Page:</div>
                                        <select class="form-control" size="15" id="selectedlists" multiple>
                                            <option :id="index" v-for="(selectedlist, index) in selectedlists"
                                                :value="selectedlist" :key="selectedlist">
                                                {{ selectedlist }}
                                            </option>
                                        </select>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="row">
                            <div class="col-sm-10"></div>
                            <div class="col-sm-2">
                                <button v-show="!editmode" class="btn btn-primary" @click="add()">
                                    Add Role
                                </button>
                                <button v-show="editmode" class="btn btn-primary" @click="editrec()">
                                    Edit Role
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            </form>
        </div>
    </Layout>
</template>
//#print {
  //height: 11in;
  //width: 8.5in;
//}
<style scoped>
.changebg {
    background-color: #f7f6ebfb;
}

.alert-custom {
    background-color: yellow;
}

.swal-overlay {
    background-color: rgba(43, 165, 137, 0.45);
}

.swal-modal {
    background-color: rgba(63, 255, 106, 0.69);
    border: 3px solid white;
}
</style>