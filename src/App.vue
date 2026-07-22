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

/* 2. SỬA TRIỆT ĐỂ CÁC BẢNG MENU DROPDOWN & CASCADER (Đè màu trắng) */
.el-cascader__dropdown,
.el-cascader-panel,
.el-cascader-menu,
.el-cascader-menu__list,
.el-dropdown-menu,
.el-popper {
  background-color: #1e293b !important; /* Xám Sapphire tối */
  border: 1px solid #334155 !important;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.6) !important;
  border-radius: 12px !important;
}

/* Từng dòng mục nhỏ bên trong Menu Cascader */
.el-cascader-node,
.el-dropdown-menu__item {
  background-color: transparent !important;
  color: #f8fafc !important; /* Màu chữ trắng sáng rõ nét */
  border-bottom: 1px solid rgba(255, 255, 255, 0.05) !important;
  padding: 10px 16px !important;
  transition: all 0.15s ease !important;
}

/* Hiệu ứng rê chuột vào mục menu */
.el-cascader-node:hover,
.el-cascader-node:focus,
.el-cascader-node.is-selectable:hover,
.el-dropdown-menu__item:hover {
  background-color: #0284c7 !important; /* Đổi sang màu Xanh Cyan tươi */
  color: #ffffff !important;
}

/* Mục đang được chọn hiện tại */
.el-cascader-node.is-active {
  color: #38bdf8 !important;
  font-weight: 700 !important;
}

/* Đường ranh giới giữa các cột Menu Cascader */
.el-cascader-menu {
  border-right: 1px solid #334155 !important;
}

/* Tam giác chỉ mũi tên nhỏ trên cùng của Menu */
.el-popper[x-placement^="bottom"] .popper__arrow,
.el-popper[x-placement^="bottom"] .popper__arrow::after {
  border-bottom-color: #1e293b !important;
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