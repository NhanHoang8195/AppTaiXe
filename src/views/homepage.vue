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
    <gmap-map id="map" :center="center" :zoom="10" style="height: 300px">
      <gmap-marker :position="taixe.location" :clickable="true" :draggable="true" @click="clickMarker($event,marker)" @dragend="dragMarker($event, marker)" :label="'A'"></gmap-marker>
      <template v-for="marker in taixeInfo">
        <template v-for="user in userInfo" v-if="marker.keyUser===user['.key'] && marker.username===taixe.username"> <!--v-if="taixe.keyUser===user['.key']" -->
              <gmap-marker :position="user.location" :clickable="true" :draggable="true" @click="clickMarker($event,marker)" @dragend="dragMarker($event, marker)" :label="'B'" ></gmap-marker>
          </template>
      </template>
    </gmap-map>
  </div>
  <div class="col-sm-6">
    <table class="table table-hover table-bordered">
      <thead>
        <tr>
          <th>Time</th>
          <th>Địa chỉ đón</th>
          <th>Số điện thoại</th>
          <th>Bạn muốn đón khách hàng này?</th>
        </tr>
      </thead>
      <tbody>
        <template v-for="marker in taixeInfo" v-if="marker.username===taixe.username">
          <tr v-for="user in userInfo" v-if="user['.key']===marker.keyUser">
            <td>{{user.date}}</td>
            <td> {{user.address}} </td>
            <td>{{user.phone}} </td>
            <td>
              <button :disabled="isAgree" @click="agree(true,marker,user)" class="btn btn-primary btn-sm">Đồng ý</button>
              <button :disabled="isAgree" @click="agree(false, marker,user)" class="btn btn-primary btn-sm">Không</button>
            </td>
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
          lat: 10.7843695,
          lng: 106.6844089
        },
        status: 'DANG-SAN-SANG',
        keyUser: '' // id cua user dang duoc don
      },
      isLogin: 'notLoggin', // check login or not
      isAgree: false, //Dong y nhan khach hay khong
      center: {}

    }
  },

  methods: {
    login: function() {

      for (let i = 0; i < this.taixeInfo.length; i++) {
        if (this.taixe.username === this.taixeInfo[i].username && this.taixe.password === this.taixeInfo[i].password) { // match user

          this.isLogin = 'logginSuccess';
          this.taixe = this.taixeInfo[i];
          delete this.taixe['.key'];
          console.log(this.taixeInfo[i]);
          this.center = this.taixe.location;
          break;
        } else {
          this.isLogin = 'logginFail';
        }
      }
      //  taixeInfoRef.push(this.taixe)
    },
    direction: function(marker, user) {
      var directionsService = new google.maps.DirectionsService;
      var directionsDisplay = new google.maps.DirectionsRenderer({
        draggable: true
      });
      var source = new google.maps.LatLng(marker.location.lat, marker.location.lng);
      var des = new google.maps.LatLng(user.location.lat, user.location.lng);
      directionsDisplay.setMap(new google.maps.Map(document.getElementById('map')));
      directionsService.route({
        origin: source,
        destination: des,
        travelMode: 'DRIVING'
      }, function(response, status) {
        if (status === 'OK') {
          directionsDisplay.setDirections(response);
        } else {
          window.alert('Directions request failed due to ' + status);
        }
      });
    },
    agree: function(obj, marker, user) {
      console.log('user', user);
      console.log('marker', marker);
      // let path = 'taixe-info' + '/' + marker['.key'];
      //
      // let updateTaixe = db.ref(path);
      //
      // delete marker['.key'];
      // if (obj === true) {
      //   marker.status = 'DANG-CHO-KHACH';
      //   let pathUser = 'user-info/' + user['.key'];
      //   delete user['.key'];
      //   let updateUser = db.ref(pathUser);
      //
      //   user.status = 'DA-CO-XE';
      //   updateUser.set(user);
      //   user['.key'] = pathUser.slice(pathUser.indexOf('/') + 1);
      //
      // } else {
      //   marker.status = 'TU-CHOI' + marker.keyUser;
      //   this.isAgree = false;
      // }
      // updateTaixe.set(marker);
      // marker['.key'] = path.slice(path.indexOf('/') + 1);
      // this.isAgree = true;
      // this.direction(marker, user);
    }

  },
  // updated() {
  //   if(this.isLogin==='logginSuccess'){
  //     for(let i = 0; i < this.userInfo.length; i++)
  //       if(this.userInfo[i].key)
  //     this.direction(taixe, )
  //   }

  //}
}
</script>
