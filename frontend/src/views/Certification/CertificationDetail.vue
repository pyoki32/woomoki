<template>
    <div>

    
        <!-- <v-row> -->
        <v-col class="cng-name">
            <router-link :to="{ name: 'SeedDetail', params: { seedId: CertInfo.cng_id }}">
                <span class="d-flex justify-center align-center mb-3 title-size">
                    {{ CertInfo.title }}
                </span>
            </router-link>
            <span class="d-flex justify-center">
                <v-chip class="white--text" :color="color">
                    {{ this.category }}
                </v-chip>
            </span>
        </v-col>
        <!-- </v-row> -->
    <v-container class="container-size">
        <v-row>

        
        <v-list-item class="border-list">
            <router-link :to="{ name: 'UserPage', params: { userNickname: CertInfo.nickname }}">
                <v-list-item-avatar size="55">
                    <v-img :src="CertUserImg"></v-img>
                </v-list-item-avatar>
            </router-link>

            <v-list-item-content>
                <v-list-item-title>
                    <router-link :to="{ name: 'UserPage', params: { userNickname: CertInfo.nickname }}">
                        <span class="nickname-bold" style="font-size:15px; color:black;"> {{ CertInfo.nickname }}</span>
                    </router-link>

                </v-list-item-title>
                <v-list-item-subtitle>
                    <span class="date">{{ CertInfo.create_date }}</span>
                </v-list-item-subtitle>
            </v-list-item-content>
            <CertificationShare></CertificationShare>
            <v-menu v-if="checkUser" offset-y open-on-click bottom left>
                
                <template v-slot:activator="{ on, attrs }">
                   <v-btn icon color="black" v-bind="attrs" v-on="on">
                        <v-icon>fas fa-ellipsis-v</v-icon>
                    </v-btn>
                </template>
                <v-list>
                    <v-list-item>
                        <v-list-item-title>
                            <v-btn class="ma-2" icon color="#AED864" v-on:click="updateCert()">
                                <v-btn plain text>수정</v-btn>
                            </v-btn>
                        </v-list-item-title>
                    </v-list-item>
                    <v-list-item>
                        <v-list-item-title>
                            <v-dialog v-model="dialog" persistent max-width="290">
                                <template v-slot:activator="{ on, attrs }">
                                    <v-btn class="ma-2" icon color="#EF5350" v-bind="attrs" v-on="on">
                                        <v-btn plain text>삭제</v-btn>
                                    </v-btn>
                                </template>
                                <v-card>
                                    <v-card-title class="headline">
                                        인증글을 삭제하시겠습니까?
                                    </v-card-title>
                                    <v-card-text>한번 삭제된 인증글은 되돌릴 수 없습니다! 인증글을 삭제 시, 재인증을 시도해주세요.</v-card-text>
                                    <v-card-actions>
                                        <v-spacer></v-spacer>
                                        <v-btn color="green darken-1" text @click="dialog = false">
                                            취소
                                        </v-btn>
                                        <v-btn color="red darken-1" text @click="dialog = false; deleteCert()">
                                            삭제
                                        </v-btn>
                                    </v-card-actions>
                                </v-card>
                            </v-dialog>
                        </v-list-item-title>
                    </v-list-item>
                </v-list>
            </v-menu>
        </v-list-item>
        </v-row>

        <div class="detail">

            <v-row class="img">
                <!-- s3 주소 주석 풀기
                <v-img :src="photoUrl + CertInfo.img"></v-img> -->
                <v-img width="35vw" height="35vw" :src="CertInfo.img"></v-img>
            </v-row>
            <v-row class="d-flex justify-start content">
                <v-col class="d-flex align-center div-content">
                    <router-link :to="{ name: 'UserPage', params: { userNickname: CertInfo.nickname }}">
                        <span class="nickname-bottom">{{ CertInfo.nickname }}</span>
                    </router-link>
                </v-col>

                <v-col class="d-flex justify-end">
                    <v-btn icon x-large @click="getScrap">
                        <v-icon :color="scrapped ? 'red' : '' ">mdi-heart</v-icon>
                        </v-btn>
                </v-col>
            </v-row>
            <v-divider></v-divider>
            <v-row class="content-margin">
                <div class="ml-7">{{ CertInfo.content }}</div>
            </v-row>
            <v-row class="good-bad-btn" v-if="showResultBtn && CertInfo.result==0">
                <v-btn class="ma-2" outlined fab color="blue" @click.stop="dialogSuccess = true">
                    <v-icon>mdi-thumb-up</v-icon>
                </v-btn>

                <v-dialog v-model="dialogSuccess" max-width="350">
                    <v-card>
                        <v-card-title class="headline">
                            인증 결과가 <span style="color:blue; margin-left:2%;">성공</span> 인가요?
                        </v-card-title>

                        <v-card-text>
                            한번 등록된 인증 결과는 번복 할 수 없어요! <br>신중히 결정해주세요.
                        </v-card-text>

                        <v-card-actions>
                            <v-spacer></v-spacer>

                            <v-btn color="grey" text @click="dialogSuccess = false">
                                취소
                            </v-btn>

                            <v-btn color="green darken-1" text @click="dialogSuccess = false; resultSuccess()">
                                계속
                            </v-btn>
                        </v-card-actions>
                    </v-card>
                </v-dialog>
                <v-btn class="ma-2" outlined fab color="red" @click.stop="dialogFail = true">
                    <v-icon>mdi-thumb-down</v-icon>
                </v-btn>
                <v-dialog v-model="dialogFail" max-width="350">
                    <v-card>
                        <v-card-title class="headline">
                            인증 결과가 <span style="color:red; margin-left:2%; ">실패</span>인가요?
                        </v-card-title>

                        <v-card-text>
                            한번 등록된 인증 결과는 번복 할 수 없어요! <br> 신중히 결정해주세요.
                        </v-card-text>

                        <v-card-actions>
                            <v-spacer></v-spacer>

                            <v-btn color="grey" text @click="dialogFail = false">
                                취소
                            </v-btn>

                            <v-btn color="green darken-1" text @click="dialogFail = false; resultFail()">
                                계속
                            </v-btn>
                        </v-card-actions>
                    </v-card>
                </v-dialog>

            </v-row>
            <v-row class="result-done" v-if="showResultBtn && CertInfo.result!=0">
                확인이 완료된 인증글입니다.
            </v-row>
        <v-divider></v-divider>
        <CommentInsert />
        </div>
        </v-container>
        <v-container class="bottom-container-size">

        <CommentList v-for="(comment, index) in comments" :key="index" :comment="comment" />

        </v-container>
    </div>
</template>

<script>
    import CommentInsert from '@/views/Certification/components/CommentInsert.vue'
    import CommentList from '@/views/Certification/components/CommentList.vue'
    import CertificationShare from '@/views/Certification/components/CertificationShare.vue'
    import {
        mapState
    } from "vuex";
    import axios from "axios";

    export default {
        name: 'CertificationDetail',
        components: {
            CommentList,
            CommentInsert,
            CertificationShare
        },
        data() {
            return {
                CertUserImg: "",
                CertInfo: [],
                comments: [],
                UserInfo: [],
                Nicknames: [],
                ProfileImgs: [],
                // photoUrl:"https://s3.ap-northeast-2.amazonaws.com/cert-photo-upload/",
                dialog: false,
                dialogFail: false,
                dialogSuccess: false,
                scrapped: false,
                confirmed: false,
                checkUser: false,
                showResultBtn: false,
                likeCount: 0,
                category: "",
                color: "",
            };
        },
        computed: {
            ...mapState('UserStore', ['user']),
            getCheckSeedOwner() {
                if (this.user.user_id === this.$route.params.cngUserId) {
                    return true
                } else {
                    return false
                }
            },
        },
        created() {
            this.getCategory()
            this.CheckisLikeSeed();
            this.detailCert();
            this.detailComment();
            const certId = this.$route.params.certId;
            const cngId = this.$route.params.cngId;
            const cngUserId = this.$route.params.cngUserId;
            if (certId === undefined || cngId === undefined || cngUserId === undefined) {
                this.$router.go(-1);
            }
            // console.log("check create: " + this.checkUser);
        },
        methods: {
            async detailCert() {
                const userId = this.$store.state.UserStore.user.user_id;
                console.log("userid: " + userId);
                const certId = this.$route.params.certId;
                await axios.get(`http://i4a303.p.ssafy.io/api/detailCertification/${certId}`)
                    .then((response) => {
                        this.CertInfo = response.data;
                        const certUserId = this.CertInfo.user_id;
                        const cngUserId = this.$route.params.cngUserId;
                        console.log("지금 인증글에 해당하는 결과: " + this.CertInfo.result)

                        console.log("cngUserId: " + cngUserId);
                        console.log("certUserId: " + certUserId);
                        if (userId === certUserId) {
                            this.checkUser = true;
                        }

                        console.log("cngUserId: " + cngUserId);
                        if (userId === Number(cngUserId)) {
                            this.showResultBtn = true;
                        }
                        console.log(response.data)
                        this.getCertUserImg();
                    })
                    .catch((err) => {
                        console.log(err)
                    })
            },
            detailComment: function () {
                const certId = this.$route.params.certId;
                console.log(certId);
                axios.get(`http://i4a303.p.ssafy.io/api/commentList/${certId}`)
                    .then((response) => {
                        console.log(response.data);
                        this.comments = response.data;
                        for (const i in this.comments) {
                            const comment = this.comments[i]
                            // const commentUserId = comment["user_id"]
                            // // console.log("cmt user id: " + commentUserId);
                            // this.getUserInfo(commentUserId);
                        }
                    })
                    .catch((err) => {
                        console.log(err)
                    })
            },

            // 댓글단 유저 정보
            async getUserInfo(commentUserId) {
                // console.log("getUserInfo를 들어왔ㅇㅓ : " + commentUserId);
                await axios.get(`http://i4a303.p.ssafy.io/api/userPage/Id/${commentUserId}`)
                    .then((response) => {
                        // console.log(response.data);
                        this.UserInfo = response.data;
                        this.Nicknames.push(this.UserInfo.nickname)
                        this.ProfileImgs.push(this.UserInfo.img)
                        // console.log("nickname: " + this.UserInfo.nickname);
                        // console.log(this.Nicknames);
                        // this.saveNickname();
                    })

                // },
                // saveNickname(){
                //     for(const i in this.comments.length){
                //         console.log("nickname value: " + this.UserInfo.nickname[i]);
                //         this.comments.push({key:i, value:this.UserInfo.nickname[i]});
                //     }
                //     // console.log("commments: " + this.comments);
                // },
            },
            updateCert: function () {
                this.$router.push({
                    name: 'CertificationUpdate',
                    params: {
                        cngId: this.$route.params.cngId,
                        certId: this.$route.params.certId,
                        cngUserId: this.$route.params.cngUserId,
                    }
                });
            },
            deleteCert() {
                const certId = this.$route.params.certId;
                axios.delete(`http://i4a303.p.ssafy.io/api/deleteCertification/${certId}`)
                    .then((response) => {
                        console.log(response.data)
                        this.$router.push({
                            name: 'SeedDetail',
                            params: {
                                seedId: this.$route.params.cngId
                            }
                        });
                    })
                    .catch((err) => {
                        console.log(err)
                    })

            },
            getScrap() {
                // 클릭 때 마다 scrapped가 토글
                this.scrapped = !this.scrapped
                const certId = this.$route.params.certId;
                const userId = this.$store.state.UserStore.user.user_id
                // 스크랩이 되어있지 않을 때 스크랩
                if (this.scrapped) {
                    axios.put(`http://i4a303.p.ssafy.io/api/likeUpCertification/${userId}/${certId}`)
                        .then((res) => {
                            this.likeCount += 1
                            console.log(res)
                        })
                        .catch((err) => {
                            console.log(err)
                        })
                } else {
                    // 스크랩 되어 있을 때 스크랩 취소
                    axios.put(`http://i4a303.p.ssafy.io/api/likeDownCertification/${userId}/${certId}`)
                        .then((res) => {
                            this.likeCount -= 1
                            console.log(res)
                        })
                        .catch((err) => {
                            console.log(err)
                        })
                }
            },
            CheckisLikeSeed: function () {
                const certId = this.$route.params.certId;
                const userId = this.$store.state.UserStore.user.user_id
                axios.get(`http://i4a303.p.ssafy.io/api/LikeAndCertification/${certId}`)
                    .then((res) => {
                        const UserList = res.data
                        this.likeCount = UserList.length

                        for (var i = 0; i < UserList.length; i++) {
                            if (UserList[i].id === Number(userId)) {
                                this.scrapped = true
                            }
                        }
                    })

            },
            getCategory: function () {
                const seedId = this.$route.params.cngId
                axios.get(`http://i4a303.p.ssafy.io/api/detailChallenge/${seedId}`)
                    .then((res) => {
                        if (res.data.category_id === '1') {
                            this.category = "건강"
                            this.color = "light-blue lighten-1"
                        } else if (res.data.category_id === '2') {
                            this.category = "생활습관"
                            this.color = "orange lighten-1"
                        } else if (res.data.category_id === '3') {
                            this.category = "독서"
                            this.color = "teal lighten-1"
                        } else if (res.data.category_id === '4') {
                            this.category = "자산"
                            this.color = "indigo lighten-1"
                        } else if (res.data.category_id === '5') {
                            this.category = "자기계발"
                            this.color = "purple lighten-1"
                        } else {
                            this.category = "취미"
                            this.color = "pink lighten-1"
                        }
                        console.log("카테고리", this.category)
                    })
                    .catch((err) => {
                        console.log(err)
                    })
            },
            resultSuccess() {

                const certId = this.$route.params.certId;
                const UpdateCertInfo = {
                    cng_id: this.CertInfo.cng_id,
                    content: this.CertInfo.content,
                    id: certId,
                    img: this.CertInfo.img,
                    user_id: this.CertInfo.user_id,
                    like_cnt: this.CertInfo.like_cnt,
                    result: 3,
                    current_week: this.CertInfo.current_week,
                    current_day: this.CertInfo.current_day,
                }

                axios.put("http://i4a303.p.ssafy.io/api/updateCertification", UpdateCertInfo)
                    .then(res => {
                        console.log(res);
                        this.$router.go(this.$router.currentRoute);
                    })
                    .catch(err => {
                        console.log(err);
                    });

                //  this.showResultBtn = false;

            },
            resultFail() {
                const certId = this.$route.params.certId;
                const UpdateCertInfo = {
                    cng_id: this.CertInfo.cng_id,
                    content: this.CertInfo.content,
                    id: certId,
                    img: this.CertInfo.img,
                    user_id: this.CertInfo.user_id,
                    like_cnt: this.CertInfo.like_cnt,
                    result: 1,
                    current_week: this.CertInfo.current_week,
                    current_day: this.CertInfo.current_day,
                }

                axios.put("http://i4a303.p.ssafy.io/api/updateCertification", UpdateCertInfo)
                    .then(res => {
                        console.log(res);
                    })
                    .catch(err => {
                        console.log(err);
                    });

                this.showResultBtn = false;
            },
            getCertUserImg() {
                const certUser = this.CertInfo.user_id;
                axios.get(`http://i4a303.p.ssafy.io/api/userPage/Id/${certUser}`)
                    .then((res) => {
                        console.log('내 프로필 잘 나와야하는데')
                        console.log(res)
                        this.CertUserImg = res.data.img
                    })
            },
        }
    }
</script>

<style lang="scss" scoped>
    a {
        text-decoration: none;
    }

    .detail {
        width: 100%;
        // margin: 10% 30% 0% 30%;
    }

    .v-divider {
        margin: 3% !important;
    }

    .result-btn-row {
        justify-content: center;
        margin-top: 3%;
    }

    .like-btn {
        float: right;
        margin-top: 5%;
        margin-bottom: 5%;
    }

    .edit-del-btn {
        justify-content: center;
        margin-top: 3%;
        margin-bottom: 3%;
    }

    .img,
    .content,
    .nickname-date-row {
        justify-content: center;
        margin-top: 2%;
    }

    .div-content {
        word-break: break-all; // 영어 범위 벗어남 해결
    }


    .cng-name {
        justify-content: center;
        font-style: italic;
        margin-bottom: 2%;
        margin-top: 2%;
    }

    .container-size {
        border: solid rgb(231, 231, 231);
        width: 40vw;
    }

    .bottom-container-size {
        // border: solid rgb(231, 231, 231);
        width: 40vw;
    }

    .nickname-bold {
        color: black;
        font-size: 17px;
        font-weight: bold;
        // margin-left: 1vw;
    }

    .nickname-bottom {
        color: black;
        font-size: 17px;
        font-weight: bold;
        margin-left: 1vw;
    }

    .title-size {
        color: black;
        font-size: 25px;
    }

    .border-list {
        border: thin solid rgb(231, 231, 231);
        margin: 0px;
    }

    .good-bad-btn,
    .result-done {
        margin-top: 5%;
        margin-bottom: 5%;
        justify-content: center;
    }

    a {
        text-decoration: none;
    }

    .content-flex {
        margin-left: 2vw;
    }

    .content-margin {
        margin-top: 5%;
    }
</style>