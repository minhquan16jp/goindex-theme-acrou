<template>
  <div class="video-container">
    <!-- Player Card: Khung phát Video chính phong cách Cinema -->
    <div class="video-card">
      <div v-if="options && options.api" class="video-content-iframe">
        <iframe
          width="100%"
          height="100%"
          :src="apiVideoUrl"
          frameborder="0"
          border="0"
          marginwidth="0"
          marginheight="0"
          scrolling="no"
          allowtransparency="true"
          allowfullscreen="true"
        ></iframe>
      </div>

      <div v-else class="video-player-wrapper">
        <vue-plyr ref="plyr" :options="options">
          <video controls crossorigin playsinline preload="metadata">
            <source :src="videoUrl" type="video/mp4" />
            <track
              kind="captions"
              label="Default"
              srclang="default"
              :src="subtitle"
              default
            />
          </video>
        </vue-plyr>
      </div>

      <!-- Control Bar & Quick Link Copy -->
      <div class="video-actions-bar">
        <div class="field-copy-group">
          <span class="link-label"><i class="fa fa-link"></i> Link Trực Tiếp:</span>
          <div class="input-with-btn">
            <input class="custom-input" type="text" readonly :value="videoUrl" @click="$event.target.select()" />
            <button class="btn-copy" @click="copy">
              <i class="fa fa-clone"></i> {{ $t("copy.text") }}
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- External Players Card: Danh sách mở bằng App ngoài -->
    <div class="external-players-card">
      <div class="card-title-group">
        <span class="title-badge">EXTERNAL PLAYERS</span>
        <h3 class="card-heading">Mở bằng Ứng Dụng Ngoại Vi</h3>
      </div>

      <div class="players-grid">
        <a
          v-for="(item, index) in players"
          :key="index"
          :href="item.scheme"
          class="player-item"
        >
          <div class="player-icon-wrapper">
            <img class="player-icon" :src="item.icon" :alt="item.name" />
          </div>
          <span class="player-name">{{ item.name }}</span>
        </a>
      </div>
    </div>
  </div>
</template>

<script>
import { decode64 } from "@utils/AcrouUtil";
import VuePlyr from "vue-plyr";

export default {
  components: {
    VuePlyr,
  },
  data: function() {
    return {
      apiVideoUrl: "",
      videoUrl: "",
      subtitle: "",
    };
  },
  mounted() {
    this.render();
    this.initMobileFullscreenFix();
  },
  beforeDestroy() {
    this.stopAllMedia();
  },
  beforeRouteLeave(to, from, next) {
    this.stopAllMedia();
    next();
  },
  methods: {
    // Kích hoạt chế độ phóng to chuẩn thiết bị trên điện thoại (đặc biệt là iOS Safari)
    initMobileFullscreenFix() {
      this.$nextTick(() => {
        const videoEl = this.$el.querySelector("video");
        if (videoEl) {
          // Bắt sự kiện khi người dùng bấm nút phóng to của Plyr trên điện thoại
          videoEl.addEventListener("webkitbeginfullscreen", () => {
            videoEl.style.objectFit = "contain";
          });
        }
      });
    },
    stopAllMedia() {
      try {
        if (this.$refs.plyr && this.$refs.plyr.player) {
          this.$refs.plyr.player.stop();
        }
      } catch (e) {
        console.error("Error stopping Plyr player:", e);
      }
      try {
        const mediaElements = document.querySelectorAll("video, audio");
        mediaElements.forEach((el) => {
          el.pause();
          el.removeAttribute("src");
          el.load();
        });
      } catch (e) {
        console.error("Error pausing media DOM elements:", e);
      }
    },
    render() {
      let path = encodeURI(this.url);
      let index = path.lastIndexOf(".");
      this.suffix = path.substring(index + 1, path.length);
      this.loadSub(path, index);
      this.videoUrl = window.location.origin + path;
      this.apiVideoUrl = this.options.api + this.videoUrl;
      if (!this.options.api) {
        let options = {
          src: this.videoUrl,
          autoplay: this.options.autoplay,
          media: this.player ? this.player.media : null,
        };
        if (this.suffix === "m3u8") {
          this.loadHls(options);
        } else if (this.suffix === "flv") {
          this.loadFlv(options);
        }
      }
    },
    loadSub(path, index) {
      this.subtitle = path.substring(0, index) + ".vtt";
    },
    loadHls(options) {
      import("@/plugin/vplayer/hls").then((res) => {
        var Hls = res.default;
        Hls({
          ...options,
          callback: (hls) => {
            if (this.player) {
              this.player.on("languagechange", () => {
                setTimeout(
                  () => (hls.subtitleTrack = this.player.currentTrack),
                  50
                );
              });
            }
          },
        });
      });
    },
    loadFlv(options) {
      import("@/plugin/vplayer/flv").then((res) => {
        var Flv = res.default;
        Flv(options);
      });
    },
    copy() {
      this.$copyText(this.videoUrl);
      if (this.$notify) {
        this.$notify({
          title: "Thành công",
          message: "Đã sao chép đường dẫn video trực tiếp!",
          type: "success",
          duration: 2000,
        });
      }
    },
  },
  computed: {
    options() {
      var options = window.themeOptions ? window.themeOptions.video : {};
      return {
        autoplay: false,
        invertTime: false,
        seekTime: 10,
        settings: ["quality", "speed", "loop"],
        ratio: "16:9",
        controls: [
          "play-large",
          "restart",
          "play",
          "progress",
          "current-time",
          "duration",
          "mute",
          "volume",
          "captions",
          "settings",
          "pip",
          "airplay",
          "download",
          "fullscreen",
        ],
        ...options,
        captions: { active: true, language: "default", ...(options ? options.captions : {}) },
      };
    },
    player() {
      return this.$refs.plyr ? this.$refs.plyr.player : null;
    },
    url() {
      if (this.$route.params.path) {
        return decode64(this.$route.params.path);
      }
      return "";
    },
    players() {
      return [
        {
          name: "IINA",
          icon: this.$cdnpath("images/player/iina.png"),
          scheme: "iina://weblink?url=" + this.videoUrl,
        },
        {
          name: "PotPlayer",
          icon: this.$cdnpath("images/player/potplayer.png"),
          scheme: "potplayer://" + this.videoUrl,
        },
        {
          name: "VLC",
          icon: this.$cdnpath("images/player/vlc.png"),
          scheme: "vlc://" + this.videoUrl,
        },
        {
          name: "Thunder",
          icon: this.$cdnpath("images/player/thunder.png"),
          scheme: "thunder://" + this.getThunder,
        },
        {
          name: "Aria2",
          icon: this.$cdnpath("images/player/aria2.png"),
          scheme: 'javascript:alert("Chức năng đang phát triển")',
        },
        {
          name: "nPlayer",
          icon: this.$cdnpath("images/player/nplayer.png"),
          scheme: "nplayer-" + this.videoUrl,
        },
        {
          name: "MXPlayer (Free)",
          icon: this.$cdnpath("images/player/mxplayer.png"),
          scheme:
            "intent:" +
            this.videoUrl +
            "#Intent;package=com.mxtech.videoplayer.ad;S.title=" +
            this.title +
            ";end",
        },
        {
          name: "MXPlayer (Pro)",
          icon: this.$cdnpath("images/player/mxplayer.png"),
          scheme:
            "intent:" +
            this.videoUrl +
            "#Intent;package=com.mxtech.videoplayer.pro;S.title=" +
            this.title +
            ";end",
        },
      ];
    },
    getThunder() {
      return Buffer.from("AA" + this.videoUrl + "ZZ").toString("base64");
    },
  },
};
</script>

<style scoped>
.video-container {
  max-width: 1040px;
  margin: 16px auto 40px auto;
  padding: 0 12px;
}

/* 1. Cinema Player Card Container */
.video-card {
  background: #ffffff;
  border-radius: 16px;
  border: 1px solid #e2e8f0;
  box-shadow: 0 12px 32px -8px rgba(0, 0, 0, 0.08);
  overflow: hidden;
  margin-bottom: 24px;
  transition: transform 0.25s ease, box-shadow 0.25s ease;
}

.video-card:hover {
  box-shadow: 0 16px 40px -10px rgba(0, 0, 0, 0.12);
}

.video-player-wrapper,
.video-content-iframe {
  position: relative;
  width: 100%;
  background: #000000;
  border-bottom: 1px solid #e2e8f0;
  overflow: hidden;
}

.video-content-iframe iframe {
  aspect-ratio: 16 / 9;
  min-height: 480px;
}

/* Tùy chỉnh màu nhấn Plyr Controls sang tông Cyan tươi mát */
>>> .plyr {
  border-radius: 0 !important;
  --plyr-color-main: #0284c7;
  --plyr-video-background: #000000;
}

>>> .plyr--video {
  max-height: 75vh;
}

/* 📱 ÉP VIDEO BẮT BUỘC TRÀN KÍN MÀN HÌNH ĐIỆN THOẠI KHI PHÓNG TO / XOAY NGANG */
>>> .plyr--fullscreen-active video,
>>> .plyr--fullscreen-fallback video,
>>> video:fullscreen {
  object-fit: fill !important;
  width: 100vw !important;
  height: 100vh !important;
}

/* Thanh công cụ Copy Link */
.video-actions-bar {
  padding: 16px 20px;
  background: #ffffff;
}

.field-copy-group {
  display: flex;
  align-items: center;
  gap: 12px;
}

.link-label {
  font-size: 13px;
  font-weight: 700;
  color: #475569;
  white-space: nowrap;
  display: flex;
  align-items: center;
  gap: 6px;
}

.input-with-btn {
  display: flex;
  align-items: center;
  width: 100%;
  background: #f8fafc;
  border: 1px solid #cbd5e1;
  border-radius: 10px;
  padding: 3px;
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
}

.input-with-btn:focus-within {
  border-color: #0284c7;
  box-shadow: 0 0 0 3px rgba(2, 132, 199, 0.15);
}

.custom-input {
  width: 100%;
  border: none;
  background: transparent;
  padding: 6px 12px;
  font-size: 13px;
  color: #1e293b;
  outline: none;
  font-family: -apple-system, BlinkMacSystemFont, "SF Mono", Roboto, monospace;
}

.btn-copy {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  background: #0284c7;
  color: #ffffff;
  border: none;
  padding: 8px 18px;
  font-size: 12.5px;
  font-weight: 600;
  border-radius: 8px;
  cursor: pointer;
  white-space: nowrap;
  transition: all 0.2s ease;
  box-shadow: 0 2px 6px rgba(2, 132, 199, 0.25);
}

.btn-copy:hover {
  background: #0369a1;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(2, 132, 199, 0.35);
}

.btn-copy:active {
  transform: translateY(0);
}

/* 2. External Players App Grid */
.external-players-card {
  background: #ffffff;
  border-radius: 16px;
  border: 1px solid #e2e8f0;
  padding: 20px 24px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.03);
}

.card-title-group {
  margin-bottom: 18px;
}

.title-badge {
  font-size: 10px;
  font-weight: 800;
  letter-spacing: 0.8px;
  background: #f0f9ff;
  color: #0284c7;
  padding: 3px 8px;
  border-radius: 6px;
  border: 1px solid #bae6fd;
}

.card-heading {
  font-size: 15px;
  font-weight: 700;
  color: #0f172a;
  margin-top: 6px;
}

.players-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(105px, 1fr));
  gap: 14px;
}

.player-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 12px 8px;
  background: #f8fafc;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  text-decoration: none;
  transition: all 0.2s cubic-bezier(0.16, 1, 0.3, 1);
}

.player-item:hover {
  background: #ffffff;
  border-color: #0284c7;
  transform: translateY(-2px);
  box-shadow: 0 6px 18px rgba(2, 132, 199, 0.15);
}

.player-icon-wrapper {
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 8px;
}

.player-icon {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.player-name {
  font-size: 12px;
  font-weight: 600;
  color: #475569;
  text-align: center;
}

.player-item:hover .player-name {
  color: #0284c7;
}

/* 📱 TỐI ƯU RIÊNG CHO MÀN HÌNH ĐIỆN THOẠI (< 640px) */
@media (max-width: 640px) {
  .video-container {
    padding: 0 6px;
    margin-top: 8px;
  }
  .video-card {
    border-radius: 12px;
  }
  .field-copy-group {
    flex-direction: column;
    align-items: flex-start;
  }
  .players-grid {
    grid-template-columns: repeat(4, 1fr);
    gap: 8px;
  }
  .player-item {
    padding: 8px 4px;
  }
  .player-icon-wrapper {
    width: 32px;
    height: 32px;
  }
  .player-name {
    font-size: 10px;
  }
  /* Bắt video tự động vừa khít chiều rộng màn hình đứng */
  >>> .plyr--video {
    max-height: 50vh;
  }
  >>> .plyr video {
    object-fit: contain;
  }
}
</style>