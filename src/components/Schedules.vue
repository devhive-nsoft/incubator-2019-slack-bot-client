<template>
   <div id="Schedules" class="tabcontent">
<router-view @reload-schedules="reloadSchedules()" @show-notification="showNotification($event)"></router-view>
<div id="header">
        <h1>Schedules</h1>
      </div>
   <div id="divlist">
    <ul id="list">
      <li id="title-li">
        <div class="column1 column">
          <h5>Message</h5>
          <p>
            <i class="material-icons" @click="sortBy('message.title')">arrow_drop_down</i>
          </p>
        </div>
        <div class="column2 column">
          <h5>Next Run</h5>
          <p>
            <i class="material-icons" @click="sortBy('runAt')">arrow_drop_down</i>
          </p>
        </div>
        <div class="column3 column">
          <h5>Active At</h5>
          <p>
            <i class="tooltip, material-icons" @click="sortBy('active')">arrow_drop_down</i>
          </p>
        </div>
        <div class="column6 column">
          <h5>Repeat</h5>
          <p>
            <i class="tooltip, material-icons" @click="sortBy('repeat')">arrow_drop_down</i>
          </p>
        </div>
        <div class="column5 column">
          <h5>Channel</h5>
          <p>
            <i class="tooltip, material-icons" @click="sortBy('channel')">arrow_drop_down</i>
          </p>
        </div>
        <div class="column4 column">
          <p>
            <i class="material-icons">filter_list</i>
          </p>
        </div>
      </li>

      <li v-for="schedule in schedulesData" :key="schedule.scheduleId">
        <div class="linear1"></div>
        <div class="linear2"></div>
        <div class="column1 column">
          <p>{{ schedule.message.title }}</p>
        </div>
        <div class="column2 column">
          <p>{{ schedule.runAt | shortDate}}</p>
        </div>
        <div class="column3 column">
          <p>{{ schedule.active | activeView }}</p>
        </div>
        <div class="column6 column">
          <p>{{ schedule.repeat | repeatView }}</p>
        </div>
        <div class="column5 column">
          <p>{{ schedule.channel }}</p>
        </div>
        <div class="column4 column">
          <i class="material-icons" @click="editSchedule(schedule.scheduleId)">create</i>
          <i class="material-icons" @click="deleteSchedule(schedule.scheduleId)">delete</i>
        </div>

      </li>
    </ul>
  </div>

    <div id="footer">
      <button id="footer-btn" @click="toggleMenu" v-click-outside="hideMenu">{{ this.rowSize }} Rows</button>
      <div id="menu" v-show="menu">
        <div class="menu-article" v-for="rowValue in rowSizesValue" :key="rowValue" @click="setRows(rowValue)">{{ rowValue }}</div>
      </div>

      <div class="text-xs-center">
        <v-pagination v-model="page" :length="pagesSize" :total-visible="5" @input="changePage"></v-pagination>
      </div>
    </div>

    <button @click="showScheduleForm" id="btn">+</button>

    <div id="notification" v-show="showNoti" :class="{redBorder: errorOccured, greenBorder: !errorOccured}">
      <input type="text" v-model="textNoti" readonly :class="{redText: errorOccured, greenText: !errorOccured}" />
      <button @click="showNotification" :class="{redBackground: errorOccured, greenBackground: !errorOccured}">OK</button>
    </div>
   </div>
</template>

<script>
import {CURRENT_USER_ROLE, THEME} from "../constants/index.js";
import {ACCESS_TOKEN} from "../constants/index.js";
import axios from "axios";
import ClickOutside from "vue-click-outside";

export default {
  name: "schedules",

  data(){
       return {
      schedulesData: [],
      rowSize: 20, //Number of rows
      pagesSize: 1, //Number of pages
      page: 1, //current active page
      menu: false,
      rowSizesValue: [5, 10, 20],
      sortType: "desc",
      textNoti: "",
      errorOccured: false,
      showNoti: false,
      sortByValue: "createdAt",
    }  
  },

  mounted: function() { 
    if(localStorage.getItem(THEME)=="light"){
document.getElementById("header").style.backgroundColor="white";
document.getElementById("Schedules").style.backgroundColor="white";
}else if(localStorage.getItem(THEME)=="dark") {
document.getElementById("header").style.backgroundColor="black";
document.getElementById("Schedules").style.backgroundColor="black";
}
    //onload funkcija
    //this.create();
  },

  filters: {
    shortDate(value) {
      let dateVar = new Date(value);
      let dataVar2 = dateVar.toDateString();
      let data = dataVar2.substring(4);
      return data;
    },
    activeView(value){
      if(value == true)
        return "Active";
      else
        return "Not active";
    },
    repeatView(value){
      if(value == true)
        return "Repeat";
      else
        return "Not repeat";
    }
  },

  methods: {
    async reloadSchedules() {
      var pg = this.page - 1;
      try 
      {
        const res = await axios.get("http://localhost:8080/api/schedules?page=" + pg + "&size=" + this.rowSize + "&sort=" + this.sortByValue + "," + this.sortType);

        if (res.data.totalPages < this.page)
          this.changePage(res.data.totalPages);

        this.schedulesData = res.data.content;
        this.pagesSize = res.data.totalPages;
      } 
      catch (err) 
      {
        alert(err);
      }
    },

    showNotification(value) {
       if(value == -1)
       {
           this.textNoti = "Some error have occured";
           this.errorOccured = true;
       }
       else
       {
          this.errorOccured = false;
          this.textNoti = "Succes"; 
       }
       this.showNoti = !this.showNoti;
       setTimeout(this.closeNoti, 1500)
       {}
    },

    closeNoti(){
        this.showNoti = false;
    },

    showScheduleForm() {
      this.$router.push("/dashboard/schedules/newSchedule");
    },

    async editSchedule(id) {
      this.$router.push("/dashboard/schedules/updateSchedule/" + id);
    },

    async deleteSchedule(id) {
      await axios.delete("http://localhost:8080/api/schedules/" + id);
      var pg = this.page - 1;
      try 
      {
        const res = await axios.get("http://localhost:8080/api/schedules?page=" + pg + "&size=" + this.rowSize + "&sort=" + this.sortByValue + "," + this.sortType);
        
        if (res.data.numberOfElements == 0) 
        {
          if (this.page != 1) 
            this.changePage(this.page - 1);
        }
        this.messagesData = res.data.content;
        if (res.data.totalPages == 0) 
            this.pagesSize = 1;
        else 
            this.pagesSize = res.data.totalPages;
        this.showNotification(200);
      } 
      catch (err) 
      {
        this.showNotification(-1);
      }
      this.reloadSchedules();
    },

    toggleMenu() {
      this.menu = !this.menu;
    },

    hideMenu() {
      this.menu = false;
    },

    setRows(value) {
      if (value != this.rowSize) {
        this.rowSize = value;
        this.reloadSchedules();
      }
      this.menu = !this.menu;
    },

    changePage(nextPage) {
      this.page = nextPage;
      this.reloadSchedules(nextPage - 1);
    },

    async create() {
      try 
      {
        const res = await axios.get("http://localhost:8080/api/schedules?page=0&size=" + this.rowSize + "&sort=" + this.sortByValue + "," + this.sortType);
        this.schedulesData = res.data.content;
        if (res.data.totalPages == 0) 
            this.pagesSize = 1;
        else 
            this.pagesSize = res.data.totalPages;
        this.rowSize = res.data.size;
      }
      catch (err) {
       //alert(err);
        console.log(err);
      }
    },

    sortBy(value)
    {
      if (this.sortType == "desc") 
        this.sortType = "asc";
      else 
        this.sortType = "desc";
      
      this.sortByValue = value;
      this.reloadSchedules();
    }
  },
  directives: {
    ClickOutside
  }
  }
</script>

<style scoped>

h1 {
  margin-top: 0px;
  margin-left: 20px;
  width: 200px;
  float: left;
}
#header {
  margin-left: 15px;
  margin-right: 10px;
  display: block;
  width: 99%;
  float: right;
  background-color: white;
  height: 72px;
}
#divlist {
  padding-left: 10px;
  padding-right: 10px;
  width: 100%;
  height: 100vh;
}
body {font-family: "Lato", sans-serif;
background:#EAEAEA;}


#Schedules{
  height: 100vh;
  width: 100%;
  padding: 20px;
}

#btn {
    float:right;
    height: 60px;
    width: 60px;
    border-radius: 50%;
    font-size: 30px;
    position: absolute;
    bottom: 50px;
    right: 13px;
    background-color:  #006BF5;;
    color: white;
    border: 1px solid white;
    font-family: 'Courier New', Courier, monospace;
    line-height: 60px;
    z-index: 99;
}

#btn:hover{
  cursor: pointer;
}

#footer{
  bottom: 0%;
}

.column1{
  width: 35%;
  margin-right: 10px;
}

.column2{
  width: 7%;
  margin-right: 25px;
  margin-left: 0px;
  position: relative;
  right: 15px;
}

#title-li .column2{
  position: unset;
}

#title-li .column4{
  margin-left: 30px;
}

.column3{
  margin-left: 5px;
}

.column4{
  width: 33%;
  margin-left: 0px;
}

.column5{
  width: 11%;
  margin-left: 0px;
  padding-left: 20px;
}

#title-li .column5{
  padding-left: 0px;
}

.column6{
  width: 7%;
  margin-left: 0px;
}

.linear1{
  left: 34%;
}

.linear2{
  left: 67%;
}

</style>
