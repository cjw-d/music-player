<template>
  <div class="result">
    <div class="head">
      {{ $route.query.p }}
      <span class="sub-title">共找到{{ count }}个结果</span>
    </div>
    <el-tabs v-model="activeName" @tab-click="handleClick">
      <el-tab-pane label="歌曲" name="songs">
        <div class="res-songs-list">
          <thead>
            <th></th>
            <th>音乐标题</th>
            <th>歌手</th>
            <th>专辑</th>
            <th>时长</th>
          </thead>
          <tbody>
            <tr
              v-for="(item, index) in songList"
              :key="index"
              @dblclick="playMusic(item.id)"
            >
              <td>{{ index + 1 }}</td>
              <td>
                {{ item.name }}
                <i class="el-icon-video-play" @click="playMusic(item.id)"></i>
              </td>
              <td>{{ item.artists[0].name }}</td>
              <td>{{ item.album.name }}</td>
              <td>{{ item.duration }}</td>
            </tr>
          </tbody>
        </div>
      </el-tab-pane>
      <el-tab-pane label="歌单" name="lists">
        <div class="items">
          <div class="item" v-for="(item, index) in playList" :key="index">
            <div class="img-wrap" @click="toPlayList(item.id)">
              <img :src="item.coverImgUrl" alt="" />
            </div>
            <p class="name">{{ item.name }}</p>
          </div>
        </div>
      </el-tab-pane>
      <el-tab-pane label="MV" name="mv">
        <div class="items">
          <div class="item" v-for="(item, index) in mv" :key="index">
            <div class="img-wrap" @click="toMV(item.id)">
              <img :src="item.cover" alt="" />
            </div>
            <p class="name">{{ item.name }}</p>
          </div>
        </div>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import axios from "axios";
export default {
  data() {
    return {
      activeName: "songs",
      songList: [],
      playList: [],
      count: 0,
      mv: [],
      page: 1,
    };
  },
  watch: {
    activeName() {
      let type = 1;
      let limit = 10;
      switch (this.activeName) {
        case "songs":
          type = 1;
          break;
        case "lists":
          type = 1000;
          break;
        case "mv":
          type = 1004;
          break;
        default:
          break;
      }
      axios({
        url: "https://autumnfish.cn/search",
        method: "get",
        params: {
          keywords: this.$route.query.p,
          limit,
          type,
        },
      }).then((res) => {
        if (res.status !== 200) {
          this.$message.error("获取信息失败");
        } else {
          if (type == 1) {
            this.songList = res.data.result.songs;
            for (let i = 0; i < this.songList.length; i++) {
              let duration = this.songList[i].duration;
              let min = parseInt(duration / 1000 / 60);
              let sec = parseInt((duration / 1000) % 60);
              if (min < 10) {
                min = "0" + min;
              }
              if (sec < 10) {
                sec = "0" + sec;
              }
              this.songList[i].duration = min + ":" + sec;
            }
            this.count = res.data.result.songCount;
          } else if (type == 1000) {
            this.playList = res.data.result.playlists;
            this.count = res.data.result.playlistCount;
          } else {
            this.mv = res.data.result.mvs;
            this.count = res.data.result.mvCount;
          }
        }
      });
    },
  },
  methods: {
    handleClick() {},
    getLists() {
      axios({
        url: "https://autumnfish.cn/search",
        method: "get",
        params: {
          keywords: this.$route.query.p,
          limit: 10,
          type: 1,
        },
      }).then((res) => {
        if (res.status !== 200) {
          this.$message.error("获取信息失败");
        } else {
          this.songList = res.data.result.songs;
          for (let i = 0; i < this.songList.length; i++) {
            let duration = this.songList[i].duration;
            let min = parseInt(duration / 1000 / 60);
            let sec = parseInt((duration / 1000) % 60);
            if (min < 10) {
              min = "0" + min;
            }
            if (sec < 10) {
              sec = "0" + sec;
            }
            this.songList[i].duration = min + ":" + sec;
          }
          this.count = res.data.result.songCount;
        }
      });
    },
    playMusic(id) {
      axios({
        url: "https://autumnfish.cn/song/url",
        method: "get",
        params: {
          id,
        },
      }).then((res) => {
        if (res.status !== 200) {
          this.$message.error("获取信息失败");
        } else {
          this.$parent.musicUrl = res.data.data[0].url;
        }
      });
    },
    toPlayList(id) {
      this.$router.push(`/playlists?q=${id}`);
    },
    toMV(id) {
      this.$router.push(`/mv?q=${id}`);
    },
  },
  created() {
    this.getLists();
  },
};
</script>

<style>
.head {
  font-size: 30px;
  font-weight: 800;
  margin: 10px;
}
.res-songs-list tbody tr td {
  padding: 50px;
  text-align: center;
  border-bottom: 1px solid #979797;
}
.res-songs-list tbody tr {
  border-bottom: 1px solid black;
}
.res-songs-list thead th {
  font-size: 14px;
  font-weight: 200;
}
.el-icon-video-play {
  color: brown;
}
.el-icon-video-play:hover {
  cursor: pointer;
}
.sub-title {
  font-size: 14px;
  color: #979797;
}
</style>