<template>
  <nav class="navbar navbar-inverse" id="nav">
    <div class="container-fluid">
      <div class="navbar-header">
        <a class="navbar-brand" href="#">PicMap</a>
      </div>
      <ul class="nav navbar-nav navbar-right">
          <li><a>
              <div class="custom-file">
                  <input v-if="this.username != 'NULL'" type="file" class="custom-file-input" id="inputGroupFile02" style="display: none;" @change="inputHandler($event)">
                  <label class="custom-file-label" for="inputGroupFile02"><span class="glyphicon glyphicon-upload"></span> Choose file </label>
              </div>
          </a></li>
          <li><a href="#" @click="signOut()"><span class="glyphicon glyphicon-user"></span> Sign Out</a></li>
      </ul>
    </div>
  </nav>
</template>
<script>
export default {
  name: "HomePage",
  props: ["lon", "lat", "username"],
  data() {
    return {
      file: undefined,
      
      /**
       * The Auth2 parameters, as seen on
       * https://developers.google.com/identity/sign-in/web/reference#gapiauth2initparams.
       * As the very least, a valid client_id must present.
       * @type {Object}
       */

      googleSignInParams: {
        client_id:
          "728975308271-pk06eacp60cv1j16ngm0sft0a9gr1snj.apps.googleusercontent.com"
      }
    };
  },
  methods: {
    signOut() {
      console.log("ORANGE");
      this.$emit("logOut");
    },

    async requestImage(file) {
      console.log("Orange!");
      var req = new XMLHttpRequest();
      req.onreadystatechange = () => {
        if (req.readyState == 4 && req.status == 200) {
          this.processRequest(req.responseText);
          console.log("BLUE: " + req.responseText);
        }
      };
      var request_url = "https://api.imgur.com/3/image";
      var api_key = "bf6ae890fb73e6b";
      req.open("POST", request_url, true); // true for asynchronous
      req.setRequestHeader("Authorization", "Client-ID " + api_key);
      req.send(file);
      console.log("White!");
    },

    processRequest(response_text) {
      if (response_text == "Not found") {
        console.log("Imgur album not found.");
      } else {
        //Response is here
        var json = JSON.parse(response_text);
        let imageLink = json.data.link;
        console.log(imageLink);

        //update user schema with new photo sub doc by passing in url parameters
        let url =
          "https://pic-app-client.herokuapp.com/users/" +
          this.username +
          `?url=${imageLink}&lon=${this.lon}&lat=${this.lat}`;
        console.log(url);
        var xhr = new XMLHttpRequest();
        xhr.open("PUT", url, true);
        xhr.responseType = "text";
        console.log("Request about to onload");
        xhr.onload = () => {
          console.log("in onload for update image xmlhttpr");
          if (xhr.readyState === xhr.DONE) {
            if (xhr.status === 200) {
              console.log("trying to add marker to map after image upload");
              /*  this.users = xhr.response;
                  let photo = this.users.photos[this.users.photos.length-1];
                  var el = document.createElement('div').classList.add('marker');
                  new mapboxgl.Marker(el).setLngLat([photo.coordinates.longitude, photo.coordinates.latitude]).setPopup(new mapboxgl.Popup({ offset: 25 }).setHTML(`<img src=${photo.url} alt="" height="84" width="84">`)).addTo(map); */
              //Emit event to reload map
              this.$emit("addedmarker");
            }
          }
        };
        xhr.send(null);
      }
    },

    inputHandler(event) {
      //Handle image upload here
      this.file = event.target.files[0];
      console.log("File has been changed");
      this.requestImage(this.file);
    }
  }
};

components: {
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.g-signin-button {
  /* This is where you control how the button looks. Be creative! */
  display: inline-block;
  padding: 4px 8px;
  border-radius: 3px;
  background-color: #3c82f7;
  color: #fff;
  box-shadow: 0 3px 0 #0f69ff;
}
#nav {
  margin-bottom: 0px;
}
</style>
