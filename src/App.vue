<template>
  <div id="app">
    <div class="container">
      <div class="" v-show="!authentication">
        <button type="button" name="button" @click="login">login</button>
      </div>
      <div class="" v-show="authentication">
        <div class="row">
          <div class="col-xs-6 ">
            <img class="img-circle" :src="user.photoURL" alt="" />
          </div>
          <div class="col-xs-6 ">
            {{user.displayName}}<br><br>
          </div>
          <div class="col-xs-6 ">
            <button type="button" name="button" @click="logout">logout</button><br>
          </div>
        </div>
        <div class="row">
          <div class="col-xs-10 col-xs-offset-1  posts">
            <div class="row">
              <div class="col-xs-12 post"  v-for="post in posts">
                <div class="row">
                  <div class="col-xs-3">
                    <img class="img-circle media-object display-photo" :src="post.photoURL" alt="" />
                  </div>
                  <div class="col-xs-9">
                    <h4 class="media-heading"><span class="label label-default">{{post.name}}:</span> </h4>
                     {{post.text}}<button type="button" name="button" @click="removePost(post)" v-if="user.uid === post.uid">X</button>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="row">
          <div class="col-xs-10 col-xs-offset-1 ">
            <textarea class="form-control input-text" name="name" rows="2" cols="" v-model="text" @keydown.enter="addPost"></textarea>
          </div>
          <div class="clearfix visible-xs-block"></div>
          <div class="col-xs-2 col-xs-offset-1 ">
            <button type="button" name="button" @click="addPost">Post</button><br>
          </div>

        </div>
      </div>
    </div>

    <!-- <router-view></router-view> -->
  </div>
</template>

<script>
import firebase from 'firebase'
var config = {
  apiKey: 'AIzaSyBaObZL7eq8Tkt-O1tLvCzseYTA3xQdk2A',
  authDomain: 'learn-vuefire.firebaseapp.com',
  databaseURL: 'https://learn-vuefire.firebaseio.com',
  storageBucket: 'learn-vuefire.appspot.com',
  messagingSenderId: '1060902645723'
}
var firebaseApp = firebase.initializeApp(config)
var db = firebaseApp.database()
var provider = new firebase.auth.FacebookAuthProvider()
export default {
  name: 'app',
  data () {
    return {
      posts: [],
      user: '',
      token: '',
      authentication: false,
      text: ''
    }
  },
  mounted () {
    var vm = this
    // ผูก database เข้ากับตัวแปร posts แบบ Array  AsObject
    vm.$bindAsArray('posts', db.ref('posts'))
    // check ทุกครั้งว่ามีคน login อยู่หรือไม่
    firebase.auth().onAuthStateChanged(function (user) {
      if (user) {
        vm.user = user
        vm.authentication = true
      }
    })
    firebase.auth().getRedirectResult().then(function (result) {
      if (result.credential) {
        // เข้าแค่ครังแรกที่กด Login จะได้ token มาใช้
        vm.token = result.credential.accessToken
        console.log('token' + vm.token)
      }
    })
  },
  methods: {
    login () {
      firebase.auth().signInWithRedirect(provider)
    },
    logout () {
      var vm = this
      firebase.auth().signOut().then(function () {
        // Sign-out successful.
        // clear ข้อมูล
        vm.user = ''
        vm.token = ''
        vm.authentication = false
      }, function (error) {
        console.log(error)
      })
    },
    addPost () {
      // เพิ่มข้อมูล
      var vm = this
      this.$firebaseRefs.posts.push({
        text: vm.text,
        uid: vm.user.uid,
        name: vm.user.displayName,
        photoURL: vm.user.photoURL
      })
      vm.text = ''
    },
    removePost (post) {
      // ลบข้อมูล
      console.log(post['.key'])
      this.$firebaseRefs.posts.child(post['.key']).remove()
    }
  }
}
</script>

<style>
.box {
  /*border: 1px black solid;*/
}
.posts {
  height: 70vh;
  border: 1px black solid;
  border-radius: 20px;
  overflow: auto;
}
.post{
  margin-top: 5%;
}
.display-photo {
  width: 100%;
}
.input-text {
  width: 100%;
}
</style>
