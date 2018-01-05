<template>
<div>
  <template v-if="isLogin==='notLoggin' || isLogin==='logginFail'">
        <form class="col-sm-6 col-sm-offset-3" @submit.prevent="login">
            <div class="form-group">
              <label for="exampleInputEmail1">Username</label> <template v-if="isLogin==='logginFail'"><p style="color:red ">Username hoặc password chưa chính xác</p></template>
  <input v-model="taixe.username" type="text" class="form-control" id="exampleInputEmail1" placeholder="Username" onblur="this.placeholder='Username'" onfocus="this.placeholder=''" required>
</div>
<div class="form-group">
  <label for="exampleInputPassword1">Password</label>
  <input v-model="taixe.password" type="password" class="form-control" id="exampleInputPassword1" placeholder="Password" onblur="this.placeholder='Password'" onfocus="this.placeholder=''" required>
</div>
<button type="submit" class="btn btn-primary">Submit</button>
</form>
</template>
<template v-if="isLogin==='logginSuccess'">
<!--login successfully -->
<div>
  <center>
    <h2>Xin chào {{taixe.username}}</h2>
  </center>
  <div class="col-sm-5">
    <gmap-map ref="map" id="map" :center="center" :zoom="10" style="height: 300px">

      <template v-for="marker in taixeInfo" v-if="marker.username===taixe.username">
        <gmap-marker :position="marker.location"  :draggable="true" :title="'Vị trí của bạn'"  :label="'A'"  :clickable="false"  @dragend="dragMarker($event, marker)"></gmap-marker>
        <!-- @dragend="dragMarker($event, marker)" -->
        <template v-for="user in userInfo"> <!--v-if="taixe.keyuser===user['.key']" -->
             <template v-for="userdetail in marker.keyuser" v-if="userdetail===user['.key']">
              <gmap-marker :position="user.location"  :draggable="false" @click="clickMarker($event,user)"  :label="'B'", :title="user.phone" ></gmap-marker>
            </template>
      </template>
      </template>
    </gmap-map>
    <template v-if="displayInfo">
      <div>
      <table class="table table-hover table-bordered">
        <thead>
          <tr>
            <th>#</th>
            <th>Chir d</th>
          </tr>
        </thead>
        <tbody >
          <tr>
            <th>Số điện thoại</th>
            <td>{{obj.phone}}</td>
          </tr>
          <tr>
            <th>Địa điểm đón</th>
            <td>{{obj.address}}</td>
          </tr>
        </tbody>
      </table>
      </div>
    </template>
    <template v-else>
      <div id="directionPanel">

      </div>
    </template>
  </div>
  <div class="col-sm-7">
    <table class="table table-hover table-bordered">
      <thead>
        <tr>

          <th>Time</th>
          <th>Địa chỉ đón</th>
          <th>Số điện thoại</th>
          <th>Status</th>
        </tr>
      </thead>
      <tbody>
        <template v-for="marker in taixeInfo" v-if="marker.username===taixe.username && marker.status!=='KET-THUC'">
          <tr v-for="user in userInfo">
             <template v-for="userdetail in marker.keyuser" v-if="userdetail===user['.key']">
              <td>{{user.date}}</td>
              <td> {{user.address}} </td>
              <td>{{user.phone}} </td>
              <td>
                <template v-if="marker.status==='DANG-SAN-SANG'">
                  <button  @click="agree(true,marker,user)" class="btn btn-primary btn-sm">Đồng ý</button>
                  <button @click="agree(false, marker,user)" class="btn btn-primary btn-sm">Không</button>
                </template>
        <template v-else-if="marker.status==='DANG-CHO-KHACH'">
                  <button @click="start(marker)" class="btn btn-success btn-sm">Bắt đầu</button>
                </template>
        <template v-else>
                  <button @click="end(marker, user)" class="btn btn-success btn-sm">Kết thúc</button>
                </template>
        </td>
        </template>

        </tr>

        </template>
      </tbody>
    </table>
  </div>
</div>
</template>

</div>
</template>

<script>
import Firebase from 'firebase'
//var bcrypt = require('bcrypt');
let saltRounds = 10;
let config = {
  apiKey: "AIzaSyBdayjA3EmnHntCcCA0caJvfUQPESPassM",
  authDomain: "app-ddv.firebaseapp.com",
  databaseURL: "https://app-ddv.firebaseio.com",
  projectId: "app-ddv",
  storageBucket: "app-ddv.appspot.com",
  messagingSenderId: "966606157223"
}
let app = Firebase.initializeApp(config);
let db = app.database();
let taixeInfoRef = db.ref('taixe-info');
let userInfoRef = db.ref('user-info');
var directionsService;
var directionsDisplay;
export default {
  firebase: {
    taixeInfo: taixeInfoRef,
    userInfo: userInfoRef
  },
  data() {
    return {
      taixe: {
        username: '',
        password: '',
        location: {
          lat: 10.7480929,
          lng: 106.6352362
        },
        status: 'DANG-SAN-SANG'
      },
      isLogin: 'notLoggin', // check login or not
      center: {},
      obj: {},
      displayInfo: false,
      userDirection: {}
    }
  },

  methods: {
    dragMarker: function(event, obj) { // keo tha vi tri hien tai
      this.taixe.location.lat = event.latLng.lat();
      this.taixe.location.lng = event.latLng.lng();
      var updateLinks = {};
      updateLinks['taixe-info/' + this.taixe['.key'] + '/location'] = this.taixe.location;
      db.ref().update(updateLinks);
      directionsDisplay.setPanel(null);
      directionsDisplay.setMap(null);
      this.direction(this.taixe, this.userDirection);
    },
    clickMarker: function(event, obj) { // click vao 1 marker hien len info cua maker do
      this.displayInfo = true;
      this.obj = obj;
    },
    login: function() {

      for (let i = 0; i < this.taixeInfo.length; i++) {
        if (this.taixe.username === this.taixeInfo[i].username && this.taixe.password === this.taixeInfo[i].password) { // match user
          this.isLogin = 'logginSuccess';
          this.taixe = this.taixeInfo[i];
          this.center = this.taixe.location;
          break;
        } else {
          this.isLogin = 'logginFail';
        }
      }
      //  taixeInfoRef.push(this.taixe)
    },
    direction: function(marker, user) {
      directionsService = new google.maps.DirectionsService();
      directionsDisplay = new google.maps.DirectionsRenderer({
        map: this.$refs['map'].$mapObject,
        suppressMarkers: true,
        panel: document.getElementById('directionPanel')
      });
      var source = new google.maps.LatLng(marker.location.lat, marker.location.lng);
      var des = new google.maps.LatLng(user.location.lat, user.location.lng);
      directionsService.route({
        origin: source,
        destination: des,
        travelMode: 'DRIVING'
      }, function(response, status) {
        if (status === 'OK') {
          console.log(response);
          directionsDisplay.setDirections(response);
        } else {
          window.alert('Directions request failed due to ' + status);
        }
      });
      console.log(directionsDisplay);
    },
    agree: function(obj, marker, user) {
      this.taixe = marker;
      this.userDirection = user;
      var updateLinks = {};

      if (obj === true) {
        updateLinks['taixe-info' + '/' + marker['.key'] + '/status'] = 'DANG-CHO-KHACH';
        for (let i = 0; i < this.taixeInfo.length; i++) {
          for (let key in this.taixeInfo[i].keyuser) {
            if (this.taixeInfo[i].username === this.taixe.username) {
              if (this.taixeInfo[i].keyuser[key] !== user['.key']) {
                updateLinks['taixe-info' + '/' + this.taixeInfo[i]['.key'] + '/keyuser/' + key] = null;
              }
            } else if (this.taixeInfo[i].keyuser[key] === user['.key']) {
              updateLinks['taixe-info' + '/' + this.taixeInfo[i]['.key'] + '/keyuser/' + key] = null;
            }
          }
        }
        updateLinks['user-info/' + user['.key'] + '/status'] = 'DA-CO-XE';
        db.ref().update(updateLinks);
        this.direction(marker, user);
      } else {
        let count = 0;
        for (let i = 0; i < this.taixeInfo.length; i++) {
          for (let key in this.taixeInfo[i].keyuser) {
            if (this.taixeInfo[i].keyuser[key] === user['.key']) {
              count++;
              break;
            }
          }
        }
        if (count === this.taixeInfo.length) {
          updateLinks['user-info' + '/' + user['.key'] + '/status/'] = 'KHONG-CO-XE';
          db.ref().update(updateLinks);
          updateLinks = {}
        }
        for (let key in this.taixe.keyuser) {
          if (this.taixe.keyuser[key] === user['.key']) {
            updateLinks['taixe-info' + '/' + this.taixe['.key'] + '/keyuser/' + key] = null;
            db.ref().update(updateLinks);

            break;
          }
        }
      }

    },
    start: function(obj) {
      var updateLinks = {};
      updateLinks['taixe-info' + '/' + obj['.key'] + '/status'] = 'BAT-DAU';
      db.ref().update(updateLinks);
    },
    end: function(obj, user) {
      var updateLinks = {};
      updateLinks['taixe-info' + '/' + obj['.key'] + '/status'] = 'DANG-SAN-SANG';
      updateLinks['taixe-info' + '/' + obj['.key'] + '/keyuser'] = null;
      updateLinks['user-info' + '/' + user['.key'] + '/status'] = 'KET-THUC';
      updateLinks['taixe-info' + '/' + obj['.key'] + '/location'] = user.location;
      db.ref().update(updateLinks);
      directionsDisplay.setMap(null);

    }
  }
}
</script>
