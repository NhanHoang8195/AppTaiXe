<template>
  <div >
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
       <template v-if="isLogin==='logginSuccess'"> <!--login successfully -->
        <div>
          <center>
            <h2>Xin chào {{taixe.username}}</h2>
          </center>
          <div class="col-sm-5">
            <gmap-map :center="center" :zoom="4" style="height: 300px" >
              <template  v-for="marker in taixeInfo">
                <gmap-marker  v-if="marker.username===taixe.username " :position="marker.location" :clickable="true" :draggable="true" @click="clickMarker($event,marker)" @dragend="dragMarker($event, marker)" :label="'A'"></gmap-marker>
                <gmap-marker  v-if="marker.username===taixe.username && (typeof marker.khachhang !=='undefined')"   :position="marker.khachhang.location" :clickable="true" :draggable="true" @click="clickMarker($event,marker)" @dragend="dragMarker($event, marker)" :label="'B'" ></gmap-marker>
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
              <tbody >
                <template v-for="marker in taixeInfo">
                  <tr v-if="marker.username===taixe.username && (typeof marker.khachhang !=='undefined')" >
                    <td>{{marker.khachhang.date}}</td>
                    <td>  {{marker.khachhang.address}}  </td>
                    <td>{{marker.khachhang.phone}}  </td>
                    <td>
                      <button @click.prevent="agree(true, marker)" class="btn btn-primary btn-sm">Đồng ý</button>
                      <button @click.prevent="agree(false, marker)" class="btn btn-primary btn-sm">Không</button>
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
export default {
  firebase() {
    return {
      taixeInfo : taixeInfoRef
    }

  },
  data() {
    return {
      taixe: {
        username: '',
        password: '',
        location: {
          lat:10.774597,
          lng:106.667954
        },
        status: 'DANG-SAN-SANG'
      },
      isLogin:'notLoggin', // check login or not
      isAgree: true, //Dong y nhan khach hay khong
      center: {
        lat: 14.058324,
        lng: 108.277199
      },
      markers:[
        {
          location: {
          lat: 14.058324,
          lng: 108.277199
        }
        }
      ]

    }
  },
  methods: {
    login: function() {
      for(let i = 0; i < this.taixeInfo.length; i++) {
        if(this.taixe.username === this.taixeInfo[i].username && this.taixe.password === this.taixeInfo[i].password) {  // match user

          this.isLogin = 'logginSuccess';
          this.taixe=this.taixeInfo[i];
          break;
        } else {
          this.isLogin = 'logginFail';
        }
      }
  //    taixeInfoRef.push(this.taixe)
      },
    agree: function(obj, marker) {
      let path = 'taixe-info' + '/' + marker['.key'];

      let updateTaixe = db.ref(path);
      delete this.taixe['.key'];

      if(obj === true) {
        taixe.status='DANG-CHO-KHACH';
      } else {
        this.isAgree = false;
        delete this.taixe.khachhang;
      }
      updateTaixe.set(this.taixe);
    }

  },
  // mouted() {
  //   console.log('haha');
  // }
  // updated(){
  //   if(typeof this.taixe.khachhang !== 'undefined') { // dang cho confirm co nhan khach hay khong
  //     console.log('xxx');
  //     console.log(this.taixe);
  //     this.isAgree = true;
  //   } else if(this.isLogin==='logginSuccess'){
  //     this.isAgree=false;
  //     console.log('yyy');
  //   }
  //   console.log('aaa');
  // }
}
</script>
