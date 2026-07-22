<template>
  <div id="app">
    <Layout ref="layout" />
  </div>
</template>

<script>
import util from "@/libs/util";
import Layout from "./views/Layout";

export default {
  name: "App",
  components: {
    Layout,
  },
  data: function() {
    return {
      github: "https://github.com/minhquan16jp/goindex-theme-acrou",
    };
  },
  watch: {
    "$i18n.locale": "i18nHandle",
  },
  created() {
    this.i18nHandle(this.$i18n.locale);
  },
  mounted() {
    this.checkVersion();
  },
  methods: {
    i18nHandle(val) {
      util.cookies.set("lang", val);
      document.querySelector("html").setAttribute("lang", val);
    },
    checkVersion() {
      let g2index_version = window.gdconfig.version;
      let app_version = process.env.VUE_APP_G2INDEX_VERSION;
      if (!g2index_version || app_version !== g2index_version) {
        this.$notify({
          title: this.$t("notify.title"),
          dangerouslyUseHTMLString: true,
          message: this.$t("checkVersion.tips").replace("${url}", this.github),
          duration: 0,
          type: "success",
        });
      }
    },
  },
};
</script>
<style>
/* ==========================================
   GLOBAL DARK MODE SYSTEM - MINKUAN THEME
   ========================================== */

/* 1. Nền tổng toàn trang web (Dark Cyberpunk) */
html, body, #app, .el-container, .main-body {
  background-color: #0b0f19 !important;
  color: #f8fafc !important;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif !important;
  color-scheme: dark !important; /* Báo cho trình duyệt biết đây là Dark Site */
}

/* 2. Sửa Menu Dropdown sổ xuống (Trực tiếp Element UI gốc) */
.el-dropdown-menu,
.el-popper,
.el-cascader__dropdown {
  background-color: #1e293b !important;
  border: 1px solid #334155 !important;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.5) !important;
  border-radius: 12px !important;
  padding: 6px !important;
}

.el-dropdown-menu__item {
  color: #cbd5e1 !important;
  border-radius: 8px !important;
  margin: 2px 0 !important;
}

.el-dropdown-menu__item:hover,
.el-dropdown-menu__item:focus {
  background-color: #0284c7 !important;
  color: #ffffff !important;
}

/* 3. Đường dẫn Breadcrumb */
.el-breadcrumb__item .el-breadcrumb__inner,
.el-breadcrumb__item .el-breadcrumb__inner a {
  color: #94a3b8 !important;
}

.el-breadcrumb__item:last-child .el-breadcrumb__inner {
  color: #38bdf8 !important;
  font-weight: 700;
}

/* 4. Fix màu hiệu ứng Loading */
.el-loading-mask {
  background-color: rgba(11, 15, 25, 0.85) !important;
  backdrop-filter: blur(8px) !important;
}

.el-loading-spinner .path {
  stroke: #38bdf8 !important;
}

/* 5. Thanh cuộn Scrollbar */
::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}
::-webkit-scrollbar-track {
  background: #0b0f19;
}
::-webkit-scrollbar-thumb {
  background: #334155;
  border-radius: 4px;
}
::-webkit-scrollbar-thumb:hover {
  background: #0284c7;
}
</style>
