<template>
  <div class="watchvideo">
    <Header></Header>
    <div class="videocontainer">
      <a-empty
        class="empty"
        v-if="free === 1"
        image="https://gw.alipayobjects.com/mdn/miniapp_social/afts/img/A*pevERLJC9v0AAAAAAAAAAABjAQAAAQ/original"
        :image-style="{
          height: '260px',
        }"
      >
        <span slot="description"> The trial is over, please pay first </span>
        <a-button
          type="primary"
          style="font-size: 18px; font-weight: bold"
          @click="gobuy"
          >>Purchase Course</a-button
        >
      </a-empty>
      <video-player
        :key="id"
        v-if="free === 0 || free === 2"
        class="video-player vjs-custom-skin"
        ref="videoPlayer"
        :playsinline="true"
        :options="playerOptions"
      />
    </div>
    <div class="videobot">
      <a-tabs default-active-key="1">
        <a-tab-pane key="1" tab=" Chapters">
          <div class="demo-infinite-container1">
            <a-list :split="false" :data-source="chapterData">
              <a-list-item
                slot="renderItem"
                slot-scope="item"
                @click="loadvideo(item)"
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
        <a-tab-pane key="2" tab="Recommend">
          <h3 class="recommendtitle">Related Courses Recommendation</h3>
          <a-row :gutter="16">
            <a-col
              :span="8"
              style="margin-top: 30px"
              v-for="item in recommenddata"
              :key="item.id"
            >
              <Card
                :id="item.id"
                :title="item.name"
                :diff="item.level"
                :price="item.price"
                :url="item.posturl"
              ></Card>
            </a-col>
          </a-row>
        </a-tab-pane>
        <a-tab-pane key="3" tab="Comment" force-render>
          <div class="demo-infinite-container2">
            <a-empty
              description="No comment yet"
              v-if="!commentdata.length"
            ></a-empty>
            <a-list
              v-if="commentdata.length"
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
                    <span>{{ item.commenttime }}</span>
                  </a-tooltip>
                </a-comment>
              </a-list-item>
            </a-list>
          </div>
          <a-comment>
            <a-avatar slot="avatar" :src="posturl" />
            <div slot="content">
              <a-form-item>
                <a-textarea :rows="4" :value="value" @change="handleChange" />
                <span>
                  <a-rate v-model="score" :tooltips="startip" />
                  <span class="ant-rate-text">{{ startip[value - 1] }}</span>
                </span>
              </a-form-item>
              <a-form-item>
                <a-button
                  html-type="submit"
                  :loading="submitting"
                  type="primary"
                  style="font-weight: bold"
                  @click="handleSubmit"
                >
                  Add Comment
                </a-button>
              </a-form-item>
            </div>
          </a-comment>
        </a-tab-pane>
      </a-tabs>
    </div>
  </div>
</template>
<script>
import Vue from "vue";
import Header from "../components/header.vue";
import moment from "moment";
import Card from "../components/course-card.vue";
import axios from "axios";
export default Vue.extend({
  inject: ["reload"],
  name: "watch-video",
  components: {
    Header,
    Card,
  },
  data() {
    return {
      startip: ["terrible", "bad", "normal", "good", "wonderful"],
      score: 0,
      chapterData: [],
      playerOptions: {
        playbackRates: [0.7, 1.0, 1.5, 2.0], // 播放速度
        autoplay: false, //如果true,浏览器准备好时开始回放。
        muted: false, // 默认情况下将会消除任何音频。
        loop: false, // 导致视频一结束就重新开始。
        preload: "auto", // 建议浏览器在<video>加载元素后是否应该开始下载视频数据。auto浏览器选择最佳行为,立即开始加载视频（如果浏览器支持）
        language: "zh-CN",
        aspectRatio: "100:35", // 将播放器置于流畅模式，并在计算播放器的动态大小时使用该值。值应该代表一个比例 - 用冒号分隔的两个数字（例如"16:9"或"4:3"）
        fluid: true, // 当true时，Video.js player将拥有流体大小。换句话说，它将按比例缩放以适应其容器。
        // src: this.videourl,
        sources: [
          {
            type: "video/webm" || "video/ogg" || "video/mp4", // 这里的种类支持很多种：基本视频格式、直播、流媒体等，具体可以参看git网址项目
            src: "", // url地址
          },
        ],
        // poster:
        //   "https://p1.music.126.net/5zs7IvmLv7KahY3BFzUmrg==/109951163635241613.jpg?param=600y500", // 你的封面地址
        notSupportedMessage:
          "This video is temporarily unavailable, please try again later", // 允许覆盖Video.js无法播放媒体源时显示的默认信息。
        controlBar: {
          timeDivider: true,
          durationDisplay: true,
          remainingTimeDisplay: false,
          fullscreenToggle: true, // 全屏按钮
        },
      },
      loading: false,
      courseicon: require("../assets/vd.png"),
      busy: false,
      commentdata: [],
      update: true,
      recommenddata: [],
      submitting: false,
      value: "",
      moment,
      coursedata: [],
      isfree: 2,
      free: 0,
      id: 0,
    };
  },

  created() {
    this.uid = localStorage.getItem("uid");
    this.postrul = localStorage.getItem("posturl");
    this.geturl();
  },
  mounted() {
    this.getParams();
    this.getChapter();
    this.getComment();
    this.getRecommend();
  },
  methods: {
    gobuy() {
      this.$router.push({
        path: "/course-detail",
        query: {
          cid: this.cid,
          name: this.coursename,
          price: this.price,
        },
      });
    },
    geturl() {
      this.playerOptions["sources"][0]["src"] = this.$route.query.url;
    },
    loadvideo(item) {
      this.url = item.url;
      this.playerOptions["sources"][0]["src"] = this.url;
      this.free = item.isfree;
      // console.log(this.videourl, item.url);
    },
    getParams() {
      this.cid = this.$route.query.cid;
      this.coursename = this.$route.query.coursename;
      this.price = this.$route.query.price;
    },
    getChapter() {
      this.loading = true;
      axios({
        method: "get",
        url: "http://localhost:9998/elec5620/main/chapter?id=" + this.cid,
      })
        .then((response) => {
          this.loading = false;
          if (response.data.success === true) {
            this.chapterData = response.data.data;
            for (let i in this.chapterData) {
              this.id = this.chapterData[i].id;
            }
            console.log("章节", this.chapterData);
          } else {
            this.$message.error("Chpaters loading failed.");
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
        url: "http://localhost:9998/elec5620/main/comment?id=" + this.cid,
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
    getRecommend() {
      axios({
        method: "get",
        url: "http://localhost:9998/elec5620/main/recommend?id=" + this.cid,
      })
        .then((response) => {
          this.loading = false;
          if (response.data.success === true) {
            this.recommenddata = response.data.data;
            console.log("推荐", this.recommenddata);
          } else {
            this.$message.error("Recommend Courses loading failed.");
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },
    handleSubmit() {
      if (!this.value) {
        return;
      }
      this.submitting = true;
      axios({
        method: "post",
        url: "http://localhost:9998/elec5620/main/makecomment",
        data: {
          cid: this.cid,
          comment: this.value,
          //改uid
          uid: this.uid,
        },
      })
        .then((response) => {
          if (response.data.success !== true) {
            this.$message.error("Comment failed.");
          }
        })
        .catch((error) => {
          console.log(error);
        });
      axios({
        method: "post",
        url: "http://localhost:9998/elec5620/main/score",
        data: {
          cid: this.cid,
          score: this.score,
        },
      })
        .then((response) => {
          if (response.data.success !== true) {
            this.$message.error("Score added failed.");
          }
        })
        .catch((error) => {
          console.log(error);
        });
      setTimeout(() => {
        this.submitting = false;
        this.getComment();
        this.value = "";
        this.score = 0;
        this.$message.success("Comment Added");
      }, 1000);
    },
    handleChange(e) {
      this.value = e.target.value;
    },
  },
});
</script>
<style lang="less">
.watchvideo {
  .videocontainer {
    margin: 0 auto;
    width: 100%;
    max-height: 200px;
    background: rgba(143, 189, 203, 0.6);
    .empty {
      position: absolute;
      top: 150px;
      width: 100%;
      height: 400px;
    }
  }
  .videobot {
    margin: 0 auto;
    position: absolute;
    width: 1100px;
    top: 650px;
    left: 82px;
    right: 82px;
    height: 400px;
    .freeic {
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
    .demo-infinite-container1 {
      border: 1px solid #e8e8e8;
      border-radius: 4px;
      overflow: auto;
      padding: 10px 30px;
      height: 400px;
    }
    .demo-infinite-container2 {
      border: 1px solid #e8e8e8;
      border-radius: 4px;
      overflow: auto;
      padding: 3px 10px;
      height: 300px;
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
    .recommendtitle {
      position: absolute;
      left: 20px;
      top: 45px;
    }
  }
}
</style>
