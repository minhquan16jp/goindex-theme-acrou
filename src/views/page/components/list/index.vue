<template>
  <table class="dark-table">
    <thead>
      <tr>
        <th
          v-for="(column, index) in columns"
          v-bind:key="index"
          :class="column.class"
          :style="column.style"
        >
          {{ column.name }}
          <span class="caret-wrapper" v-if="index === 0">
            <i class="sort-caret ascending"></i>
            <i class="sort-caret descending"></i>
          </span>
        </th>
      </tr>
    </thead>
    <tbody>
      <tr 
        v-for="(file, index) in data" 
        v-bind:key="index"
        :class="{ 'row-watched': isMarked(file.name) }"
        class="table-row"
      >
        <!-- Cột 1: Tên File/Folder -->
        <td
          @click.self="
            action(
              file,
              file.mimeType !== 'application/vnd.google-apps.folder'
                ? 'view'
                : ''
            )
          "
          :title="file.name"
          class="name-cell"
        >
          <div class="file-item-group">
            <svg class="iconfont file-icon" aria-hidden="true">
              <use :xlink:href="icons(file.mimeType)" />
            </svg>
            <span class="file-name-text">{{ file.name }}</span>
          </div>

          <span
            class="has-text-grey g2-file-desc"
            v-if="isShowDesc"
            v-html="file.description"
          ></span>
        </td>

        <!-- Cột 2: Trạng Thái Học Tập -->
        <td class="status-cell">
          <button 
            type="button"
            class="modern-status-btn"
            :class="isMarked(file.name) ? 'btn-marked' : 'btn-unmarked'"
            @click.stop="toggleMark(file.name)"
          >
            <span v-if="isMarked(file.name)" class="btn-inner">
              <svg class="icon-svg" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clip-rule="evenodd"/>
              </svg>
              Đã học
            </span>
            <span v-else class="btn-inner">
              <span class="unmarked-dot"></span>
              Đánh dấu
            </span>
          </button>
        </td>

        <!-- Cột 3: Dung Lượng -->
        <td class="is-hidden-mobile is-hidden-touch size-cell">
          <span class="size-badge">{{ file.size }}</span>
        </td>

        <!-- Cột 4: Nút Thao Tác -->
        <td class="is-hidden-mobile is-hidden-touch action-cell">
          <div class="action-btn-group">
            <button class="action-btn" @click.stop="action(file,'copy')" :title="$t('list.opt.copy')">
              <i class="fa fa-copy"></i>
            </button>
            <button class="action-btn" @click.stop="action(file, '_blank')" :title="$t('list.opt.newTab')">
              <i class="fa fa-external-link"></i>
            </button>
            <button 
              class="action-btn download-btn" 
              @click.stop="action(file, 'down')"
              v-if="file.mimeType !== 'application/vnd.google-apps.folder'"
              :title="$t('list.opt.download')"
            >
              <i class="fa fa-download"></i>
            </button>
          </div>
        </td>
      </tr>
    </tbody>
  </table>
</template>

<script>
export default {
  props: {
    data: {
      type: Array,
      default: () => [],
    },
    icons: {
      type: Function,
    },
    action: {
      type: Function,
    }
  },
  data() {
    return {
      markedStore: {}
    };
  },
  created() {
    this.loadStore();
  },
  computed: {
    columns() {
      return [
        { name: this.$t("list.title.file"), style: "" },
        {
          name: "Trạng thái",
          style: "width: 140px; text-align: center;",
        },
        {
          name: this.$t("list.title.size"),
          style: "width: 12%; text-align: right;",
          class: "is-hidden-mobile is-hidden-touch",
        },
        {
          name: this.$t("list.title.operation"),
          style: "width: 14%; text-align: center;",
          class: "is-hidden-mobile is-hidden-touch",
        },
      ];
    },
    isShowDesc() {
      return window.themeOptions.render.desc || false;
    },
  },
  methods: {
    loadStore() {
      try {
        this.markedStore = JSON.parse(localStorage.getItem('minkuan_manual_marked') || '{}');
      } catch(e) {
        this.markedStore = {};
      }
    },
    isMarked(name) {
      return !!this.markedStore[name];
    },
    toggleMark(name) {
      try {
        let store = JSON.parse(localStorage.getItem('minkuan_manual_marked') || '{}');
        if (store[name]) {
          delete store[name];
        } else {
          store[name] = true;
        }
        localStorage.setItem('minkuan_manual_marked', JSON.stringify(store));
        this.markedStore = { ...store };
      } catch(e) {}
    }
  }
};
</script>

<style scoped>
.dark-table {
  width: 100%;
  border-collapse: separate;
  border-spacing: 0;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
}

/* Header Bảng */
.dark-table th {
  padding: 14px;
  font-size: 11px;
  font-weight: 800;
  color: #94a3b8 !important; /* Màu chữ tiêu đề sáng rõ */
  text-transform: uppercase;
  letter-spacing: 0.8px;
  border-bottom: 2px solid #334155 !important;
}

/* Các Dòng Bảng */
.table-row {
  transition: all 0.2s ease;
}

.table-row:hover {
  background-color: #334155 !important; /* Sáng dòng lên khi di chuột */
}

.table-row td {
  padding: 16px 14px;
  border-bottom: 1px solid #334155 !important;
  vertical-align: middle;
}

.table-row:last-child td {
  border-bottom: none !important;
}

/* Dòng Đã Học */
.row-watched {
  opacity: 0.4;
}

.row-watched .file-name-text {
  text-decoration: line-through;
  color: #64748b !important;
}

/* Tên File/Folder - ĐẶC BIỆT CẢI TIẾN MÀU TRẮNG SÁNG */
.file-item-group {
  display: inline-flex;
  align-items: center;
  gap: 12px;
}

.file-icon {
  width: 24px;
  height: 24px;
  flex-shrink: 0;
}

.file-name-text {
  font-size: 14px;
  font-weight: 600;
  color: #f8fafc !important; /* TRẮNG TÍNH SÁNG NÉT, RÕ RÀNG 100% */
  transition: color 0.15s ease;
}

.table-row:hover .file-name-text {
  color: #38bdf8 !important; /* Xanh Neon sáng khi hover */
}

/* Cột Trạng Thái */
.status-cell {
  text-align: center;
}

.modern-status-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 6px 14px;
  font-size: 12px;
  font-weight: 700;
  border-radius: 20px;
  border: 1px solid transparent;
  cursor: pointer;
  transition: all 0.2s ease;
  outline: none;
}

/* Nút Chưa Đánh Dấu (Dark UI) */
.btn-unmarked {
  background-color: #0f172a !important;
  color: #94a3b8 !important;
  border-color: #334155 !important;
}

.btn-unmarked:hover {
  background-color: #334155 !important;
  color: #ffffff !important;
  border-color: #475569 !important;
}

/* Nút Đã Học (Dark Neon Green) */
.btn-marked {
  background-color: rgba(16, 185, 129, 0.2) !important;
  color: #34d399 !important;
  border-color: rgba(52, 211, 153, 0.4) !important;
  box-shadow: 0 0 10px rgba(52, 211, 153, 0.2);
}

.btn-marked:hover {
  background-color: rgba(16, 185, 129, 0.3) !important;
}

.btn-inner {
  display: flex;
  align-items: center;
  gap: 6px;
}

.icon-svg {
  width: 14px;
  height: 14px;
}

.unmarked-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background-color: #64748b;
}

/* Dung Lượng */
.size-cell {
  text-align: right;
}

.size-badge {
  font-size: 12px;
  color: #94a3b8 !important;
  font-family: monospace;
}

/* Thao Tác */
.action-cell {
  text-align: center;
}

.action-btn-group {
  display: inline-flex;
  align-items: center;
  gap: 8px;
}

.action-btn {
  width: 32px;
  height: 32px;
  border-radius: 8px;
  border: 1px solid #334155 !important;
  background-color: #0f172a !important;
  color: #cbd5e1 !important;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-size: 13px;
  transition: all 0.2s ease;
}

.action-btn:hover {
  background-color: #38bdf8 !important;
  color: #0f172a !important;
  border-color: #38bdf8 !important;
  box-shadow: 0 0 12px rgba(56, 189, 248, 0.4);
}

.download-btn:hover {
  background-color: #34d399 !important;
  color: #0f172a !important;
  border-color: #34d399 !important;
  box-shadow: 0 0 12px rgba(52, 211, 153, 0.4);
}
</style>