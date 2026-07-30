<template>
  <div class="video-container">
    <!-- 1. KHUNG PHÁT VIDEO CHÍNH (CINEMA CARD) -->
    <div class="video-card">
      <!-- Top Bar: Tên file đang phát -->
      <div class="video-card-header">
        <div class="playing-title-group">
          <span class="status-dot"></span>
          <h2 class="playing-title" :title="currentFileName">
            {{ currentFileName }}
          </h2>
        </div>
        <div class="header-actions">
          <button
            class="nav-btn"
            :disabled="!prevFile"
            title="Video trước"
            @click="playFile(prevFile)"
          >
            <i class="fa fa-step-backward"></i>
          </button>
          <button
            class="nav-btn"
            :disabled="!nextFile"
            title="Video tiếp theo"
            @click="playFile(nextFile)"
          >
            <i class="fa fa-step-forward"></i>
          </button>
        </div>
      </div>

      <!-- Player Frame -->
      <div v-if="isApiMode" class="video-content-iframe">
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
        <vue-plyr ref="plyr" :options="plyrOptions">
          <video
            controls
            crossorigin="anonymous"
            playsinline
            webkit-playsinline
            preload="metadata"
            :key="videoUrl"
          >
            <source :src="videoUrl" type="video/mp4" />
            <track
              v-if="subtitle"
              kind="captions"
              label="Default"
              srclang="default"
              :src="subtitle"
              default
            />
          </video>
        </vue-plyr>
      </div>

      <!-- Quick Actions Bar: Copy Link & Info -->
      <div class="video-actions-bar">
        <div class="field-copy-group">
          <span class="link-label"><i class="fa fa-link"></i> Đường dẫn trực tiếp:</span>
          <div class="input-with-btn">
            <input
              class="custom-input"
              type="text"
              readonly
              :value="videoUrl"
              @click="$event.target.select()"
            />
            <button class="btn-copy" type="button" @click="copy">
              <i class="fa fa-clone"></i> {{ copyBtnText }}
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- 2. DANH SÁCH TỆP TRONG THƯ MỤC (FILE EXPLORER / PLAYLIST) -->
    <div class="file-explorer-card">
      <div class="explorer-header">
        <div class="explorer-title-group">
          <div class="icon-badge"><i class="fa fa-folder-open"></i></div>
          <div>
            <h3 class="explorer-heading">Danh Sách Tệp Trong Thư Mục</h3>
            <p class="explorer-sub">
              {{ folderFiles.length }} tệp khả dụng • Nhấp để đổi video phát
            </p>
          </div>
        </div>

        <!-- Ô Tìm Kiếm Tệp -->
        <div class="search-box">
          <i class="fa fa-search search-icon"></i>
          <input
            v-model="searchQuery"
            type="text"
            placeholder="Lọc tệp..."
            class="search-input"
          />
          <button v-if="searchQuery" class="clear-btn" @click="searchQuery = ''">
            <i class="fa fa-times"></i>
          </button>
        </div>
      </div>

      <!-- Danh sách tập tin -->
      <div class="file-list-wrapper custom-scrollbar">
        <div v-if="filteredFiles.length === 0" class="empty-state">
          <i class="fa fa-film empty-icon"></i>
          <p>Không tìm thấy tệp video phù hợp</p>
        </div>

        <div
          v-for="(file, index) in filteredFiles"
          :key="file.path || file.name || index"
          class="file-item"
          :class="{ active: isCurrentFile(file) }"
          @click="playFile(file)"
        >
          <div class="file-info-left">
            <div class="file-type-icon">
              <i v-if="isCurrentFile(file)" class="fa fa-play-circle text-active"></i>
              <i v-else-if="isVideoFile(file.name)" class="fa fa-file-video-o"></i>
              <i v-else class="fa fa-file-o"></i>
            </div>
            <div class="file-details">
              <span class="file-name" :title="file.name">{{ file.name }}</span>
              <span v-if="file.size" class="file-size">{{ file.size }}</span>
            </div>
          </div>

          <div class="file-status-right">
            <span v-if="isCurrentFile(file)" class="badge-playing">
              <i class="fa fa-volume-up"></i> Đang phát
            </span>
            <span v-else class="btn-play-hover">
              Phát <i class="fa fa-chevron-right"></i>
            </span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { decode64 } from "@utils/AcrouUtil";
import VuePlyr from "vue-plyr";

export default {
  name: "VideoPlayer",
  components: {
    VuePlyr,
  },
  data() {
    return {
      apiVideoUrl: "",
      videoUrl: "",
      subtitle: "",
      suffix: "",
      searchQuery: "",
      folderFiles: [],
      // Đưa config plyr vào data để tránh lỗi reactivity crash render
      plyrOptions: {
        autoplay: false,
        invertTime: false,
        seekTime: 10,
        settings: ["quality", "speed", "loop"],
        ratio: "16:9",
        fullscreen: {
          enabled: true,
          fallback: false,
          iosNative: true,
        },
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
          "fullscreen",
        ],
        captions: {
          active: true,
          language: "default",
        },
      },
    };
  },
  computed: {
    url() {
      try {
        if (this.$route && this.$route.params && this.$route.params.path) {
          return decode64(this.$route.params.path);
        }
      } catch (e) {
        console.error("Lỗi giải mã URL path:", e);
      }
      return "";
    },
    currentFileName() {
      if (!this.url) return "Đang tải video...";
      const parts = this.url.split("/");
      return decodeURIComponent(parts[parts.length - 1] || "Video");
    },
    copyBtnText() {
      try {
        return this.$t ? this.$t("copy.text") : "Sao chép";
      } catch (e) {
        return "Sao chép";
      }
    },
    isApiMode() {
      const globalOpts =
        typeof window !== "undefined" && window.themeOptions
          ? window.themeOptions.video
          : null;
      return !!(globalOpts && globalOpts.api);
    },
    player() {
      return this.$refs.plyr ? this.$refs.plyr.player : null;
    },
    filteredFiles() {
      if (!this.searchQuery) return this.folderFiles;
      const query = this.searchQuery.toLowerCase();
      return this.folderFiles.filter((f) =>
        (f.name || "").toLowerCase().includes(query)
      );
    },
    currentIndex() {
      return this.folderFiles.findIndex((f) => this.isCurrentFile(f));
    },
    prevFile() {
      if (this.currentIndex > 0) {
        return this.folderFiles[this.currentIndex - 1];
      }
      return null;
    },
    nextFile() {
      if (
        this.currentIndex !== -1 &&
        this.currentIndex < this.folderFiles.length - 1
      ) {
        return this.folderFiles[this.currentIndex + 1];
      }
      return null;
    },
  },
  watch: {
    "$route.params.path"() {
      this.render();
    },
  },
  created() {
    // Merge themeOptions an toàn trước khi mount component
    if (typeof window !== "undefined" && window.themeOptions && window.themeOptions.video) {
      this.plyrOptions = {
        ...this.plyrOptions,
        ...window.themeOptions.video,
      };
    }
  },
  mounted() {
    this.render();
    this.loadFolderFiles();
  },
  beforeDestroy() {
    this.stopAllMedia();
  },
  beforeRouteLeave(to, from, next) {
    this.stopAllMedia();
    next();
  },
  methods: {
    stopAllMedia() {
      try {
        if (this.player && typeof this.player.stop === "function") {
          this.player.stop();
        }
      } catch (e) {
        console.error("Lỗi khi dừng Plyr player:", e);
      }

      try {
        const mediaElements = document.querySelectorAll("video, audio");
        mediaElements.forEach((el) => {
          el.pause();
          el.removeAttribute("src");
          el.load();
        });
      } catch (e) {
        console.error("Lỗi khi pause media DOM elements:", e);
      }
    },
    render() {
      const path = encodeURI(this.url || "");
      const index = path.lastIndexOf(".");
      this.suffix = index !== -1 ? path.substring(index + 1) : "";
      this.loadSub(path, index);

      const origin = typeof window !== "undefined" ? window.location.origin : "";
      this.videoUrl = origin + path;

      const globalOpts =
        typeof window !== "undefined" && window.themeOptions
          ? window.themeOptions.video
          : null;

      if (globalOpts && globalOpts.api) {
        this.apiVideoUrl = globalOpts.api + this.videoUrl;
      } else {
        const options = {
          src: this.videoUrl,
          autoplay: this.plyrOptions.autoplay,
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
      if (index !== -1) {
        this.subtitle = path.substring(0, index) + ".vtt";
      }
    },
    loadHls(options) {
      import("@/plugin/vplayer/hls")
        .then((res) => {
          const Hls = res.default;
          if (typeof Hls === "function") {
            Hls({
              ...options,
              callback: (hls) => {
                if (this.player) {
                  this.player.on("languagechange", () => {
                    setTimeout(() => {
                      if (hls && this.player) {
                        hls.subtitleTrack = this.player.currentTrack;
                      }
                    }, 50);
                  });
                }
              },
            });
          }
        })
        .catch((e) => console.error("Lỗi load Hls plugin:", e));
    },
    loadFlv(options) {
      import("@/plugin/vplayer/flv")
        .then((res) => {
          const Flv = res.default;
          if (typeof Flv === "function") {
            Flv(options);
          }
        })
        .catch((e) => console.error("Lỗi load Flv plugin:", e));
    },
    loadFolderFiles() {
      if (typeof window !== "undefined" && window.driveFiles) {
        this.folderFiles = window.driveFiles;
        return;
      }

      const currentPath = this.url || "";
      if (this.currentFileName) {
        this.folderFiles = [
          {
            name: this.currentFileName,
            path: currentPath,
          },
        ];
      }
    },
    isCurrentFile(file) {
      if (!file || !file.name) return false;
      return file.name === this.currentFileName || file.path === this.url;
    },
    isVideoFile(filename) {
      if (!filename) return false;
      const ext = filename.split(".").pop().toLowerCase();
      return ["mp4", "mkv", "webm", "m3u8", "flv", "avi", "mov"].includes(ext);
    },
    playFile(file) {
      if (!file || this.isCurrentFile(file)) return;

      if (file.path) {
        const encoded =
          typeof window.btoa !== "undefined"
            ? window.btoa(file.path)
            : file.path;

        if (this.$router) {
          this.$router.push({ params: { path: encoded } }).catch(() => {});
        } else {
          window.location.hash = `#/${encoded}`;
        }
      } else if (file.url) {
        this.videoUrl = window.location.origin + encodeURI(file.url);
      }
    },
    copy() {
      if (!this.videoUrl) return;

      if (this.$copyText) {
        this.$copyText(this.videoUrl);
      } else if (navigator.clipboard) {
        navigator.clipboard.writeText(this.videoUrl);
      }

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
};
</script>

<style scoped>
/* Nhập CSS trực tiếp hoặc đảm bảo Vue-Plyr CSS không gây xung đột */
@import "~vue-plyr/dist/vue-plyr.css";

.video-container {
  max-width: 1040px;
  margin: 20px auto 50px auto;
  padding: 0 16px;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
}

/* 1. CINEMA PLAYER CARD */
.video-card {
  background: #ffffff;
  border-radius: 20px;
  border: 1px solid #e2e8f0;
  box-shadow: 0 20px 40px -15px rgba(0, 0, 0, 0.07);
  overflow: hidden;
  margin-bottom: 24px;
}

.video-card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 14px 20px;
  background: #f8fafc;
  border-bottom: 1px solid #f1f5f9;
}

.playing-title-group {
  display: flex;
  align-items: center;
  gap: 10px;
  overflow: hidden;
}

.status-dot {
  width: 9px;
  height: 9px;
  background-color: #10b981;
  border-radius: 50%;
  box-shadow: 0 0 0 3px rgba(16, 185, 129, 0.2);
  flex-shrink: 0;
}

.playing-title {
  font-size: 14px;
  font-weight: 600;
  color: #1e293b;
  margin: 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.header-actions {
  display: flex;
  align-items: center;
  gap: 6px;
}

.nav-btn {
  background: #ffffff;
  border: 1px solid #cbd5e1;
  color: #475569;
  width: 32px;
  height: 32px;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s ease;
  font-size: 12px;
}

.nav-btn:hover:not(:disabled) {
  background: #0284c7;
  color: #ffffff;
  border-color: #0284c7;
}

.nav-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

/* FRAME KHUNG PHÁT 16:9 CHUẨN */
.video-player-wrapper,
.video-content-iframe {
  position: relative;
  width: 100%;
  aspect-ratio: 16 / 9;
  background: #000000;
  overflow: hidden;
}

.video-content-iframe iframe {
  width: 100%;
  height: 100%;
}

:deep(.plyr) {
  width: 100% !important;
  height: 100% !important;
  border-radius: 0 !important;
  --plyr-color-main: #0284c7;
  --plyr-video-background: #000000;
}

:deep(.plyr--video) {
  height: 100% !important;
  max-height: none !important;
}

:deep(.plyr__video-wrapper) {
  height: 100% !important;
}

:deep(.plyr video) {
  width: 100% !important;
  height: 100% !important;
  object-fit: contain !important;
}

/* THANH ACTION COPY LINK */
.video-actions-bar {
  padding: 14px 20px;
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
  transition: border-color 0.2s ease;
}

.input-with-btn:focus-within {
  border-color: #0284c7;
}

.custom-input {
  width: 100%;
  border: none;
  background: transparent;
  padding: 6px 12px;
  font-size: 13px;
  color: #1e293b;
  outline: none;
  font-family: monospace;
}

.btn-copy {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  background: #0284c7;
  color: #ffffff;
  border: none;
  padding: 8px 16px;
  font-size: 12.5px;
  font-weight: 600;
  border-radius: 8px;
  cursor: pointer;
  white-space: nowrap;
  transition: background 0.2s ease;
}

.btn-copy:hover {
  background: #0369a1;
}

/* 2. FILE EXPLORER CARD */
.file-explorer-card {
  background: #ffffff;
  border-radius: 20px;
  border: 1px solid #e2e8f0;
  padding: 20px;
  box-shadow: 0 10px 30px -10px rgba(0, 0, 0, 0.04);
}

.explorer-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 16px;
  gap: 16px;
}

.explorer-title-group {
  display: flex;
  align-items: center;
  gap: 12px;
}

.icon-badge {
  width: 42px;
  height: 42px;
  background: #e0f2fe;
  color: #0284c7;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 18px;
}

.explorer-heading {
  font-size: 16px;
  font-weight: 700;
  color: #0f172a;
  margin: 0;
}

.explorer-sub {
  font-size: 12px;
  color: #64748b;
  margin: 2px 0 0 0;
}

/* SEARCH BOX */
.search-box {
  position: relative;
  display: flex;
  align-items: center;
  width: 220px;
}

.search-icon {
  position: absolute;
  left: 12px;
  color: #94a3b8;
  font-size: 13px;
}

.search-input {
  width: 100%;
  padding: 8px 30px 8px 34px;
  background: #f8fafc;
  border: 1px solid #cbd5e1;
  border-radius: 10px;
  font-size: 13px;
  color: #1e293b;
  outline: none;
  transition: all 0.2s ease;
}

.search-input:focus {
  border-color: #0284c7;
  background: #ffffff;
  box-shadow: 0 0 0 3px rgba(2, 132, 199, 0.1);
}

.clear-btn {
  position: absolute;
  right: 10px;
  border: none;
  background: transparent;
  color: #94a3b8;
  cursor: pointer;
}

/* DANH SÁCH FILE ITEM */
.file-list-wrapper {
  max-height: 380px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 6px;
  padding-right: 4px;
}

.file-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 14px;
  background: #f8fafc;
  border: 1px solid transparent;
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.file-item:hover {
  background: #f0f9ff;
  border-color: #bae6fd;
}

.file-item.active {
  background: #f0f9ff;
  border-color: #0284c7;
  box-shadow: 0 2px 8px rgba(2, 132, 199, 0.08);
}

.file-info-left {
  display: flex;
  align-items: center;
  gap: 12px;
  overflow: hidden;
}

.file-type-icon {
  font-size: 18px;
  color: #64748b;
  width: 24px;
  text-align: center;
  flex-shrink: 0;
}

.text-active {
  color: #0284c7;
}

.file-details {
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.file-name {
  font-size: 13.5px;
  font-weight: 500;
  color: #334155;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.file-item.active .file-name {
  font-weight: 700;
  color: #0284c7;
}

.file-size {
  font-size: 11px;
  color: #94a3b8;
}

.file-status-right {
  flex-shrink: 0;
  margin-left: 12px;
}

.badge-playing {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  font-size: 11px;
  font-weight: 700;
  color: #0284c7;
  background: #e0f2fe;
  padding: 4px 10px;
  border-radius: 20px;
}

.btn-play-hover {
  font-size: 12px;
  font-weight: 600;
  color: #64748b;
  opacity: 0;
  transition: opacity 0.2s ease;
}

.file-item:hover .btn-play-hover {
  opacity: 1;
  color: #0284c7;
}

.empty-state {
  padding: 40px 0;
  text-align: center;
  color: #94a3b8;
}

.empty-icon {
  font-size: 32px;
  margin-bottom: 8px;
}

/* Custom Scrollbar */
.custom-scrollbar::-webkit-scrollbar {
  width: 5px;
}
.custom-scrollbar::-webkit-scrollbar-track {
  background: #f1f5f9;
  border-radius: 10px;
}
.custom-scrollbar::-webkit-scrollbar-thumb {
  background: #cbd5e1;
  border-radius: 10px;
}

/* RESPONSIVE MOBILE (< 640px) */
@media (max-width: 640px) {
  .video-container {
    padding: 0 8px;
    margin-top: 10px;
  }
  .explorer-header {
    flex-direction: column;
    align-items: stretch;
  }
  .search-box {
    width: 100%;
  }
  .field-copy-group {
    flex-direction: column;
    align-items: flex-start;
  }
}
</style>