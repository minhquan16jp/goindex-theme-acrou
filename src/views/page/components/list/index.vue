<template>
  <table class="custom-table">
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
/* Biến màu cho Table Light Mode */
.custom-table {
  --th-color: #64748b;
  --border-color: #f1f5f9;
  --text-color: #1e293b;
  --hover-bg: #f8fafc;
  --unmarked-bg: #f1f5f9;
  --unmarked-text: #64748b;
  --unmarked-border: #e2e8f0;
  --btn-bg: #ffffff;
  --btn-border: #e2e8f0;
  --btn-text: #64748b;
  
  width: 100%;
  border-collapse: separate;
  border-spacing: 0;
}

/* Biến màu cho Table Dark Mode */
@media (prefers-color-scheme: dark) {
  .custom-table {
    --th-color: #94a3b8;
    --border-color: #334155;
    --text-color: #f1f5f9;
    --hover-bg: #334155;
    --unmarked-bg: #334155;
    --unmarked-text: #cbd5e1;
    --unmarked-border: #475569;
    --btn-bg: #334155;
    --btn-border: #475569;
    --btn-text: #cbd5e1;
  }
}

.custom-table th {
  padding: 12px 14px;
  font-size: 12px;
  font-weight: 700;
  color: var(--th-color);
  text-transform: uppercase;
  letter-spacing: 0.6px;
  border-bottom: 2px solid var(--border-color);
}

.table-row {
  transition: all 0.18s ease-in-out;
}

.table-row:hover {
  background-color: var(--hover-bg) !important;
}

.table-row td {
  padding: 14px;
  border-bottom: 1px solid var(--border-color);
  vertical-align: middle;
}

.table-row:last-child td {
  border-bottom: none;
}

.row-watched {
  opacity: 0.45;
}

.row-watched .file-name-text {
  text-decoration: line-through;
  color: #64748b;
}

.file-item-group {
  display: inline-flex;
  align-items: center;
  gap: 10px;
}

.file-icon {
  width: 22px;
  height: 22px;
  flex-shrink: 0;
}

.file-name-text {
  font-size: 14px;
  font-weight: 500;
  color: var(--text-color);
  transition: color 0.15s ease;
}

.table-row:hover .file-name-text {
  color: #38bdf8;
}

.status-cell {
  text-align: center;
}

.modern-status-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 5px 14px;
  font-size: 12px;
  font-weight: 600;
  border-radius: 20px;
  border: 1px solid transparent;
  cursor: pointer;
  transition: all 0.2s ease;
  outline: none;
}

.btn-unmarked {
  background-color: var(--unmarked-bg);
  color: var(--unmarked-text);
  border-color: var(--unmarked-border);
}

.btn-unmarked:hover {
  opacity: 0.8;
}

.btn-marked {
  background-color: #064e3b;
  color: #34d399;
  border-color: #059669;
}

.btn-marked:hover {
  background-color: #047857;
}

.btn-inner {
  display: flex;
  align-items: center;
  gap: 5px;
}

.icon-svg {
  width: 14px;
  height: 14px;
}

.unmarked-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background-color: #94a3b8;
}

.size-cell {
  text-align: right;
}

.size-badge {
  font-size: 12px;
  color: var(--th-color);
  font-family: monospace;
}

.action-cell {
  text-align: center;
}

.action-btn-group {
  display: inline-flex;
  align-items: center;
  gap: 6px;
}

.action-btn {
  width: 30px;
  height: 30px;
  border-radius: 8px;
  border: 1px solid var(--btn-border);
  background-color: var(--btn-bg);
  color: var(--btn-text);
  display: inline-flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-size: 12px;
  transition: all 0.15s ease;
}

.action-btn:hover {
  background-color: #0284c7;
  color: #ffffff;
  border-color: #0284c7;
}

.download-btn:hover {
  background-color: #10b981;
  border-color: #10b981;
}
</style>