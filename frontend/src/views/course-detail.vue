<template>
  <div class="coursedetail">
    <Header :personal="true"></Header>
    <div class="courses">
      <div class="detailcard">
        <img class="detailimg" :src="posturl" />
        <p class="detailtitle">{{ classname }}</p>
        <h3 style="position: absolute; left: 460px; top: 120px">
          Last updated {{ updatetime }}
        </h3>
        <a-tag
          class="detag"
          style="
            color: black;
            background: rgba(255, 169, 40, 0.77);
            border: 1.5px solid #dc813e;
            box-sizing: border-box;
            border-radius: 20px;
          "
          color="rgba(255, 169, 40, 0.77)"
        >
          {{ tag }}
        </a-tag>
        <a-rate class="rate" v-model="score" disabled />
        <span class="ratetext"
          >{{ score }}&nbsp; &nbsp;( &nbsp;{{
            ratings > 0 ? ratings + " ratings" : ratings + "  rating"
          }}
          &nbsp;)</span
        >
        <h3 class="detdesc">{{ desc }}</h3>
        <!-- <a-button
          @click="trailCourse"
          style="background: rgba(62, 166, 199, 0.6); left: 460px"
          class="detbut"
          >Trial Course</a-button
        > -->
        <a-button
          @click="buycourse"
          style="
            background: rgba(62, 166, 199, 0.6);
            left: 450px;
            background: rgba(255, 169, 40, 0.77);
            border: 4px solid rgba(251, 153, 104, 0.7);
          "
          class="detbut"
          >Purchase</a-button
        >
      </div>
      <div class="detbot">
        <a-tabs default-active-key="1" @change="getComment">
          <a-tab-pane key="1" tab=" Chapters">
            <div class="demo-infinite-container">
              <a-list :data-source="chapterData" :loading="loading">
                <a-list-item
                  slot="renderItem"
                  slot-scope="item"
                  @click="trailCourse(item)"
                >
                  <a-badge v-show="item.isfree === 0">
                    <img class="freeic" src="../assets/free.png" slot="count" />
                  </a-badge>
                  <a-list-item-meta>
                    <h2 slot="title">{{ item.name }}</h2>
                    <a-avatar slot="avatar" :src="courseicon" />
                  </a-list-item-meta>
                </a-list-item>
              </a-list>
            </div>
          </a-tab-pane>
          <a-tab-pane key="2" tab="Comment" force-render>
            <div class="demo-infinite-container">
              <a-list
                class="comment-list"
                :header="`${commentdata.length} Comments`"
                item-layout="horizontal"
                :data-source="commentdata"
              >
                <a-list-item slot="renderItem" slot-scope="item">
                  <a-comment :author="item.username" :avatar="item.avatarurl">
                    <p slot="content">
                      {{ item.comment }}
                    </p>
                    <a-tooltip slot="datetime">
                      <span>{{ item.datetime }}</span>
                    </a-tooltip>
                  </a-comment>
                </a-list-item>
              </a-list>
            </div>
          </a-tab-pane>
        </a-tabs>
      </div>
    </div>
  </div>
</template>
<script>
import Vue from "vue";
import Header from "../components/header.vue";
import moment from "moment";
import axios from "axios";

// import Card from "../components/course-card.vue";
export default Vue.extend({
  components: {
    Header,
    // Card,
  },
  data() {
    return {
      // url: require("../assets/java.jpg"),
      // updatetime: "September 2021  ",
      tag: "BEST SELLER",
      score: 0,
      ratenum: "",
      loading: false,
      courseicon: require("../assets/vd.png"),
      busy: false,
      desc: "",
      coursedata: [],
      posturl: "",
      price: 0,
      classname: "",
      diff: "",
      updatetime: "",
      ratings: "",
      chapterData: [],
      commentdata: [],
      moment,
      cid: 0,
      uid: 0,
    };
  },

  created() {
    this.getparams();
    this.uid = parseInt(localStorage.getItem("uid"));
    this.cid = parseInt(localStorage.getItem("cid"));
    this.id = this.cid;
    console.log("uid", this.uid);
  },
  mounted() {
    this.detail();
    this.getChapter();
  },

  methods: {
    getparams() {
      this.id = this.$route.query.cid;
    },
    detail() {
      axios({
        method: "get",
        url: "http://localhost:9998/elec5620/main/detail?id=" + this.id,
      })
        .then((response) => {
          if (response.data.success === true) {
            this.coursedata = response.data.data;
            this.posturl = this.coursedata.posturl;
            this.price = this.coursedata.price;
            this.diff = this.coursedata.level;
            this.classname = this.coursedata.name;
            this.updatetime = this.coursedata.lastupdate;
            this.desc = this.coursedata.desc;
            this.score = parseInt(this.coursedata.score);
            this.ratings = parseInt(this.coursedata.ratings);
            // this.$router.push({
            //   path: "/course-detail",
            //   query: {
            //     data: this.data,
            //   },
            // });
          } else {
            this.$message.error("Course detail loading failed.");
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },
    getChapter() {
      this.loading = true;
      axios({
        method: "get",
        url: "http://localhost:9998/elec5620/main/chapter?id=" + this.id,
      })
        .then((response) => {
          this.loading = false;
          if (response.data.success === true) {
            this.chapterData = response.data.data;
          } else {
            this.$message.error("Chapter loading failed.");
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },
    getComment() {
      this.loading = true;
      axios({
        method: "get",
        url: "http://localhost:9998/elec5620/main/comment?id=" + this.id,
      })
        .then((response) => {
          this.loading = false;
          if (response.data.success === true) {
            this.commentdata = response.data.data;
            console.log("评论", this.commentdata);
          } else {
            this.$message.error("Comment loading failed.");
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },
    trailCourse(item) {
      this.$router.push({
        path: "/watch-video",
        query: {
          cid: this.cid,
          id: item.id,
          url: item.url,
          name: item.name,
          coursename: this.classname,
          price: this.price,
        },
      });
    },
    buycourse() {
      this.$router.push({
        path: "/payment",
        query: {
          coursedata: this.coursedata,
        },
      });
    },
  },
});
</script>
<style lang="less">
.coursedetail {
  width: 100%;
  height: 1500px;
  background: rgba(143, 189, 203, 0.6);
  .courses {
    display: block;
    margin-top: 250px;
    background: white;
    height: 100%;
    .detailcard {
      position: absolute;

      top: 200px;
      width: 78.5%;
      left: 11.6%;
      right: 11.6%;
      height: 350px;
      box-shadow: 0px 4px 4px rgba(0, 0, 0, 0.5);
      background: #ffffff;
      .detailimg {
        position: absolute;
        border-radius: 40px;
        width: 364px;
        height: 200px;
        left: 60px;
        top: 70px;
      }
      .detailtitle {
        position: absolute;
        left: 460px;
        top: 70px;
        font-weight: bold;
        font-size: 28px;
        line-height: 42px;

        color: #000000;
      }
      .detag {
        position: absolute;
        left: 460px;
        top: 155px;
      }
      .rate {
        position: absolute;
        left: 580px;
        top: 150px;
      }
      .ratetext {
        position: absolute;
        left: 730px;
        top: 155px;
      }
      .detdesc {
        position: absolute;
        left: 460px;
        top: 200px;
        width: 600px;
        max-height: 200px;
        font-size: 14px;
        text-align: left;
        line-height: 16px;
        color: #000000;
        font-weight: lighter;
        overflow: hidden;
        text-overflow: ellipsis;
        display: -webkit-box;
        -webkit-line-clamp: 4;
        line-clamp: 4;
        -webkit-box-orient: vertical;
      }
      .detdesc:hover {
        text-overflow: inherit;
        overflow: visible;
        white-space: pre-line; /*合并空白符序列，但是保留换行符。*/
      }
      .detbut {
        text-align: center;
        font-size: 17px;
        font-weight: bold;
        position: absolute;
        width: 130px;
        height: 60px;
        top: 270px;
        background: rgba(62, 166, 199, 0.6);
        border: 4px solid rgba(62, 166, 199, 0.6);
        box-sizing: border-box;
        box-shadow: 0px 4px 4px rgba(0, 0, 0, 0.25);
        border-radius: 60px;
      }
    }
    .detbot {
      position: absolute;

      top: 600px;
      width: 78.5%;
      left: 11.6%;
      right: 11.6%;
      height: 400px;
      .freeic {
        // float: left;
        width: 50px;
        height: 55px;
      }
      .ant-tabs-nav .ant-tabs-tab-active {
        background: #acd9e7;
        border-radius: 15px;
        color: black;
      }
      .ant-tabs-ink-bar {
        background: transparent;
      }
      .demo-infinite-container {
        border: 1px solid #e8e8e8;
        border-radius: 4px;
        overflow: auto;
        padding: 8px 24px;
        max-height: 300px;
      }
      .demo-loading-container {
        position: absolute;
        bottom: 40px;
        width: 100%;
        text-align: center;
      }
      .ant-list-item {
        flex-direction: column;
        flex-wrap: nowrap;
        align-content: space-around;
        justify-content: center;
        align-items: flex-start;
        .ant-list-item-meta {
          text-align: left;
          width: 600px;
        }
      }
      .ant-comment-content-detail {
        text-align: left;
      }
    }
  }
}
</style>
