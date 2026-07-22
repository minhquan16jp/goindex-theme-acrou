<template>
  <div class="main-container">
    <headmd
      :option="headmd"
      v-if="renderHeadMD && headmd.display"
      style="margin: 1rem 0;"
    ></headmd>

    <bread-crumb ref="breadcrumb"></bread-crumb>

    <!-- Progress Card (Hỗ trợ Dark Mode) -->
    <div class="progress-card" v-if="files.length > 0">
      <div class="progress-info">
        <div class="progress-text">
          <span class="progress-badge">TIẾN ĐỘ HỌC TẬP</span>
          <span class="progress-count">Đã hoàn thành <strong>{{ markedCount }}</strong> / {{ files.length }} bài</span>
        </div>
        <div class="progress-percentage-text">{{ progressPercentage }}%</div>
      </div>
      <div class="progress-track">
        <div class="progress-fill" :style="{ width: progressPercentage + '%' }"></div>
      </div>
    </div>

    <!-- Main List Card Container -->
    <div class="golist-card" v-loading="loading">
      <list-view
        :data="files"
        v-if="mode === 'list'"
        :icons="getIcon"
        :action="action"
        :copy="copy"
      />
      <grid-view
        class="g2-content"
        :data="files"
        v-if="mode !== 'list'"
        :getIcon="getIcon"
        :action="action"
        :thum="thum"
      />
      <infinite-loading
        v-show="!loading"
        ref="infinite"
        spinner="bubbles"
        @infinite="infiniteHandler"
      >
        <div slot="no-more"></div>
        <div slot="no-results"></div>
      </infinite-loading>
      <div
        v-show="files.length === 0"
        class="has-text-centered no-content"
      ></div>
    </div>

    <div
      class="is-divider custom-divider"
      :data-content="
        $t('list.total') + ' ' + files.length + ' ' + $t('list.item')
      "
    ></div>

    <readmemd
      :option="readmemd"
      v-if="renderReadMeMD && readmemd.display"
    ></readmemd>

    <viewer
      v-if="viewer && images && images.length > 0"
      :images="images"
      class="is-hidden"
      ref="viewer"
      :options="{ toolbar: true, url: 'data-source' }"
      @inited="inited"
    >
      <img
        v-for="image in images"
        :src="thum(image.thumbnailLink)"
        :data-source="image.path"
        :key="image.path"
        :alt="image.name"
        class="image"
      />
    </viewer>
  </div>
</template>

<script>
import {
  formatDate,
  formatFileSize,
  checkoutPath,
  checkView,
  checkExtends,
} from "@utils/AcrouUtil";
import { mapState, mapActions } from "vuex";
import BreadCrumb from "../common/BreadCrumb";
import ListView from "./components/list";
import GridView from "./components/grid";
import Markdown from "../common/Markdown";
import InfiniteLoading from "vue-infinite-loading";

export default {
  name: "GoList",
  components: {
    BreadCrumb,
    ListView,
    GridView,
    Headmd: Markdown,
    Readmemd: Markdown,
    InfiniteLoading,
  },
  data: function() {
    return {
      infiniteId: +new Date(),
      loading: true,
      page: {
        page_token: null,
        page_index: 0,
      },
      files: [],
      viewer: false,
      icon: {
        "application/vnd.google-apps.folder": "icon-folder",
        "video/mp4": "icon-mp4",
        "video/x-matroska": "icon-mkv",
        "video/x-msvideo": "icon-avi",
        "video/webm": "icon-webm",
        "video/x-flv": "icon-video",
        "application/x-mpegURL": "icon-video",
        "audio/mpegurl": "icon-video",
        "audio/mp3": "icon-audio",
        "audio/flac": "icon-audio",
        "audio/x-m4a": "icon-audio",
        "audio/wav": "icon-audio",
        "audio/ogg": "icon-audio",
        "text/plain": "icon-txt",
        "text/markdown": "icon-markdown",
        "text/x-ssa": "icon-ass",
        "text/html": "icon-html",
        "text/x-python-script": "icon-python",
        "text/x-java": "icon-java",
        "text/x-sh": "icon-sh",
        "application/x-subrip": "icon-srt",
        "application/zip": "icon-zip",
        "application/x-zip-compressed": "icon-zip",
        "application/rar": "icon-rar",
        "application/pdf": "icon-pdf",
        "application/json": "icon-json",
        "application/x-yaml": "icon-yml",
        "application/vnd.openxmlformats-officedocument.wordprocessingml.document":
          "icon-word",
        "application/vnd.android.package-archive": "icon-app",
        "application/x-msdownload": "icon-exe",
        "application/x-apple-diskimage": "icon-dmg",
        "application/vnd.google-apps.shortcut": "icon-link",
        "image/bmp": "icon-img",
        "image/jpeg": "icon-img",
        "image/png": "icon-img",
        "image/gif": "icon-img",
      },
      headmd: { display: false, file: {}, path: "" },
      readmemd: { display: false, file: {}, path: "" },
    };
  },
  computed: {
    ...mapState("acrou/view", ["mode"]),
    images() {
      return this.files.filter((file) => file.mimeType.startsWith("image/"));
    },
    renderHeadMD() {
      return window.themeOptions.render.head_md || false;
    },
    renderReadMeMD() {
      return window.themeOptions.render.readme_md || false;
    },
    markedCount() {
      try {
        let store = JSON.parse(localStorage.getItem('minkuan_manual_marked') || '{}');
        return this.files.filter(f => !!store[f.name]).length;
      } catch(e) {
        return 0;
      }
    },
    progressPercentage() {
      if (this.files.length === 0) return 0;
      return Math.round((this.markedCount / this.files.length) * 100);
    }
  },
  created() {
    this.render();
  },
  methods: {
    ...mapActions("acrou/aplayer", ["add"]),
    ...mapActions("acrou/db", ["set"]),
    infiniteHandler($state) {
      if (!this.page.page_token) {
        return;
      }
      this.page.page_index++;
      this.render($state);
    },
    render($state) {
      this.headmd = { display: false, file: {}, path: "" };
      this.readmemd = { display: false, file: {}, path: "" };
      var path = this.$route.path;
      var password = localStorage.getItem("password" + path);
      let q = this.$route.query.q;
      var p = {
        q: q ? decodeURIComponent(q) : "",
        password: password || null,
        ...this.page,
      };
      this.axios
        .post(path, p)
        .then((res) => {
          var body = res.data;
          if (body) {
            if (body.error && body.error.code == "401") {
              this.checkPassword(path);
              return;
            }
            var data = body.data;
            if (!data) return;
            this.page = {
              page_token: body.nextPageToken,
              page_index: body.curPageIndex,
            };
            if ($state) {
              this.files.push(...this.buildFiles(data.files));
            } else {
              this.files = this.buildFiles(data.files);
            }
            if (data.files) {
              this.renderMd(data.files, path);
            }
          }
          if (body.nextPageToken) {
            this.$refs.infinite.stateChanger.loaded();
          } else {
            this.$refs.infinite.stateChanger.complete();
          }
          this.loading = false;
        })
        .catch(() => {
          this.loading = false;
        });
    },
    buildFiles(files) {
      var path = this.$route.path;
      if (!files) return [];

      let cleanFiles = files.filter((item) => {
        return !['.DS_Store', 'HEAD.md', 'README.md'].includes(item.name);
      });

      return cleanFiles
        .map((item) => {
          var p = path + checkoutPath(item.name, item);
          let isFolder =
            item.mimeType === "application/vnd.google-apps.folder";
          let size = isFolder ? "-" : formatFileSize(item.size);
          return {
            path: p,
            ...item,
            modifiedTime: formatDate(item.modifiedTime),
            size: size,
            isFolder: isFolder,
          };
        })
        .sort((a, b) => {
          if (a.isFolder && b.isFolder) {
            return 0;
          }
          return a.isFolder ? -1 : 1;
        });
    },
    checkPassword(path) {
      var pass = prompt(this.$t("list.auth"), "");
      localStorage.setItem("password" + path, pass);
      if (pass != null && pass != "") {
        this.render(path);
      } else {
        this.$router.go(-1);
      }
    },
    copy(path) {
      let origin = window.location.origin;
      path = origin + encodeURI(path);
      this.$copyText(path);
    },
    thum(url) {
      return url ? `/${this.$route.params.id}:view?url=${url}` : "";
    },
    inited(viewer) {
      this.$viewer = viewer;
    },
    action(file, target, isSearch = true) {
      if (file.mimeType === "application/vnd.google-apps.shortcut") {
        this.$notify({
          title: "notify.title",
          message: "error.shortcut_not_down",
          type: "warning",
        });
        return;
      }

      let cmd = this.$route.params.cmd;
      if (cmd && cmd === "search" && isSearch) {
        this.goSearchResult(file, target);
        return;
      }

      if (file.mimeType.startsWith("image/") && target === "view") {
        this.viewer = true;
        this.$nextTick(() => {
          let index = this.images.findIndex((item) => item.path === file.path);
          this.$viewer.view(index);
        });
        return;
      }
      if (
        file.mimeType.startsWith("audio/") &&
        file.mimeType.indexOf("mpegurl") == -1 &&
        target === "view"
      ) {
        if (window.aplayer) {
          this.add({
            audio: {
              id: file.id,
              name: file.name,
              artist: "none",
              url: file.path,
            },
            play: true,
          });
        }
        return;
      }
      this.target(file, target);
    },
    target(file, target) {
      let path = file.path;
      if (target === "_blank") {
        window.open(path);
        return;
      }
      if (target === "copy") {
        this.copy(path);
        return;
      }
      if (target === "down" || (!checkExtends(path) && !file.isFolder)) {
        location.href = path.replace(/^\/(\d+:)\//, "/$1down/");
        return;
      }
      if (target === "view") {
        let checkViewPath = checkView(path);
        this.set({
          path: `page.${checkViewPath}`,
          value: file,
        });
        this.$router.push({
          path: checkViewPath,
        });
        return;
      }
      if (file.mimeType === "application/vnd.google-apps.folder") {
        this.$router.push({
          path: path,
        });
        return;
      }
    },
    renderMd(files, path) {
      var cmd = this.$route.params.cmd;
      if (cmd) {
        return;
      }
      files.forEach((item) => {
        if (item.name === "HEAD.md") {
          this.headmd = {
            display: true,
            file: item,
            path: path + item.name,
          };
        }
        if (item.name === "README.md") {
          this.readmemd = {
            display: true,
            file: item,
            path: path + item.name,
          };
        }
      });
    },
    goSearchResult(file, target) {
      this.loading = true;
      let id = this.$route.params.id;
      this.axios
        .post(`/${id}:id2path`, { id: file.id })
        .then((res) => {
          this.loading = false;
          let data = res.data;
          if (data) {
            file.path = `/${id}:${data}`;
            this.action(file, target, false);
          }
        })
        .catch((e) => {
          this.loading = false;
          console.log(e);
        });
    },
    getIcon(type) {
      return "#" + (this.icon[type] ? this.icon[type] : "icon-file");
    },
  },
};
</script>

<style scoped>
/* Biến màu Light Mode (Sáng) */
:root {
  --card-bg: #ffffff;
  --card-border: rgba(226, 232, 240, 0.8);
  --progress-bg: linear-gradient(135deg, #ffffff 0%, #f8fafc 100%);
  --progress-border: #e2e8f0;
  --progress-count-color: #475569;
  --progress-track: #e2e8f0;
}

/* Biến màu Dark Mode (Tối) */
@media (prefers-color-scheme: dark) {
  :root {
    --card-bg: #1e293b;
    --card-border: #334155;
    --progress-bg: #1e293b;
    --progress-border: #334155;
    --progress-count-color: #94a3b8;
    --progress-track: #334155;
  }
}

.main-container {
  max-width: 1140px;
  margin: 0 auto;
  padding: 0 12px;
}

/* Card Container */
.golist-card {
  background: var(--card-bg, #ffffff);
  border-radius: 16px;
  box-shadow: 0 10px 30px -5px rgba(0, 0, 0, 0.05);
  border: 1px solid var(--card-border, #e2e8f0);
  padding: 12px 20px;
  margin-top: 16px;
  transition: all 0.3s ease;
}

/* Progress Card */
.progress-card {
  background: var(--progress-bg, #ffffff);
  border: 1px solid var(--progress-border, #e2e8f0);
  border-radius: 14px;
  padding: 16px 20px;
  margin-top: 14px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.03);
}

.progress-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.progress-text {
  display: flex;
  align-items: center;
  gap: 12px;
}

.progress-badge {
  font-size: 10px;
  font-weight: 800;
  letter-spacing: 0.5px;
  background: #064e3b;
  color: #34d399;
  padding: 3px 8px;
  border-radius: 6px;
  border: 1px solid #059669;
}

.progress-count {
  font-size: 13px;
  color: var(--progress-count-color, #475569);
}

.progress-percentage-text {
  font-size: 16px;
  font-weight: 800;
  color: #10b981;
}

.progress-track {
  width: 100%;
  height: 8px;
  background-color: var(--progress-track, #e2e8f0);
  border-radius: 99px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #10b981 0%, #059669 100%);
  border-radius: 99px;
  transition: width 0.5s ease;
}

.custom-divider {
  margin: 24px 0;
  opacity: 0.6;
}
</style>