<template>
  <v-layout row justify-center style="background-color:#f7f7f7;" class="mb-4">
      <v-card>
        <v-card-title style="padding-left:40px;">
          <span class="headline">Portfolio Writer</span>
        </v-card-title>
        <v-card-text class="pb-0 pt-0">
          <v-container grid-list-md pt-0>
            <v-layout wrap>
              <!-- image preview -->
              <v-flex xs12>
                <img :src="imageUrl" height="150" v-if="imageUrl" />
                <input type="file" style="display:none" ref="image" accept="image/*" @change="onLocalImagePicked">
              </v-flex>
              <!-- title -->
              <v-flex xs12>
                <v-text-field label="Title" v-model="title" required></v-text-field>
              </v-flex>
              <!-- ImgBtn start -->
              <v-flex xs2 text-xs-center pl-0 class="bg-1">
                <button @click="useLocalFile" class="button button--wayra button--border-thin button--text-medium button--size-xs"
                  style="min-width:90%; max-width:90%;padding:0.3em 0.5em;margin:0;">
                    Use Local Image
                </button>
              </v-flex>

              <v-flex xs2>
                <template>
                  <div class="text-xs-center bg-1">
                    <v-dialog v-model="dialog" width="500">
                      <template v-slot:activator="{ on }">
                        <button v-on="on" class="button button--wayra button--border-thin button--text-medium button--size-xs"
                          style="min-width:90%; max-width:90%;padding:0.3em 0.5em;margin:0;">
                            Use Url Image
                        </button>
                      </template>
                      <v-card>
                        <v-card-title
                          class="headline grey lighten-2"
                          primary-title
                        >
                          사진 업로드
                        </v-card-title>
                        <v-card-text>
                          <v-text-field label="Add Img By URL" v-model="selectUrl" required @change="useUrlImg"></v-text-field>
                        </v-card-text>
                        <v-divider></v-divider>
                        <v-card-actions>
                          <v-spacer></v-spacer>
                          <v-btn color="primary" flat @click="dialog = false">올리기!</v-btn>
                        </v-card-actions>
                      </v-card>
                    </v-dialog>
                  </div>
                </template>
              </v-flex>


              <v-flex xs2 text-xs-center class="bg-1">
                <button @click="useRandomImg" class="button button--wayra button--border-thin button--text-medium button--size-xs"
                  style="min-width:90%; max-width:90%;padding:0.3em 0.5em;margin:0;">
                    Random Image
                </button>
              </v-flex>
              <v-flex xs2 text-xs-center class="bg-1">
                <button @click="clearimg" class="button button--wayra button--border-thin button--text-medium button--size-xs"
                  style="min-width:90%; max-width:90%;padding:0.3em 0.5em;margin:0;">
                    Delete Image
                </button>
              </v-flex>
              <!-- ImageBtn end -->
              <v-flex xs2>
              </v-flex>
              <!-- Markdown editor -->
              <v-flex xs12 class="mt-3">
                <template>
                  <div id="app">
                    <vue-editor v-model="text"></vue-editor>
                  </div>
                </template>
              </v-flex>
            </v-layout>
          </v-container>
        </v-card-text>
        <v-card-actions class="bg-1">
          <v-spacer></v-spacer>
          <button @click="clear" class="button button--wayra2 button--border-thin button--text-medium button--size-xs"
            style="min-width:120px; max-width:120px;padding:0.3em 0.5em;margin:0.2em;">
              Clear
          </button>
          <router-link to="/portfolio">
            <button class="button button--wayra2 button--border-thin button--text-medium button--size-xs"
              style="min-width:120px; max-width:120px;padding:0.3em 0.5em;margin:0.2em;">
                Back
            </button>
          </router-link>
          <button @click="save" class="button button--wayra2 button--border-thin button--text-medium button--size-xs"
            style="min-width:120px; max-width:120px;padding:0.3em 0.5em;margin:0.2em;">
              Save
          </button>
        </v-card-actions>
      </v-card>
  </v-layout>
</template>

<script>
import { VueEditor } from "vue2-editor";
import FirebaseService from '@/services/FirebaseService'

export default {
  name : 'PortfolioWriter',
  components : {
    VueEditor,
  },
  data() {
    return {
      text : '',  //bind with markdown editor
      title : '',
      imageName : '',
      imageUrl : '',
      imageFile : '',
      selectUrl : '',
      storageUrl : '',
      portfolioId : '',
      userEmail : '',
      dialog: false
    };
  },
  mounted() {
    // console.log("this.$route.params.id : " + this.$route.params.id)
    // console.log(this.$store.getters.getUser.email)

    //If modify portfolio, PortfolioWriter.vue can get data from Portdetail.vue
    this.portfolioId = this.$route.params.id
    this.title = this.$route.params.title
    this.imageUrl = this.$route.params.imgSrc
    this.text = this.$route.params.body
    
    //Get userinfo from vuex
    if (this.$route.params.user){
      this.userEmail = this.$route.params.user
    }
    else{
      this.$store.dispatch("checkUserStatus")
      .then(()=>{
        this.userEmail = this.$store.getters.getUser.email;
      });
    }
    
  },
  methods : {
    //Save Portfolio
    save : function(event) {
      //Blank check
      if(this.text == '' || this.title == '' || this.imageUrl == '') {

        var alertMsg = '';
        if(this.imageUrl == '') {
          alertMsg = '이미지는 필수항목입니다. 이미지를 선택해주세요.'
        } else if(this.title == '') {
          alertMsg = '제목은 필수항목입니다. 제목을 입력해주세요.'
        } else if(this.text == '') {
          alertMsg = '내용은 필수항목입니다. 내용을 입력해주세요';
        }

        this.$swal({
            type: 'error',
            title: 'Oops...',
            text: alertMsg
        });
      }

      else {
        //Call Firebase service
        console.log(this.title)
        FirebaseService.postPortfolio(this.userEmail, this.title, this.text, this.imageUrl, this.portfolioId)
        this.dialog = false

        //Reinitialize data
        this.imageName = ''
        this.imageUrl = ''
        this.imageFile = ''
        this.text = ''
        this.title = ''

        //Popup
        this.$swal({
          type : 'success',
          title : 'Great!',
          html : '저장되었습니다!</br>더 작성하시겠습니까?',
          showCancelButton : true,
          showConfirmButton : true,
        }).then((result) => {
          if(result.value) {
            // if clink ok
          }
          else {
            this.goPortfolio();
          }
        })
      }
    },
    clear : function(event) { // ClearBtn
      this.text = ''
      this.title = ''
    },
    clearimg(){ // DeleteImageBtn
      this.imageUrl = ''
    },
    useUrlImg(){ // UseUrlImageBtn
      this.imageUrl = this.selectUrl
      this.selectUrl = ''
      this.dialog = false;
      this.onUrlImagePicked(this.imageUrl)
    },
    useRandomImg(){ // RandomImgBtn
      this.imageUrl = 'https://source.unsplash.com/random/800x600'
      this.onUrlImagePicked(this.imageUrl)
    },
    useLocalFile() { // UseLocalImageBtn
      this.$refs.image.click()
    },
    onLocalImagePicked(e) { // Transform Local Image to base64 type data url
      console.log("select section")
      const files = e.target.files
      if(files[0] !== undefined) {
        this.imageName = files[0].name
        console.log("name : " + this.imageName)
        const fr = new FileReader()
        fr.readAsDataURL(files[0])
        fr.addEventListener('load', () => {
          this.imageUrl = fr.result
          this.imageFile = files[0]
          console.log("url : " + this.imageUrl)
          console.log("file : " + this.imageFile)
          console.log(this.imageUrl.name)
        })
      } else {
        this.imageName=''
        this.imageFile=''
        this.imageUrl=''
      }
    },
    onUrlImagePicked(url) { // Transform Url Image to base64 type data url
      const image2base64 = require('image-to-base64');
      image2base64(url)
        .then(
          (response) => {
              this.imageUrl = 'data:image/jpeg;base64,' + response
              console.log("i264 : " +this.imageUrl)
            }
        )
    },
    goPortfolio() {
      this.$router.push('/portfolio');
    }
  }
};
</script>
