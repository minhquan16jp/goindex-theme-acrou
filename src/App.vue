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
/* =========================================================
   FIX BẮT BUỘC CHO CÁC DROPDOWN/POPER BỊ VĂNG RA KHỎI VUE ROOT
   ========================================================= */

/* 1. Nền tối cho khung Cascader Dropdown */
body .el-cascader__dropdown,
body .el-cascader-panel,
body .el-cascader-menu,
body .el-cascader-menu__list,
body .el-dropdown-menu,
body .el-popper {
  background-color: #1e293b !important;
  border: 1px solid #334155 !important;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.6) !important;
  border-radius: 12px !important;
}

/* 2. Màu nền và chữ từng nút trong Menu Cascader */
body .el-cascader-node,
body .el-dropdown-menu__item {
  background-color: #1e293b !important;
  color: #f8fafc !important; /* Chữ trắng sáng rõ nét */
  border-bottom: 1px solid rgba(255, 255, 255, 0.05) !important;
}

/* 3. Hiệu ứng Hover di chuột vào từng dòng */
body .el-cascader-node:hover,
body .el-cascader-node:focus,
body .el-cascader-node.is-selectable:hover,
body .el-dropdown-menu__item:hover {
  background-color: #0284c7 !important; /* Xanh Cyan tươi nổi bật */
  color: #ffffff !important;
}

/* 4. Mục đang được chọn */
body .el-cascader-node.is-active {
  color: #38bdf8 !important;
  font-weight: 700 !important;
}

/* 5. Cột đường kẻ phân cách giữa các menu */
body .el-cascader-menu {
  border-right: 1px solid #334155 !important;
}

/* Mũi tên nhỏ trên đỉnh */
body .el-popper[x-placement^="bottom"] .popper__arrow,
body .el-popper[x-placement^="bottom"] .popper__arrow::after {
  border-bottom-color: #1e293b !important;
}
</style>