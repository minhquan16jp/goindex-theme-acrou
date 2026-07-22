<template>
  <table class="table is-hoverable">
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
      >
        <!-- Cột 1: Tên file / thư mục -->
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
          style="vertical-align: middle;"
        >
          <div class="file-name-wrapper">
            <svg class="iconfont" aria-hidden="true" style="margin-right: 8px;">
              <use :xlink:href="icons(file.mimeType)" />
            </svg>
            <span class="file-title">{{ file.name }}</span>
          </div>

          <span
            class="has-text-grey g2-file-desc"
            v-if="isShowDesc"
            v-html="file.description"
          ></span>
        </td>

        <!-- Cột 2: Cột Trạng thái (Thay thế Ngày sửa đổi) -->
        <td style="vertical-align: middle; text-align: center; width: 140px;">
          <button 
            type="button"
            class="status-btn"
            :class="isMarked(file.name) ? 'btn-marked' : 'btn-unmarked'"
            @click.stop="toggleMark(file.name)"
          >
            <span v-if="isMarked(file.name)" class="btn-content">
              <svg class="check-icon" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clip-rule="evenodd"/>
              </svg>
              Đã học
            </span>
            <span v-else class="btn-content">
              <span class="dot-icon"></span>
              Đánh dấu
            </span>
          </button>
        </td>

        <!-- Cột 3: Dung lượng -->
        <td class="is-hidden-mobile is-hidden-touch" style="vertical-align: middle;">
          {{ file.size }}
        </td>

        <!-- Cột 4: Thao tác (Copy, Tab mới, Tải về) -->
        <td class="is-hidden-mobile is-hidden-touch" style="vertical-align: middle;">
          <span class="icon action-icon" @click.stop="action(file,'copy')">
            <i
              class="fa fa-copy faa-shake animated-hover"
              :title="$t('list.opt.copy')"
              aria-hidden="true"
            ></i>
          </span>
          <span class="icon action-icon" @click.stop="action(file, '_blank')">
            <i
              class="fa fa-external-link faa-shake animated-hover"
              :title="$t('list.opt.newTab')"
              aria-hidden="true"
            ></i>
          </span>
          <span
            class="icon action-icon"
            @click.stop="action(file, 'down')"
            v-if="file.mimeType !== 'application/vnd.google-apps.folder'"
          >
            <i
              class="fa fa-download faa-shake animated-hover"
              aria-hidden="true"
              :title="$t('list.opt.download')"
            ></i>
          </span>
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
          name: "Trạng thái", // Thay đổi tên cột
          style: "width: 140px; text-align: center;",
        },
        {
          name: this.$t("list.title.size"),
          style: "width: 12%",
          class: "is-hidden-mobile is-hidden-touch",
        },
        {
          name: this.$t("list.title.operation"),
          style: "width: 14%",
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
/* Định dạng dòng đã học */
.row-watched {
  opacity: 0.55;
  background-color: #fafafa;
}

.file-name-wrapper {
  display: inline-flex;
  align-items: center;
}

.file-title {
  font-weight: 500;
  color: #2c3e50;
}

/* Nút bấm thiết kế hiện đại (Pill Badge) */
.status-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 4px 12px;
  font-size: 12px;
  font-weight: 600;
  border-radius: 20px;
  border: 1px solid transparent;
  cursor: pointer;
  transition: all 0.2s ease-in-out;
  outline: none;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
}

/* Trạng thái chưa đánh dấu */
.btn-unmarked {
  background-color: #f3f4f6;
  color: #6b7280;
  border-color: #e5e7eb;
}

.btn-unmarked:hover {
  background-color: #e5e7eb;
  color: #374151;
  transform: translateY(-1px);
}

/* Trạng thái đã đánh dấu */
.btn-marked {
  background-color: #ecfdf5;
  color: #059669;
  border-color: #a7f3d0;
}

.btn-marked:hover {
  background-color: #d1fae5;
  transform: translateY(-1px);
}

.btn-content {
  display: flex;
  align-items: center;
  gap: 4px;
}

.check-icon {
  width: 14px;
  height: 14px;
}

.dot-icon {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background-color: #9ca3af;
  display: inline-block;
}

.action-icon {
  cursor: pointer;
  margin-right: 8px;
  transition: color 0.15s ease;
}

.action-icon:hover {
  color: #3273dc;
}
</style>