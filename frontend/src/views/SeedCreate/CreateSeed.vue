<template>
  <v-container class="container-size">

    <v-stepper non-linear v-model="e1">
      <v-stepper-header>
        <v-stepper-step color="#AED864" complete-icon="fas fa-check-circle" :complete="e1 > 1" step="1">
          씨앗 종류
        </v-stepper-step>
        <v-divider></v-divider>

        <v-stepper-step color="#AED864" complete-icon="fas fa-check-circle" :complete="e1 > 2" step="2">
          씨앗 내용
        </v-stepper-step>
        <v-divider></v-divider>

        <v-stepper-step color="#AED864" complete-icon="fas fa-check-circle" step="3">
          기타
        </v-stepper-step>
      </v-stepper-header>

      <v-stepper-items>
        <v-stepper-content step="1">
          <SeedCategory class="pt-3" @transferLifestyle="receiveLifestyle" @transferHealth="receiveHealth"
            @transferBook="receiveBook" @transferAssets="receiveAssets" @transferGrown="receiveGrown"
            @transferHobby="receiveHobby"></SeedCategory>

          <div class="d-flex justify-end mb-1">

            <router-link to="/">
              <v-btn text class="ml-2 mr-2">
                취소
              </v-btn>
            </router-link>

            <v-btn color="#AED864" class="seed-btn" @click="e1 = 2" :disabled="isSubmitCategory === false">
              다음
            </v-btn>


          </div>
        </v-stepper-content>

        <v-stepper-content step="2">
          <SeedThumbnail @transferThumbnail="receiveThumbnail" class="mb-5"></SeedThumbnail>
          <SeedTitle @transferTitle="receiveTitle"></SeedTitle>
          <SeedContent @transferContent="receiveContent"></SeedContent>
          <div class="d-flex justify-end mb-1">


            <router-link to="/">
              <v-btn text class="ml-2 mr-2">
                취소
              </v-btn>
            </router-link>
            <v-btn text @click="e1 = 1" style="margin-right:1%">
              뒤로가기
            </v-btn>
            <v-btn color="#AED864" @click="[e1 = 3, uploadThumbnail()]" class="seed-btn"
              :disabled="BasicInfo.isSubmitBasicInfo === false">
              다음
            </v-btn>
          </div>
        </v-stepper-content>

        <v-stepper-content step="3">
          <SeedCertificationImg @transferCertificationImg="receiveCertificationImg" class="mb-5"></SeedCertificationImg>
          <SeedCertificationWeek @transferWeek="receiveWeek"></SeedCertificationWeek>
          <SeedCertificationDay @transferDay="receiveDay"></SeedCertificationDay>
          <SeedDate @transferDate="receiveDate"></SeedDate>
          <SeedPeople @transferPeople="receivePeople"></SeedPeople>
          <SeedDeposit @transferDeposit="receiveDeposit"></SeedDeposit>
          <SeedCheckbox @transferCheckbox="receiveCheckbox"></SeedCheckbox>

          <div class="text-end mb-2">
            

            <router-link to="/">
              <v-btn text class="ml-2 mr-2">
                취소
              </v-btn>
            </router-link>

            <v-btn text @click="e1 = 2" style="margin-right:1%">
              뒤로가기
            </v-btn>

            <v-btn color="#AED864" class="seed-btn" @click="InsertSeed" :disabled="EtcInfo.isSubmitEtcInfo === false">
              생성
            </v-btn>
          </div>


        </v-stepper-content>
      </v-stepper-items>
    </v-stepper>

  </v-container>
</template>

<script>
  import SeedThumbnail from "./components/SeedThumbnail.vue"
  import SeedTitle from "./components/SeedTitle.vue"
  import SeedContent from "./components/SeedContent.vue"
  import SeedDate from "./components/SeedDate.vue"
  import SeedCategory from "./components/SeedCategory.vue"
  import SeedPeople from "./components/SeedPeople.vue"
  import SeedDeposit from "./components/SeedDeposit.vue"
  import SeedCertificationImg from "./components/SeedCertificationImg.vue"
  import SeedCertificationWeek from "./components/SeedCertificationWeek.vue"
  import SeedCertificationDay from "./components/SeedCertificationDay.vue"
  import SeedCheckbox from "./components/SeedCheckbox.vue"
  import axios from 'axios'

  export default {
    name: 'CreateSeed',
    components: {
      SeedThumbnail,
      SeedTitle,
      SeedContent,
      SeedDate,
      SeedCategory,
      SeedPeople,
      SeedDeposit,
      SeedCertificationImg,
      SeedCertificationWeek,
      SeedCertificationDay,
      SeedCheckbox
    },
    data: function () {
      return {
        e1: 1,
        slides: 7,
        sheet: false,
        userId: this.$store.state.UserStore.user.user_id,
        category: "",
        thumbnail: "",
        title: "",
        content: "",
        certificationImg: "",
        like_cnt: 0,
        dates: "",
        start_date: "",
        end_date: "",
        week: "",
        day: "",
        people: 0,
        joinDeposit: "",
        checkbox: false,
        isSubmitCategory: false,
        BasicInfo: {
          isSubmit: {
            isSubmitThumbnail: false,
            isSubmitTitle: false,
            isSubmitContent: false,
          },
          isSubmitBasicInfo: false,
        },
        EtcInfo: {
          isSubmit: {
            isCertificationImg: false,
            isDate: false,
            isWeek: false,
            isDay: false,
            isPeople: false,
            isjoinDeposit: false,
            isCheckbox: false,
          },
          isSubmitEtcInfo: false,
        },
        isSubmitTotalSeedData: false,

        albumBucketName: "cert-photo-upload",
        bucketRegion: "ap-northeast-2",
        IdentityPoolId: "ap-northeast-2:8cf7cb29-d051-4f38-885f-09b1e4dd8153",

        photoURL: "https://s3.ap-northeast-2.amazonaws.com/cert-photo-upload/",
        photoKey: null,
        thumbnailFile: null,
        certExampleImg: null,
        checkThumbnail: false,

      }
    },
    methods: {

      InsertSeed: function () {

        function getDateStr(myDate) {
          if ((myDate.getMonth() + 1) < 10 && myDate.getDate() >= 10) {
            return (myDate.getFullYear() + '-0' + (myDate.getMonth() + 1) + '-' + myDate.getDate())
          } else if ((myDate.getMonth() + 1) >= 10 && myDate.getDate() < 10) {
            return (myDate.getFullYear() + '-' + (myDate.getMonth() + 1) + '-0' + myDate.getDate())
          } else if ((myDate.getMonth() + 1) < 10 && myDate.getDate() < 10) {
            return (myDate.getFullYear() + '-0' + (myDate.getMonth() + 1) + '-0' + myDate.getDate())
          } else {
            return (myDate.getFullYear() + '-' + (myDate.getMonth() + 1) + '-' + myDate.getDate())
          }

        }

        const days = this.week * 7
        const start_year = this.dates.slice(0, 4)
        const start_date = new Date(Number(start_year), Number(this.dates.slice(5, 7)) - 1, Number(this.dates.slice(
          8, 10)));
        var dayOfMonth = start_date.getDate()
        start_date.setDate(dayOfMonth + days)
        const end_date = getDateStr(start_date)
        console.log('end_date', end_date)

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
            Key: this.certificationImg,
            Body: this.certExampleImg,
            ACL: 'public-read'
          }, (err, data) => {
            if (err) {
              console.log(err)
              return alert('There was an error uploading your photo: ', err.message);
            }
            console.log(data);
          }

        );

        this.thumbnail = this.photoURL + this.thumbnail
        this.certificationImg = this.photoURL + this.certificationImg

        const SeedData = {
          category_id: this.category,
          cert_count: 0,
          content: this.content,
          end_date: end_date,
          example_img: this.certificationImg,
          join_deposit: Number(this.joinDeposit),
          like_cnt: this.like_cnt,
          max_people: this.people,
          day: this.day,
          week: this.week,
          start_date: this.dates,
          sum_img: this.thumbnail,
          title: this.title,
          user_id: this.userId
        }

        axios.post("http://i4a303.p.ssafy.io/api/insertChallenge", SeedData)
          .then((res) => {
            this.$router.push({
              name: "Main"
            })
          })
          .catch((err) => {
            console.log(err)
          })


      },
      uploadThumbnail() {
        console.log("썸네일 들어옴: " + this.thumbnail);

        // AWS Setting Start

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
            Key: this.thumbnail,
            Body: this.thumbnailFile,
            ACL: 'public-read'
          }, (err, data) => {
            if (err) {
              console.log(err)
              return alert('There was an error uploading your photo: ', err.message);
            }
            console.log(data);
          }

        );


      },
      receiveHealth: function (isSubmitCategory) {
        this.isSubmitCategory = isSubmitCategory
        this.category = 1
        console.log(this.category)
      },
      receiveLifestyle: function (isSubmitCategory) {
        this.isSubmitCategory = isSubmitCategory
        this.category = 2
        console.log(this.category)
      },
      receiveBook: function (isSubmitCategory) {
        this.isSubmitCategory = isSubmitCategory
        this.category = 3
        console.log(this.category)
      },
      receiveAssets: function (isSubmitCategory) {
        this.isSubmitCategory = isSubmitCategory
        this.category = 4
        console.log(this.category)
      },
      receiveGrown: function (isSubmitCategory) {
        this.isSubmitCategory = isSubmitCategory
        this.category = 5
        console.log(this.category)
      },
      receiveHobby: function (isSubmitCategory) {
        this.isSubmitCategory = isSubmitCategory
        this.category = 6
        console.log(this.category)
      },
      receiveThumbnail: function (file, thumbnail, checkThumbnail) {
        this.thumbnailFile = file
        this.thumbnail = thumbnail
        this.BasicInfo.isSubmit.isSubmitThumbnail = checkThumbnail
        console.log("넘어온 file정보: " + this.thumbnailFile)
        console.log("넘어온 thumbnail이름 : " + this.thumbnail)
      },
      receiveTitle: function (title) {
        this.title = title
      },
      receiveContent: function (content) {
        this.content = content
      },
      receiveCertificationImg: function (file, certificationImg, checkCertImg) {
        this.certExampleImg = file
        this.certificationImg = certificationImg
        this.EtcInfo.isSubmit.isCertificationImg = checkCertImg
      },
      receiveDate: function (dates) {
        this.dates = dates
      },
      receiveWeek: function (week) {
        this.week = week
      },
      receiveDay: function (day) {
        this.day = day
      },
      receivePeople: function (people) {
        this.people = people
      },
      receiveDeposit: function (joinDeposit) {
        this.joinDeposit = joinDeposit
      },
      receiveCheckbox: function (checkbox) {
        this.checkbox = checkbox
      },
      // 검사
      checkFormBasicInfo: function () {

        // 씨앗 이름 검사
        if (this.title.length >= 5 && this.title.length <= 20) {
          this.BasicInfo.isSubmit.isSubmitTitle = true
          console.log("이름:", this.BasicInfo.isSubmit.isSubmitTitle)
        } else {
          this.BasicInfo.isSubmit.isSubmitTitle = false
        }
        // 씨앗 내용 검사
        if (this.content.length >= 10 && this.content.length <= 200) {
          this.BasicInfo.isSubmit.isSubmitContent = true
          console.log("내용:", this.BasicInfo.isSubmit.isSubmitContent)
        } else {
          this.BasicInfo.isSubmit.isSubmitContent = false
        }
        // 씨앗 사진, 이름, 내용 중 하나라도 false면 isSubmitBasicInfo는 false
        const isAllSubmitBasicInfo = Object.values(this.BasicInfo.isSubmit).every(v => {
          return v === true
        })
        if (isAllSubmitBasicInfo) {
          this.BasicInfo.isSubmitBasicInfo = true;
        } else {
          this.BasicInfo.isSubmitBasicInfo = false;
        }
      },
      checkFormEtc: function () {
        // 예시 이미지 검사

        // 날짜 검사
        if (this.dates.length > 1) {
          this.EtcInfo.isSubmit.isDate = true
          console.log("날짜:", this.EtcInfo.isSubmit.isDate)
        } else {
          this.EtcInfo.isSubmit.isDate = false
        }

        // 참여 인원
        if (this.people > 0) {
          this.EtcInfo.isSubmit.isPeople = true
          console.log("사람:", this.EtcInfo.isSubmit.isPeople)
        } else {
          this.EtcInfo.isSubmit.isPeople = false
        }

        // 몇 주
        if (this.week > 0) {
          this.EtcInfo.isSubmit.isWeek = true
        } else {
          this.EtcInfo.isSubmit.isWeek = false
          console.log("몇 주:", this.EtcInfo.isSubmit.isWeek)
        }

        // 며칠
        if (this.day > 0) {
          this.EtcInfo.isSubmit.isDay = true
          console.log("며칠:", this.EtcInfo.isSubmit.isDay)
        } else {
          this.EtcInfo.isSubmit.isDay = false
        }

        // 참여 금액 검사
        if (this.joinDeposit.length <= 5 && /^[0-9]+$/.test(this.joinDeposit)) {
          this.EtcInfo.isSubmit.isjoinDeposit = true
          console.log("금액:", this.EtcInfo.isSubmit.isjoinDeposit)
        } else {
          this.EtcInfo.isSubmit.isjoinDeposit = false
        }

        // 체크박스 검사
        if (this.checkbox === true) {
          this.EtcInfo.isSubmit.isCheckbox = true
          console.log("체크박스:", this.EtcInfo.isSubmit.isCheckbox)
        } else {
          this.EtcInfo.isSubmit.isCheckbox = false
        }

        const isAllSubmitEtcInfo = Object.values(this.EtcInfo.isSubmit).every(v => {
          return v === true
        })
        console.log(Object.values(this.EtcInfo.isSubmit))
        console.log("전체:", isAllSubmitEtcInfo)
        if (isAllSubmitEtcInfo) {
          this.EtcInfo.isSubmitEtcInfo = true;
        } else {
          this.EtcInfo.isSubmitEtcInfo = false;
        }
      }
    },
    watch: {
      thumbnail: function () {
        this.checkFormBasicInfo();
      },
      title: function () {
        this.checkFormBasicInfo();
      },
      content: function () {
        this.checkFormBasicInfo();
      },
      certificationImg: function () {
        this.checkFormEtc();
      },
      dates: function () {
        this.checkFormEtc();
      },
      week: function () {
        this.checkFormEtc();
      },
      day: function () {
        this.checkFormEtc();
      },
      people: function () {
        this.checkFormEtc();
      },
      joinDeposit: function () {
        this.checkFormEtc();
      },
      checkbox: function () {
        this.checkFormEtc();
      },
    }
  }
</script>

<style scoped>
  a {
    text-decoration: none;
  }

  .container-size {
    width: 50%;
  }

  .seed-btn {
    color: white;
  }
</style>