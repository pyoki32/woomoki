<template>
  <v-container class="container-size">
    <v-row class="d-flex justify-center mb-5">
      <h1>씨앗 내용 변경</h1>
    </v-row>
    <v-row class="d-flex justify-center mb-10">
      <v-icon color="red lighten-2">fas fa-exclamation-triangle</v-icon>
      <span class="caution-font ml-2">씨앗은 내용, 보살핌 예시 사진, 최대 참여 인원만 수정 할 수 있습니다</span>
    </v-row>
    <v-row class="img-size">
      <v-hover>
        <template v-slot:default="{ hover }">
          <v-avatar color="gery" rounded width="100%" height="450px">

            <!-- <span v-if="!SeedInfo.sum_img" class="white--text">
              {{ this.text2 }}
            </span> -->
            <v-img :class="`rounded-lg`" width="100%" height="450px" max-width="100%" max-height="100%"
              v-if="SeedInfo.sum_img" readonly :src="SeedInfo.sum_img"></v-img>
            <v-fade-transition>
              <v-overlay class="black--text" v-if="hover" absolute color="white">
                대표 사진
              </v-overlay>
            </v-fade-transition>

          </v-avatar>
        </template>
      </v-hover>
    </v-row>

    <v-row class="mt-10">
      <!-- <v-hover> -->
      <!-- <template v-slot:default="{ hover }"> -->
      <v-avatar class="img-cursor" rounded width="100%" height="450px" color="grey" @click="onClickImageUpload">
        <span v-if="!SeedInfo.example_img" class="white--text">
          {{ this.text }}
        </span>
        <v-img :class="`rounded-lg`" width="100%" height="450px" max-width="100%" max-height="100%"
          v-if="SeedInfo.example_img" :src="SeedInfo.example_img">
          <!-- <v-fade-transition>
              <v-overlay
                v-if="hover"
                absolute
                color="#036358"
              > -->
          <!-- <v-btn color="#AED864">인증 예시 사진 변경 -->
          <!-- </v-btn> -->
          <!-- </v-overlay>
            </v-fade-transition> -->
        </v-img>
        <input ref="imageInput" type="file" hidden @change="onChangeImages">
      </v-avatar>
      <!-- </template> -->
      <!-- </v-hover> -->
    </v-row>

    <v-row class="mt-10">
      <v-text-field color="#AED864" v-model="SeedInfo.title" outlined placeholder="EX) 매일매일 코딩 공부하기" label="씨앗 이름"
        readonly disabled></v-text-field>
    </v-row>

    <v-row>
      <v-textarea color="#AED864" clear-icon="fas fa-times" v-model="content" :rules="contentRules" :counter="30000"
        outlined placeholder="EX) 코딩을 꾸준히 하지 않으면 까먹기 쉽지 않나요? 각자 조금씩이라도 코딩을 하고 매일매일 커밋하는 씨앗을 심고 싶어요! 함께하실 가꾸미님 구합니다!!"
        label="씨앗에 보살피는 방법" no-resize></v-textarea>
    </v-row>


    <v-row>
      <v-slider v-model="max_people" label="최대 인원" color="grey" track-color="grey" thumb-color="#AED864"
        :thumb-label="true" min="0" max="50"></v-slider>
    </v-row>

    <v-row>
      <v-text-field color="#AED864" v-model="dateRangeText" outlined label="기간" readonly disabled></v-text-field>
    </v-row>

    <v-row>
      <v-text-field color="#AED864" clear-icon="fas fa-times" v-model="weekDay" outlined label="보살핌 횟수" disabled>
      </v-text-field>
    </v-row>

    <v-row>
      <v-text-field suffix="원" color="#AED864" v-model="SeedInfo.join_deposit" outlined label="참여 금액(원)" readonly
        disabled></v-text-field>
    </v-row>

    <v-row class="d-flex justify-end">
      <v-btn color="#AED864" class="white--text" @click="UpdateSeed" :disabled="isAllSubmit === false">
        적용
      </v-btn>
    </v-row>

  </v-container>
</template>

<script>
  import axios from 'axios'

  export default {
    name: "SeedUpdate",
    components: {},
    data: function () {
      return {
        text: "보살핌 대표 사진 변경",
        seedId: this.$route.params.seedId,
        SeedInfo: [],
        sum_img: "",
        content: "",
        contentRules: [
          v => !!v || '내용은 필수 항목입니다',
          v => v.length >= 50 || '참여자들을 위해 내용을 자세히 적어주세요',
          v => v.length <= 30000 || '30000자를 초과할 수 없습니다',
        ],
        example_img: "",
        max_people: "",
        dates: [],
        imageUrl: "https://t1.daumcdn.net/cfile/tistory/995A17455A409C9A28",
        overlay: false,
        count: [],
        isSubmit: {
          isThumbnail: false,
          isContent: false,
          isCertification: false,
          isPeople: false,
        },
        isAllSubmit: false,
        changedImg: false,

        // s3 업로드 관련

        albumBucketName: "cert-photo-upload",
        bucketRegion: "ap-northeast-2",
        IdentityPoolId: "ap-northeast-2:8cf7cb29-d051-4f38-885f-09b1e4dd8153",

        photoURL: "https://s3.ap-northeast-2.amazonaws.com/cert-photo-upload/",
        photoKey: "",
        file: null,

      }
    },
    methods: {
      onClickImageUpload() {
        this.$refs.imageInput.click();
      },
      onChangeImages(e) {
        console.log(e.target.files)
        this.file = e.target.files[0];
        if (e.target.files.length === 1) {
          this.changedImg = true;
        } else {
          this.changedImg = false;
        }
        this.SeedInfo.example_img = URL.createObjectURL(this.file); // Create File URL
        this.text = ""
        this.fileNameSetting();
      },
      async fileNameSetting() {

        const user_id = this.$store.state.UserStore.user.user_id;

        var now = new Date();

        var year = now.getFullYear(); // 연도
        var month = now.getMonth() + 1; // 월
        var date = now.getDate(); // 일
        var hours = now.getHours(); // 시간
        var minutes = now.getMinutes(); // 분
        var seconds = now.getSeconds(); // 초
        var milliseconds = now.getMilliseconds(); // 밀리초

        // console.log("현재 : ", now);
        var realtime = year + "" + month + "" + date + "_" + hours + minutes + seconds + milliseconds;
        console.log(realtime);

        this.example_img = user_id + "_" + realtime + "_" + this.file.name

      },
      getSeedDetail: function () {
        const seedId = this.seedId
        axios.get(`http://i4a303.p.ssafy.io/api/detailChallenge/${seedId}`)
          .then((res) => {
            console.log(res.data)
            this.SeedInfo = res.data
            this.sum_img = res.data.sum_img
            console.log("받아온썸네일사진값:" + this.sum_img)
            this.content = res.data.content
            this.example_img = res.data.example_img
            this.max_people = res.data.max_people

            this.dates.push(this.SeedInfo.start_date)
            this.dates.push(this.SeedInfo.end_date)
          })
          .catch((err) => {
            console.log(err)
          })
      },
      UpdateSeed: function () {

        console.log("changed img: "+this.changedImg);

        const ChangedUserInfo = {
          id: this.seedId,
          category_id: this.SeedInfo.category_id,
          user_id: this.SeedInfo.user_id,
          title: this.SeedInfo.title,
          content: this.content,
          start_date: this.SeedInfo.start_date,
          end_date: this.SeedInfo.end_date,
          cert_count: this.SeedInfo.cert_count,
          max_people: this.max_people,
          sum_img: this.SeedInfo.sum_img,
          example_img: this.SeedInfo.example_img,
          join_deposit: this.SeedInfo.join_deposit,
          like_cnt: this.SeedInfo.like_cnt,
          result: this.SeedInfo.result,
          day: this.SeedInfo.day,
          week: this.SeedInfo.week
        }

        if (this.changedImg === false) {
          axios.put('http://i4a303.p.ssafy.io/api/updateChallenge/', ChangedUserInfo)
            .then((res) => {
              this.$router.push({
                name: 'SeedDetail',
                params: {
                  seedId: this.seedId
                }
              });
              console.log(res)
            })
            .catch((err) => {
              console.log(err)
            })
        } else {
          this.photoKey = this.example_img;
          ChangedUserInfo.example_img = this.photoURL + this.example_img

          AWS.config.update({

            region: this.bucketRegion,
            credentials: new AWS.CognitoIdentityCredentials({
              IdentityPoolId: this.IdentityPoolId
            })
          });

          const s3 = new AWS.S3({

            apiVersion: "2006-03-01",
            params: {
              Bucket: this.albumBucketName
            }
          });

          // AWS Setting End

          s3.upload({
              Key: this.example_img,
              Body: this.file,
              ACL: 'public-read'
            }, (err, data) => {
              if (err) {
                console.log(err)
                return alert('There was an error uploading your photo: ', err.message);
              }
              console.log(data);
            }

          );

          axios.put('http://i4a303.p.ssafy.io/api/updateChallenge/', ChangedUserInfo)
            .then((res) => {
              this.$router.push({
                name: 'SeedDetail',
                params: {
                  seedId: this.seedId
                }
              });
              console.log(res)
            })
            .catch((err) => {
              console.log(err)
            })
        }
      },
      checkForm: function () {
        if (this.sum_img.length > 1) {
          this.isSubmit.isThumbnail = true
          console.log("대표 사진:", this.isSubmit.isThumbnail)
        } else {
          this.isSubmit.isThumbnail = false
        }

        if (this.content.length >= 10 && this.content.length <= 200) {
          this.isSubmit.isContent = true
          console.log("내용:", this.isSubmit.isContent)
        } else {
          this.isSubmit.isContent = false
        }

        if (this.example_img.length > 1) {
          this.isSubmit.isCertification = true
          console.log("예시 이미지:", this.isSubmit.isCertification)
        } else {
          this.isSubmit.isCertification = false
        }

        if (this.max_people > 0) {
          this.isSubmit.isPeople = true
          console.log("날짜:", this.isSubmit.isPeople)
        } else {
          this.isSubmit.isPeople = false
        }

        const isAllSubmit = Object.values(this.isSubmit).every(v => {
          return v === true
        })
        console.log(Object.values(this.isSubmit))
        console.log("전체:", isAllSubmit)
        if (isAllSubmit) {
          this.isAllSubmit = true;
        } else {
          this.isAllSubmit = false;
        }

      }
    },
    watch: {
      sum_img: function () {
        this.checkForm();
      },
      content: function () {
        this.checkForm();
      },
      example_img: function () {
        this.checkForm();
      },
      max_people: function () {
        this.checkForm();
      }
    },
    created() {
      this.getSeedDetail();
    },
    computed: {
      dateRangeText() {
        return this.dates.join(' ~ ')
      },
      weekDay() {
        return `${this.SeedInfo.week}주 동안 주 ${this.SeedInfo.day}회`
      },
    }
  }
</script>

<style lang="scss" scoped>
  a {
    text-decoration: none;
  }

  .container-size {
    margin-top: 10vh;
    width: 80%;
  }

  .form-margin {
    margin-top: 10vh;
  }

  .caution-font {
    font-size: 18px;
  }

  .img-cursor {
    cursor: pointer
  }

  // .img-size {
  //   width: 10%
  // }
</style>