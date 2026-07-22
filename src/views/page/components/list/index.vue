<template>
  <table class="smooth-table">
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
            class="status-btn"
            :class="isMarked(file.name) ? 'btn-marked' : 'btn-unmarked'"
            @click.stop="toggleMark(file.name)"
          >
            {{ isMarked(file.name) ? '✓ Đã học' : '• Đánh dấu' }}
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
          style: "width: 130px; text-align: center;",
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
.smooth-table {
  width: 100%;
  border-collapse: separate;
  border-spacing: 0;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
}

.smooth-table th {
  padding: 12px 14px;
  font-size: 12px;
  font-weight: 700;
  color: #64748b;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  border-bottom: 2px solid #f1f5f9;
}

.table-row {
  transition: all 0.2s cubic-bezier(0.16, 1, 0.3, 1);
}

.table-row:hover {
  background-color: #f8fafc !important;
}

.table-row td {
  padding: 14px;
  border-bottom: 1px solid #f1f5f9;
  vertical-align: middle;
}

.table-row:last-child td {
  border-bottom: none;
}

.row-watched {
  opacity: 0.5;
}

.row-watched .file-name-text {
  text-decoration: line-through;
  color: #94a3b8;
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
  color: #1e293b;
  transition: color 0.15s ease;
}

.table-row:hover .file-name-text {
  color: #0284c7;
}

.status-cell {
  text-align: center;
}

.status-btn {
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
  background-color: #f1f5f9;
  color: #64748b;
  border-color: #e2e8f0;
}

.btn-unmarked:hover {
  background-color: #e2e8f0;
  color: #334155;
}

.btn-marked {
  background-color: #ecfdf5;
  color: #059669;
  border-color: #a7f3d0;
}

.size-cell {
  text-align: right;
}

.size-badge {
  font-size: 12px;
  color: #64748b;
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
  border: 1px solid #e2e8f0;
  background-color: #ffffff;
  color: #64748b;
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