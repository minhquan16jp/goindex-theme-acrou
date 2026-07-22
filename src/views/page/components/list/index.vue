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
          <span class="caret-wrapper">
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
        :style="{ opacity: isMarked(file.name) ? '0.45' : '1' }"
      >
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
        >
          <svg class="iconfont" aria-hidden="true">
            <use :xlink:href="icons(file.mimeType)" />
          </svg>
          {{ file.name }}
          
          <!-- Nút Đánh Dấu / Bỏ Đánh Dấu -->
          <button 
            type="button"
            @click.stop="toggleMark(file.name)"
            :style="{
              marginLeft: '10px',
              padding: '2px 8px',
              fontSize: '11px',
              borderRadius: '4px',
              border: '1px solid',
              cursor: 'pointer',
              backgroundColor: isMarked(file.name) ? '#e8f5e9' : '#f5f5f5',
              color: isMarked(file.name) ? '#2e7d32' : '#666',
              borderColor: isMarked(file.name) ? '#81c784' : '#ccc'
            }"
          >
            {{ isMarked(file.name) ? '✅ Đã xem' : '⚪ Đánh dấu' }}
          </button>

          <span
            class="has-text-grey g2-file-desc"
            v-if="isShowDesc"
            v-html="file.description"
          ></span>
        </td>
        <td class="is-hidden-mobile is-hidden-touch">
          {{ file.modifiedTime }}
        </td>
        <td class="is-hidden-mobile is-hidden-touch">{{ file.size }}</td>
        <td class="is-hidden-mobile is-hidden-touch">
          <span class="icon" @click.stop="action(file,'copy')">
            <i
              class="fa fa-copy faa-shake animated-hover"
              :title="$t('list.opt.copy')"
              aria-hidden="true"
            ></i>
          </span>
          <span class="icon" @click.stop="action(file, '_blank')">
            <i
              class="fa fa-external-link faa-shake animated-hover"
              :title="$t('list.opt.newTab')"
              aria-hidden="true"
            ></i>
          </span>
          <span
            class="icon"
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
      // Biến trigger để Vue nhận biết khi localStorage thay đổi
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
          name: this.$t("list.title.moditime"),
          style: "width:20%",
          class: "is-hidden-mobile is-hidden-touch",
        },
        {
          name: this.$t("list.title.size"),
          style: "width:10.5%",
          class: "is-hidden-mobile is-hidden-touch",
        },
        {
          name: this.$t("list.title.operation"),
          style: "width:13.5%",
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
        this.markedStore = { ...store }; // Cập nhật lại state để Vue re-render ngay
      } catch(e) {}
    }
  }
};
</script>