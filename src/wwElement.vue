<template>
  <div ref="gridWrapperRef" class="ww-datagrid" :class="{ editing: isEditing, 'rows-clickable': content.enableClickSelection && content.rowSelection !== 'none' }" :style="cssVars">

    <!-- TOP TOOLBAR -->
    <div v-if="content.showToolbar" class="ww-datagrid-toolbar">
      <div class="ww-datagrid-toolbar__left">
        <span class="ww-datagrid-results-label">{{ content.resultsLabel || 'Results' }}</span>
        <span class="ww-datagrid-results-badge">{{ totalRows }} {{ content.rowsNoun || 'rows' }}</span>
      </div>
      <div class="ww-datagrid-toolbar__right" v-if="content.pagination && effectivePageSizes.length">
        <label class="ww-datagrid-rpp-label">{{ content.rowsPerPageLabel || 'Rows per page' }}</label>
        <div class="ww-datagrid-rpp-select-wrapper">
          <select class="ww-datagrid-rpp-select" :value="currentPageSize" @change="onPageSizeChange">
            <option v-for="size in effectivePageSizes" :key="size" :value="size">{{ size }}</option>
          </select>
          <svg class="ww-datagrid-rpp-chevron" viewBox="0 0 10 6" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M1 1L5 5L9 1" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </div>
      </div>
    </div>

    <!-- AG GRID -->
    <ag-grid-vue
      :rowData="rowData"
      :columnDefs="columnDefs"
      :initial-state="initialState"
      :defaultColDef="defaultColDef"
      :domLayout="content.layout === 'auto' ? 'autoHeight' : 'normal'"
      :style="style"
      :rowSelection="rowSelection"
      :selection-column-def="{ pinned: true }"
      :theme="theme"
      :getRowId="getRowId"
      :pagination="content.pagination"
      :paginationPageSize="computedPaginationPageSize"
      :paginationPageSizeSelector="false"
      :suppressPaginationPanel="content.pagination && content.showToolbar"
      :suppressMovableColumns="!content.movableColumns"
      :columnHoverHighlight="content.columnHoverHighlight"
      :locale-text="localeText"
      enableCellTextSelection
      ensureDomOrder
      :row-drag-managed="true"
      @grid-ready="onGridReady"
      @row-selected="onRowSelected"
      @selection-changed="onSelectionChanged"
      @cell-value-changed="onCellValueChanged"
      @filter-changed="onFilterChanged"
      @sort-changed="onSortChanged"
      @row-clicked="onRowClicked"
      @row-drag-end="onRowDragged"
      @row-drag-enter="onRowDragEnter"
      @column-moved="onColumnMoved"
      @pagination-changed="onPaginationChanged"
    >
    </ag-grid-vue>

    <!-- BOTTOM FOOTER -->
    <div v-if="content.showToolbar && content.pagination" class="ww-datagrid-footer">
      <div class="ww-datagrid-footer__info">
        {{ showingText }}
      </div>
      <div class="ww-datagrid-footer__pagination">

        <!-- |< First page -->
        <button
          class="ww-datagrid-page-btn ww-datagrid-page-btn--nav"
          :disabled="effectiveCurrentPage === 0"
          @click="goToPage(0)"
          aria-label="First page"
        >
          <svg class="nav-icon--wide" viewBox="0 0 10 10" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M2 1V9M8 1L4 5L8 9" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </button>

        <!-- << Skip back by window size -->
        <button
          class="ww-datagrid-page-btn ww-datagrid-page-btn--nav"
          :disabled="effectiveCurrentPage === 0"
          @click="goToPage(Math.max(0, effectiveCurrentPage - pageSkipSize))"
          aria-label="Skip back"
        >
          <svg class="nav-icon--xwide" viewBox="0 0 14 10" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M7 1L3 5L7 9M13 1L9 5L13 9" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </button>

        <!-- < Previous page -->
        <button
          class="ww-datagrid-page-btn ww-datagrid-page-btn--nav"
          :disabled="effectiveCurrentPage === 0"
          @click="goToPage(effectiveCurrentPage - 1)"
          aria-label="Previous page"
        >
          <svg viewBox="0 0 6 10" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M5 1L1 5L5 9" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </button>

        <!-- Page number buttons -->
        <template v-for="page in visiblePages" :key="page">
          <span v-if="page === '...'" class="ww-datagrid-page-ellipsis">…</span>
          <button
            v-else
            class="ww-datagrid-page-btn"
            :class="{ active: page - 1 === effectiveCurrentPage }"
            @click="goToPage(page - 1)"
          >
            {{ page }}
          </button>
        </template>

        <!-- > Next page -->
        <button
          class="ww-datagrid-page-btn ww-datagrid-page-btn--nav"
          :disabled="effectiveCurrentPage >= effectiveTotalPages - 1"
          @click="goToPage(effectiveCurrentPage + 1)"
          aria-label="Next page"
        >
          <svg viewBox="0 0 6 10" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M1 1L5 5L1 9" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </button>

        <!-- >> Skip forward by window size -->
        <button
          class="ww-datagrid-page-btn ww-datagrid-page-btn--nav"
          :disabled="effectiveCurrentPage >= effectiveTotalPages - 1"
          @click="goToPage(Math.min(effectiveTotalPages - 1, effectiveCurrentPage + pageSkipSize))"
          aria-label="Skip forward"
        >
          <svg class="nav-icon--xwide" viewBox="0 0 14 10" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M1 1L5 5L1 9M7 1L11 5L7 9" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </button>

        <!-- >| Last page -->
        <button
          class="ww-datagrid-page-btn ww-datagrid-page-btn--nav"
          :disabled="effectiveCurrentPage >= effectiveTotalPages - 1"
          @click="goToPage(effectiveTotalPages - 1)"
          aria-label="Last page"
        >
          <svg class="nav-icon--wide" viewBox="0 0 10 10" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M8 1V9M2 1L6 5L2 9" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </button>

      </div>
    </div>

  </div>
</template>

<script>
import {
  shallowRef,
  watchEffect,
  computed,
  inject,
  watch,
  nextTick,
  ref,
  onMounted,
  onUnmounted,
} from "vue";
import { AgGridVue } from "ag-grid-vue3";
import {
  AllCommunityModule,
  ModuleRegistry,
  themeQuartz,
} from "ag-grid-community";
import {
  AG_GRID_LOCALE_EN,
  AG_GRID_LOCALE_FR,
  AG_GRID_LOCALE_DE,
  AG_GRID_LOCALE_ES,
  AG_GRID_LOCALE_PT,
} from "@ag-grid-community/locale";
import ActionCellRenderer from "./components/ActionCellRenderer.vue";
import ImageCellRenderer from "./components/ImageCellRenderer.vue";
import WewebCellRenderer from "./components/WewebCellRenderer.vue";
import PhoneCellEditor from "./components/PhoneCellEditor.vue";

ModuleRegistry.registerModules([AllCommunityModule]);

export default {
  components: {
    AgGridVue,
    ActionCellRenderer,
    ImageCellRenderer,
    WewebCellRenderer,
    PhoneCellEditor,
  },
  props: {
    content: {
      type: Object,
      required: true,
    },
    uid: {
      type: String,
      required: true,
    },
    /* wwEditor:start */
    wwEditorState: { type: Object, required: true },
    /* wwEditor:end */
  },
  emits: ["trigger-event", "update:content:effect"],
  setup(props, ctx) {
    const { resolveMappingFormula } = wwLib.wwFormula.useFormula();

    const gridApi = shallowRef(null);

    // Pagination state tracked locally for custom UI
    const currentPage = ref(0);
    const totalPages = ref(0);
    const currentPageSize = ref(
      props.content.paginationPageSizeSelector?.[0] ||
      props.content.paginationPageSize ||
      10
    );

    const { value: selectedRows, setValue: setSelectedRows } =
      wwLib.wwVariable.useComponentVariable({
        uid: props.uid,
        name: "selectedRows",
        type: "array",
        defaultValue: [],
        readonly: true,
      });
    const { value: filterValue, setValue: setFilters } =
      wwLib.wwVariable.useComponentVariable({
        uid: props.uid,
        name: "filters",
        type: "object",
        defaultValue: {},
        readonly: true,
      });
    const { value: sortValue, setValue: setSort } =
      wwLib.wwVariable.useComponentVariable({
        uid: props.uid,
        name: "sort",
        type: "object",
        defaultValue: {},
        readonly: true,
      });
    const { value: columnOrder, setValue: setColumnOrder } =
      wwLib.wwVariable.useComponentVariable({
        uid: props.uid,
        name: "columnOrder",
        type: "array",
        defaultValue: [],
        readonly: true,
      });

    const { value: data, setValue: setData } =
      wwLib.wwVariable.useComponentVariable({
        uid: props.uid,
        name: "data",
        type: "object",
        defaultValue: {
          allData: [],
          total: 0,
          sortedFilteredData: [],
          totalSortedFilteredData: 0,
          perPageTotal: 0,
          totalPages: 0,
          displayedData: [],
          totalDisplayedRecords: 0,
        },
        readonly: true,
      });

    const onGridReady = (params) => {
      gridApi.value = params.api;
      const columns = params.api.getAllGridColumns();
      setColumnOrder(columns.map((col) => col.getColId()));
    };

    // ── Visibility / selection redraw ────────────────────────────────────
    // WeWeb tabs use display:none to hide content rather than unmounting it.
    // IntersectionObserver doesn't fire reliably in this case, so instead we
    // use a MutationObserver to watch the DOM tree for any ancestor becoming
    // visible again, then redraw rows to restore the selection highlight.
    const gridWrapperRef = ref(null);
    let mutationObserver = null;
    let wasHidden = false;

    function redrawIfSelected() {
      if (!gridApi.value) return;
      const selected = gridApi.value.getSelectedRows();
      if (selected && selected.length > 0) {
        nextTick(() => { gridApi.value?.redrawRows(); });
      }
    }

    function isElementHidden(el) {
      // Walk up the DOM — if any ancestor has display:none the grid is hidden
      let node = el;
      while (node && node !== document.body) {
        const style = window.getComputedStyle(node);
        if (style.display === 'none' || style.visibility === 'hidden') return true;
        node = node.parentElement;
      }
      return false;
    }

    onMounted(() => {
      const el = gridWrapperRef.value;
      if (!el || typeof MutationObserver === 'undefined') return;

      // Check initial state
      wasHidden = isElementHidden(el);

      mutationObserver = new MutationObserver(() => {
        const hidden = isElementHidden(el);
        if (wasHidden && !hidden) {
          // Grid just became visible — restore selection highlight
          redrawIfSelected();
        }
        wasHidden = hidden;
      });

      // Observe the entire subtree of the body for style/class/attribute
      // changes that could affect display — covers all tab implementations
      mutationObserver.observe(document.body, {
        attributes: true,
        attributeFilter: ['style', 'class'],
        subtree: true,
      });
    });

    onUnmounted(() => {
      if (mutationObserver) {
        mutationObserver.disconnect();
        mutationObserver = null;
      }
    });

    let initialFilter = "";
    let initialSort = "";

    watchEffect(() => {
      if (!gridApi.value) return;
      if (
        props.content.initialFilters &&
        initialFilter !== JSON.stringify(props.content.initialFilters)
      ) {
        gridApi.value.setFilterModel(props.content.initialFilters);
        initialFilter = JSON.stringify(props.content.initialFilters);
      }
      if (
        props.content.initialSort &&
        initialSort !== JSON.stringify(props.content.initialSort)
      ) {
        gridApi.value.applyColumnState({
          state: props.content.initialSort || [],
          defaultState: { sort: null },
        });
        initialSort = JSON.stringify(props.content.initialSort);
      }
    });

    watchEffect(() => {
      if (!gridApi.value) return;
      if (props.content.initialColumnsOrder) {
        gridApi.value.applyColumnState({
          state: props.content.initialColumnsOrder.map((colId) => ({ colId })),
          applyOrder: true,
        });
      }
    });

    let rafId = null;
    const scheduleVariableUpdate = () => {
      if (rafId !== null) return;
      rafId = requestAnimationFrame(() => {
        rafId = null;
        updateVariables();
      });
    };

    function updateVariables() {
      if (!gridApi.value) return;

      const isServerSide =
        props.content.serverTotalItems != null &&
        props.content.serverTotalItems > 0;

      // Collect all rows currently loaded in AG Grid
      const sortedFiltered = [];
      gridApi.value.forEachNodeAfterFilterAndSort((node) => {
        sortedFiltered.push(node.data);
      });

      let displayed = [];
      let pg = 0;
      let tp = 1;
      let pageSize = currentPageSize.value;

      if (props.content.pagination) {
        if (isServerSide) {
          // ── Server-side mode ──────────────────────────────────────────────
          // All loaded rows are the current page's displayed rows
          displayed = sortedFiltered;
          pg = (props.content.serverCurrentPage || 1) - 1; // normalise to 0-based
          tp = props.content.serverTotalPages || 1;
          pageSize = props.content.serverItemsReceived || currentPageSize.value;
        } else {
          // ── Client-side mode ──────────────────────────────────────────────
          pageSize = gridApi.value.paginationGetPageSize();
          pg = gridApi.value.paginationGetCurrentPage();
          tp = gridApi.value.paginationGetTotalPages();
          const totalDisplayed = gridApi.value.getDisplayedRowCount();
          const start = pg * pageSize;
          const end = Math.min(start + pageSize, totalDisplayed);
          for (let i = start; i < end; i++) {
            const node = gridApi.value.getDisplayedRowAtIndex(i);
            displayed.push(node.data);
          }
        }

        // Keep local reactive state in sync for the custom pagination UI
        currentPage.value = pg;
        totalPages.value = tp;
      } else {
        displayed = sortedFiltered;
        currentPage.value = 0;
        totalPages.value = 1;
      }

      const dataValue = {
        // Raw loaded rows
        allData: data.value.allData,
        sortedFilteredData: sortedFiltered,
        displayedData: displayed,

        // Counts — prefer server values in server-side mode
        total: isServerSide
          ? (props.content.serverTotalItems ?? data.value.total)
          : data.value.total,
        totalSortedFilteredData: isServerSide
          ? (props.content.serverTotalItems ?? sortedFiltered.length)
          : sortedFiltered.length,
        totalDisplayedRecords: isServerSide
          ? (props.content.serverItemsReceived ?? displayed.length)
          : displayed.length,

        // Pagination meta — prefer server values in server-side mode
        perPageTotal: pageSize,
        totalPages: isServerSide
          ? (props.content.serverTotalPages ?? tp)
          : tp,
        currentPage: isServerSide
          ? (props.content.serverCurrentPage ?? pg + 1)
          : pg + 1,
      };

      setData(dataValue);
    }

    const rowData = computed(() => {
      const data = wwLib.wwUtils.getDataFromCollection(props.content.rowData);
      return Array.isArray(data) ? data ?? [] : [];
    });

    watch(
      rowData,
      (newVal) => {
        const dataValue = { ...data.value };
        dataValue.allData = newVal;
        dataValue.total = newVal.length;
        setData(dataValue);
        scheduleVariableUpdate();
      },
      { immediate: true, deep: true }
    );

    const initialState = computed(() => {
      const state = { partialColumnState: true };
      if (props.content.initialFilters) {
        state.filter = { filterModel: props.content.initialFilters };
      }
      if (props.content.initialSort) {
        state.sort = { sortModel: props.content.initialSort };
      }
      if (props.content.initialColumnsOrder) {
        state.columnOrder = { orderedColIds: props.content.initialColumnsOrder };
      }
      return state;
    });

    const onRowSelected = (event) => {
      const name = event.node.isSelected() ? "rowSelected" : "rowDeselected";
      ctx.emit("trigger-event", { name, event: { row: event.data } });
    };

    const onRowDragged = (event) => {
      const rows = [];
      event.api.forEachNode((node) => { rows.push(node.data); });
      ctx.emit("trigger-event", {
        name: "rowDragged",
        event: {
          row: event.node.data,
          id: event.node.id,
          targetIndex: event.overIndex,
          rows,
        },
      });
    };

    const onRowDragEnter = (event) => {
      ctx.emit("trigger-event", {
        name: "rowDragStart",
        event: { row: event.node.data, id: event.node.id },
      });
    };

    const onSelectionChanged = () => {
      if (!gridApi.value) return;
      setSelectedRows(gridApi.value.getSelectedRows() || []);
    };

    const onFilterChanged = () => {
      if (!gridApi.value) return;
      const filterModel = gridApi.value.getFilterModel();
      if (JSON.stringify(filterModel || {}) !== JSON.stringify(filterValue.value || {})) {
        setFilters(filterModel);
        ctx.emit("trigger-event", { name: "filterChanged", event: filterModel });
        scheduleVariableUpdate();
      }
    };

    const onSortChanged = () => {
      if (!gridApi.value) return;
      const state = gridApi.value.getState();
      if (
        JSON.stringify(state.sort?.sortModel || []) !==
        JSON.stringify(sortValue.value || [])
      ) {
        setSort(state.sort?.sortModel || []);
        ctx.emit("trigger-event", { name: "sortChanged", event: state.sort?.sortModel || [] });
        scheduleVariableUpdate();
      }
    };

    const onColumnMoved = (event) => {
      if (!event.finished || event.source !== "uiColumnMoved") return;
      const columns = event.api.getAllGridColumns();
      setColumnOrder(columns.map((col) => col.getColId()));
      ctx.emit("trigger-event", {
        name: "columnMoved",
        event: {
          toIndex: event.toIndex,
          columnId: event.column.getColId(),
          columnsOrder: columns.map((col) => col.getColId()),
        },
      });
    };

    const onPaginationChanged = () => {
      scheduleVariableUpdate();
    };

    watch(() => props.content.pagination, () => { scheduleVariableUpdate(); });

    // Re-run variable update whenever server-side values change (i.e. after a re-fetch)
    watch(
      () => [
        props.content.serverTotalItems,
        props.content.serverTotalPages,
        props.content.serverCurrentPage,
        props.content.serverItemsReceived,
      ],
      () => { scheduleVariableUpdate(); }
    );

    // Sync page size when setting changes externally
    watch(
      () => [props.content.paginationPageSize, props.content.paginationPageSizeSelector],
      ([newSize, newSelector]) => {
        if (newSelector && Array.isArray(newSelector) && newSelector.length) {
          currentPageSize.value = newSelector[0];
        } else if (newSize) {
          currentPageSize.value = newSize;
        }
      }
    );

    /* wwEditor:start */
    const { createElement } = wwLib.useCreateElement();
    /* wwEditor:end */

    watch(
      () => [
        props.content.dynamicHeaderBackgroundColor,
        props.content.useDynamicStyleHeader,
        props.content.dynamicHeaderTextColor,
        props.content.dynamicHeaderFontWeight,
        props.content.dynamicHeaderFontSize,
        props.content.dynamicHeaderFontFamily,
      ],
      () => {
        nextTick(() => { setTimeout(() => gridApi.value?.refreshHeader(), 100); });
      }
    );

    // Pagination helpers
    const goToPage = (page) => {
      // page is always 0-based internally
      const zeroBasedPage = Math.max(0, page);
      const oneBasedPage = zeroBasedPage + 1;

      if (props.content.serverTotalItems != null && props.content.serverTotalItems > 0) {
        // Server-side mode: don't touch AG Grid pagination at all.
        // Just emit the event so the WeWeb workflow can re-fetch with the new page.
        ctx.emit("trigger-event", {
          name: "pageChanged",
          event: { page: oneBasedPage, pageSize: currentPageSize.value },
        });
      } else {
        // Client-side mode: let AG Grid handle it, then emit for consistency.
        if (!gridApi.value) return;
        gridApi.value.paginationGoToPage(zeroBasedPage);
        ctx.emit("trigger-event", {
          name: "pageChanged",
          event: { page: oneBasedPage, pageSize: currentPageSize.value },
        });
      }
    };

    const onPageSizeChange = (e) => {
      const size = Number(e.target.value);
      currentPageSize.value = size;

      if (props.content.serverTotalItems != null && props.content.serverTotalItems > 0) {
        // Server-side mode: emit only, let the workflow re-fetch page 1 with new limit
        ctx.emit("trigger-event", {
          name: "pageSizeChanged",
          event: { pageSize: size, page: 1 },
        });
      } else {
        // Client-side mode: update AG Grid directly then emit
        if (gridApi.value) {
          gridApi.value.paginationSetPageSize(size);
          gridApi.value.paginationGoToPage(0);
        }
        ctx.emit("trigger-event", {
          name: "pageSizeChanged",
          event: { pageSize: size, page: 1 },
        });
      }
    };

    function refreshData() {
      nextTick(() => { gridApi.value?.refreshCells(); });
    }

    return {
      resolveMappingFormula,
      onGridReady,
      onRowSelected,
      onSelectionChanged,
      gridApi,
      onFilterChanged,
      onSortChanged,
      onPaginationChanged,
      currentPage,
      totalPages,
      currentPageSize,
      goToPage,
      onPageSizeChange,
      localeText: computed(() => {
        switch (props.content.lang) {
          case "fr": return AG_GRID_LOCALE_FR;
          case "de": return AG_GRID_LOCALE_DE;
          case "es": return AG_GRID_LOCALE_ES;
          case "pt": return AG_GRID_LOCALE_PT;
          case "custom": return { ...AG_GRID_LOCALE_EN, ...(props.content.localeText || {}) };
          default: return AG_GRID_LOCALE_EN;
        }
      }),
      onRowDragged,
      onRowDragEnter,
      onColumnMoved,
      initialState,
      refreshData,
      rowData,
      gridWrapperRef,
      /* wwEditor:start */
      createElement,
      rawContent: inject("componentRawContent", {}),
      /* wwEditor:end */
    };
  },
  computed: {
    isServerSide() {
      return this.content.serverTotalItems != null && this.content.serverTotalItems > 0;
    },
    totalRows() {
      if (this.isServerSide) return this.content.serverTotalItems;
      return this.rowData?.length || 0;
    },
    effectiveTotalPages() {
      if (this.isServerSide && this.content.serverTotalPages != null) {
        return this.content.serverTotalPages;
      }
      return this.totalPages;
    },
    effectiveCurrentPage() {
      // Server sends 1-based page; internally we use 0-based
      if (this.isServerSide && this.content.serverCurrentPage != null) {
        return this.content.serverCurrentPage - 1;
      }
      return this.currentPage;
    },
    effectivePageSizes() {
      if (
        this.content.hasPaginationSelector === "multiple" &&
        Array.isArray(this.content.paginationPageSizeSelector) &&
        this.content.paginationPageSizeSelector.length
      ) {
        return this.content.paginationPageSizeSelector;
      }
      const size = this.content.paginationPageSize || 10;
      return [size];
    },
    computedPaginationPageSize() {
      return this.currentPageSize;
    },
    showingText() {
      const noun = this.content.rowsNoun || 'rows';
      const total = this.totalRows;
      if (this.isServerSide) {
        const curPage = this.content.serverCurrentPage || 1;
        const pageSize = this.currentPageSize;
        const start = (curPage - 1) * pageSize + 1;
        const itemsReceived = this.content.serverItemsReceived || pageSize;
        const end = Math.min(start + itemsReceived - 1, total);
        return `Showing ${start}\u2013${end} of ${total} ${noun}`;
      }
      if (!this.gridApi) return '';
      const page = this.currentPage;
      const pageSize = this.currentPageSize;
      const start = page * pageSize + 1;
      const end = Math.min((page + 1) * pageSize, total);
      return `Showing ${start}\u2013${end} of ${total} ${noun}`;
    },
    pageSkipSize() {
      // Skip by the same count as the visible window so the page buttons
      // shift by exactly one full window — mirrors what the user sees.
      let max = Math.max(3, this.content.maxPaginationButtons || 7);
      if (max % 2 === 0) max += 1;
      const sideSlots = 2;
      const ellipsisSlots = 2;
      return Math.max(1, max - sideSlots - ellipsisSlots);
    },
    visiblePages() {
      const total = this.effectiveTotalPages;
      const current = this.effectiveCurrentPage + 1; // 1-based for display

      // max must be odd and at least 3 so there's always room for current +
      // neighbours. We clamp and force odd so the window is symmetric.
      let max = Math.max(3, this.content.maxPaginationButtons || 7);
      if (max % 2 === 0) max += 1;

      // If total pages fit within the window, just show them all
      if (total <= max) {
        return Array.from({ length: total }, (_, i) => i + 1);
      }

      // How many buttons are available for the middle window after reserving
      // first page, last page, and up to 2 ellipsis slots
      const sideSlots = 2; // first + last always shown
      const ellipsisSlots = 2; // up to 2 ellipses
      const windowSize = max - sideSlots - ellipsisSlots; // inner scrolling window
      const half = Math.floor(windowSize / 2);

      const pages = [];

      // Calculate the raw window around current
      let winStart = current - half;
      let winEnd = current + half;

      // Clamp window so it doesn't bleed into the first/last anchor pages
      if (winStart <= 2) {
        winStart = 2;
        winEnd = winStart + windowSize - 1;
      }
      if (winEnd >= total - 1) {
        winEnd = total - 1;
        winStart = winEnd - windowSize + 1;
      }

      // Build the page list
      pages.push(1);
      if (winStart > 2) pages.push('...');
      for (let i = winStart; i <= winEnd; i++) pages.push(i);
      if (winEnd < total - 1) pages.push('...');
      pages.push(total);

      return pages;
    },
    defaultColDef() {
      const definition = {
        editable: false,
        resizable: this.content.resizableColumns,
        autoHeaderHeight: this.content.headerHeightMode === "auto",
        wrapHeaderText: this.content.headerHeightMode === "auto",
        cellClass:
          this.content.cellAlignmentMode === "custom"
            ? `-${this.content.cellAlignment || "left"} ||`
            : null,
      };
      if (this.content.useDynamicStyleHeader) {
        definition.headerStyle = this.getHeaderStyle;
      } else {
        definition.headerStyle = {};
      }
      return definition;
    },
    columnDefs() {
      const columns = this.content.columns.map((col) => {
        const minWidth =
          !col?.minWidth || col?.minWidth === "auto" ? null
            : wwLib.wwUtils.getLengthUnit(col?.minWidth)?.[0];
        const maxWidth =
          !col?.maxWidth || col?.maxWidth === "auto" ? null
            : wwLib.wwUtils.getLengthUnit(col?.maxWidth)?.[0];
        const width =
          !col?.width || col?.width === "auto" || col?.widthAlgo === "flex" ? null
            : wwLib.wwUtils.getLengthUnit(col?.width)?.[0];
        const flex = col?.widthAlgo === "flex" ? col?.flex ?? 1 : null;
        const commonProperties = {
          minWidth, maxWidth,
          pinned: col?.pinned === "none" ? false : col?.pinned,
          width, flex,
          hide: !!col?.hide,
          headerClass: [
            col.headerAlignment ? `-${col.headerAlignment}` : null,
            col.editable ? '-editable' : null,
          ].filter(Boolean).join(' ') || null,
          ...(this.content.cellAlignmentMode !== "custom"
            ? { cellClass: col.cellAlignment ? `-${col.cellAlignment}` : null }
            : {}),
        };
        switch (col?.cellDataType) {
          case "action":
            return {
              ...commonProperties,
              headerName: col?.headerName,
              cellRenderer: "ActionCellRenderer",
              cellRendererParams: {
                name: col?.actionName,
                label: col?.actionLabel,
                trigger: this.onActionTrigger,
                withFont: !!this.content.actionFont,
              },
              sortable: false,
              filter: false,
              colId: col?.actionName,
            };
          case "custom":
            return {
              ...commonProperties,
              headerName: col?.headerName,
              field: col?.field,
              cellRenderer: "WewebCellRenderer",
              cellRendererParams: { containerId: col?.containerId },
              sortable: col?.sortable,
              filter: col?.filter,
            };
          case "image":
            return {
              ...commonProperties,
              headerName: col?.headerName,
              field: col?.field,
              cellRenderer: "ImageCellRenderer",
              cellRendererParams: { width: col?.imageWidth, height: col?.imageHeight },
            };
          default: {
            const result = {
              ...commonProperties,
              headerName: col?.headerName,
              field: col?.field,
              sortable: col?.sortable,
              filter: col?.filter,
              editable: col?.editable,
            };
            if (col?.editable && col?.editorType === 'phone') {
              result.cellEditor = 'PhoneCellEditor';
            }
            if (col?.useCustomLabel) {
              result.valueFormatter = (params) =>
                this.resolveMappingFormula(col?.displayLabelFormula, params.value);
            }
            return result;
          }
        }
      });

      if (this.content.rowReorder && columns[0]) {
        columns[0].rowDrag = true;
      }
      return columns;
    },
    rowSelection() {
      if (this.content.rowSelection === "multiple") {
        return {
          mode: "multiRow",
          checkboxes: !this.content.disableCheckboxes,
          headerCheckbox: !this.content.disableCheckboxes,
          selectAll: this.content.selectAll || "all",
          enableClickSelection: this.content.enableClickSelection,
        };
      } else if (this.content.rowSelection === "single") {
        return {
          mode: "singleRow",
          checkboxes: !this.content.disableCheckboxes,
          enableClickSelection: this.content.enableClickSelection,
        };
      } else {
        return {
          mode: "singleRow",
          checkboxes: false,
          isRowSelectable: () => false,
          enableClickSelection: this.content.enableClickSelection,
        };
      }
    },
    style() {
      if (this.content.layout === "auto") return {};
      return { height: this.content.height || "400px" };
    },
    cssVars() {
      return {
        // Action button vars
        "--ww-data-grid_action-backgroundColor": this.content.actionBackgroundColor,
        "--ww-data-grid_action-color": this.content.actionColor,
        "--ww-data-grid_action-padding": this.content.actionPadding,
        "--ww-data-grid_action-border": this.content.actionBorder,
        "--ww-data-grid_action-borderRadius": this.content.actionBorderRadius,
        ...(this.content.actionFont
          ? { "--ww-data-grid_action-font": this.content.actionFont }
          : {
              "--ww-data-grid_action-fontSize": this.content.actionFontSize,
              "--ww-data-grid_action-fontFamily": this.content.actionFontFamily,
              "--ww-data-grid_action-fontWeight": this.content.actionFontWeight,
              "--ww-data-grid_action-fontStyle": this.content.actionFontStyle,
              "--ww-data-grid_action-lineHeight": this.content.actionLineHeight,
            }),
        // Toolbar vars
        "--ww-data-grid_toolbar-backgroundColor": this.content.toolbarBackgroundColor,
        "--ww-data-grid_toolbar-borderColor": this.content.toolbarBorderColor,
        "--ww-data-grid_toolbar-padding": this.content.toolbarPadding,
        // Results label vars
        "--ww-data-grid_results-label-color": this.content.resultsLabelColor,
        "--ww-data-grid_results-label-fontSize": this.content.resultsLabelFontSize,
        "--ww-data-grid_results-label-fontWeight": this.content.resultsLabelFontWeight,
        "--ww-data-grid_results-label-fontFamily": this.content.resultsLabelFontFamily,
        // Results badge vars
        "--ww-data-grid_results-badge-backgroundColor": this.content.resultsBadgeBackgroundColor,
        "--ww-data-grid_results-badge-color": this.content.resultsBadgeColor,
        "--ww-data-grid_results-badge-borderRadius": this.content.resultsBadgeBorderRadius,
        "--ww-data-grid_results-badge-padding": this.content.resultsBadgePadding,
        // Rows-per-page vars
        "--ww-data-grid_rpp-label-color": this.content.rppLabelColor,
        "--ww-data-grid_rpp-select-backgroundColor": this.content.rppSelectBackgroundColor,
        "--ww-data-grid_rpp-select-color": this.content.rppSelectColor,
        "--ww-data-grid_rpp-select-borderColor": this.content.rppSelectBorderColor,
        "--ww-data-grid_rpp-select-borderRadius": this.content.rppSelectBorderRadius,
        // Footer vars
        "--ww-data-grid_footer-backgroundColor": this.content.footerBackgroundColor,
        "--ww-data-grid_footer-borderColor": this.content.footerBorderColor,
        "--ww-data-grid_footer-padding": this.content.footerPadding,
        "--ww-data-grid_footer-info-color": this.content.footerInfoColor,
        "--ww-data-grid_footer-info-fontSize": this.content.footerInfoFontSize,
        // Pagination button vars
        "--ww-data-grid_page-btn-color": this.content.pageBtnColor,
        "--ww-data-grid_page-btn-backgroundColor": this.content.pageBtnBackgroundColor,
        "--ww-data-grid_page-btn-borderColor": this.content.pageBtnBorderColor,
        "--ww-data-grid_page-btn-borderRadius": this.content.pageBtnBorderRadius,
        "--ww-data-grid_page-btn-active-color": this.content.pageBtnActiveColor,
        "--ww-data-grid_page-btn-active-backgroundColor": this.content.pageBtnActiveBackgroundColor,
        "--ww-data-grid_page-btn-size": this.content.pageBtnSize,
      };
    },
    theme() {
      return themeQuartz.withParams({
        headerBackgroundColor: this.content.headerBackgroundColor,
        headerTextColor: this.content.headerTextColor,
        headerFontSize: this.content.headerFontSize,
        headerFontFamily: this.content.headerFontFamily,
        headerFontWeight: this.content.headerFontWeight,
        headerHeight:
          this.content.headerHeightMode !== "auto" ? this.content.headerHeight : undefined,
        borderColor: this.content.borderColor,
        cellTextColor: this.content.cellColor,
        cellFontFamily: this.content.cellFontFamily,
        dataFontSize: this.content.cellFontSize,
        oddRowBackgroundColor: this.content.rowAlternateColor,
        backgroundColor: this.content.rowBackgroundColor,
        rowHoverColor: this.content.rowHoverColor,
        selectedRowBackgroundColor: this.content.selectedRowBackgroundColor,
        rowVerticalPaddingScale: this.content.rowVerticalPaddingScale || 1,
        menuBackgroundColor: this.content.menuBackgroundColor,
        menuTextColor: this.content.menuTextColor,
        columnHoverColor: this.content.columnHoverColor,
        foregroundColor: this.content.textColor,
        checkboxCheckedBackgroundColor: this.content.selectionCheckboxColor,
        rangeSelectionBorderColor: this.content.cellSelectionBorderColor,
        checkboxUncheckedBorderColor: this.content.checkboxUncheckedBorderColor,
        focusShadow: this.content.focusShadow?.length ? this.content.focusShadow : undefined,
        wrapperBorderRadius: this.content.wrapperBorderRadius,
      });
    },
    isEditing() {
      /* wwEditor:start */
      return this.wwEditorState.editMode === wwLib.wwEditorHelper.EDIT_MODES.EDITION;
      /* wwEditor:end */
      // eslint-disable-next-line no-unreachable
      return false;
    },
  },
  methods: {
    getRowId(params) {
      return this.resolveMappingFormula(this.content.idFormula, params.data);
    },
    onActionTrigger(event) {
      this.$emit("trigger-event", { name: "action", event });
    },
    onCellValueChanged(event) {
      this.$emit("trigger-event", {
        name: "cellValueChanged",
        event: {
          oldValue: event.oldValue,
          newValue: event.newValue,
          columnId: event.column.getColId(),
          row: event.data,
        },
      });
    },
    onRowClicked(event) {
      this.$emit("trigger-event", {
        name: "rowClicked",
        event: {
          row: event.data,
          id: event.node.id,
          index: event.node.sourceRowIndex,
          displayIndex: event.rowIndex,
        },
      });
    },
    resetFilters() {
      if (!this.gridApi) return;
      this.gridApi.setFilterModel(null);
    },
    resetSort() {
      if (!this.gridApi) return;
      this.gridApi.applyColumnState({ state: [], defaultState: { sort: null } });
    },
    deselectAll() {
      if (!this.gridApi) return;
      this.gridApi.deselectAll();
    },
    selectAll(mode) {
      if (!this.gridApi) return;
      if (this.content.rowSelection !== "multiple") {
        wwLib.logStore.warning("Select all will have no effect, as row selection is not set to multiple");
        return;
      }
      this.gridApi.selectAll(mode || this.content.selectAll || "all");
    },
    selectRow(rowId) {
      if (!this.gridApi) return;
      const rowNode = this.gridApi.getRowNode(rowId);
      if (rowNode) rowNode.setSelected(true);
    },
    deselectRow(rowId) {
      if (!this.gridApi) return;
      const rowNode = this.gridApi.getRowNode(rowId);
      if (rowNode) rowNode.setSelected(false);
    },
    getHeaderStyle(params) {
      const colDef = params.column?.getColDef();
      const col = this.content.columns.find(
        (c) => c.field === colDef?.field || (c.actionName && c.actionName === colDef?.colId)
      );
      const id = params.column?.getColId();
      const context = {
        id,
        name: colDef?.headerName || id,
        dataType: colDef?.cellDataType,
        type: col?.cellDataType === 'dateString' ? 'date' : (col?.cellDataType || 'auto'),
      };
      return {
        backgroundColor: this.resolveMappingFormula?.(this.content.dynamicHeaderBackgroundColor, context),
        color: this.resolveMappingFormula?.(this.content.dynamicHeaderTextColor, context),
        fontWeight: this.resolveMappingFormula?.(this.content.dynamicHeaderFontWeight, context),
        fontSize: this.resolveMappingFormula?.(this.content.dynamicHeaderFontSize, context),
        fontFamily: this.resolveMappingFormula?.(this.content.dynamicHeaderFontFamily, context),
      };
    },
    /* wwEditor:start */
    generateColumns() {
      this.$emit("update:content", {
        columns: this.rowData?.[0]
          ? Object.keys(this.rowData[0]).map((key) => ({ field: key, sortable: true, filter: true }))
          : [],
      });
    },
    getOnActionTestEvent() {
      const data = this.rowData;
      if (!data || !data[0]) throw new Error("No data found");
      return { actionName: "actionName", row: data[0], id: 0, index: 0, displayIndex: 0 };
    },
    getOnCellValueChangedTestEvent() {
      const data = this.rowData;
      if (!data || !data[0]) throw new Error("No data found");
      return { oldValue: "oldValue", newValue: "newValue", columnId: "columnId", row: data[0] };
    },
    getSelectionTestEvent() {
      const data = this.rowData;
      if (!data || !data[0]) throw new Error("No data found");
      return { row: data[0] };
    },
    getRowClickedTestEvent() {
      const data = this.rowData;
      if (!data || !data[0]) throw new Error("No data found");
      return { row: data[0], id: 0, index: 0, displayIndex: 0 };
    },
    getRowDraggedTestEvent() {
      const data = this.rowData;
      if (!data || !data[0]) throw new Error("No data found");
      return { row: data[0], id: 0, targetIndex: 1, rows: data };
    },
    getRowDragStartTestEvent() {
      const data = this.rowData;
      if (!data || !data[0]) throw new Error("No data found");
      return { row: data[0], id: 0 };
    },
    getColumnMovedTestEvent() {
      const data = this.columnDefs;
      if (!data || !data[0]) throw new Error("No data found");
      return { toIndex: 1, columnId: data[0].field, columnsOrder: data.map((col) => col.field) };
    },
    /* wwEditor:end */
  },
  /* wwEditor:start */
  watch: {
    columnDefs: {
      async handler() {
        if (this.wwEditorState?.boundProps?.columns) return;
        this.gridApi?.resetColumnState();
        if (this.wwEditorState.isACopy) return;
        const columnIndex = (this.rawContent.columns || []).findIndex(
          (col) => col?.cellDataType === "custom" && !col?.containerId
        );
        if (columnIndex === -1) return;
        const newColumns = [...this.rawContent.columns];
        let column = { ...newColumns[columnIndex] };
        column.containerId = await this.createElement("ww-flexbox", {
          _state: { name: `Cell ${column.headerName || column.field}` },
        });
        newColumns[columnIndex] = column;
        this.$emit("update:content:effect", { columns: newColumns });
      },
      deep: true,
    },
  },
  /* wwEditor:end */
};
</script>

<style scoped lang="scss">
.ww-datagrid {
  position: relative;
  display: flex;
  flex-direction: column;

  /* ── TOOLBAR ─────────────────────────────────────────────── */
  .ww-datagrid-toolbar {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: var(--ww-data-grid_toolbar-padding, 12px 16px);
    background-color: var(--ww-data-grid_toolbar-backgroundColor, transparent);
    border-bottom: 1px solid var(--ww-data-grid_toolbar-borderColor, transparent);
    gap: 12px;
    flex-wrap: wrap;

    &__left {
      display: flex;
      align-items: center;
      gap: 10px;
    }

    &__right {
      display: flex;
      align-items: center;
      gap: 8px;
    }
  }

  .ww-datagrid-results-label {
    color: var(--ww-data-grid_results-label-color, inherit);
    font-size: var(--ww-data-grid_results-label-fontSize, 14px);
    font-weight: var(--ww-data-grid_results-label-fontWeight, 600);
    font-family: var(--ww-data-grid_results-label-fontFamily, inherit);
  }

  .ww-datagrid-results-badge {
    display: inline-flex;
    align-items: center;
    background-color: var(--ww-data-grid_results-badge-backgroundColor, rgba(255,255,255,0.1));
    color: var(--ww-data-grid_results-badge-color, inherit);
    border-radius: var(--ww-data-grid_results-badge-borderRadius, 999px);
    padding: var(--ww-data-grid_results-badge-padding, 2px 10px);
    font-size: 13px;
    font-weight: 500;
    white-space: nowrap;
  }

  .ww-datagrid-rpp-label {
    color: var(--ww-data-grid_rpp-label-color, inherit);
    font-size: 13px;
    white-space: nowrap;
    user-select: none;
  }

  .ww-datagrid-rpp-select-wrapper {
    position: relative;
    display: inline-flex;
    align-items: center;
  }

  .ww-datagrid-rpp-select {
    appearance: none;
    -webkit-appearance: none;
    background-color: var(--ww-data-grid_rpp-select-backgroundColor, transparent);
    color: var(--ww-data-grid_rpp-select-color, inherit);
    border: 1px solid var(--ww-data-grid_rpp-select-borderColor, currentColor);
    border-radius: var(--ww-data-grid_rpp-select-borderRadius, 6px);
    padding: 4px 28px 4px 10px;
    font-size: 13px;
    cursor: pointer;
    outline: none;

    &:focus {
      outline: none;
    }
  }

  .ww-datagrid-rpp-chevron {
    position: absolute;
    right: 8px;
    width: 10px;
    height: 10px;
    pointer-events: none;
    color: var(--ww-data-grid_rpp-select-color, inherit);
  }

  /* ── FOOTER ──────────────────────────────────────────────── */
  .ww-datagrid-footer {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: var(--ww-data-grid_footer-padding, 12px 16px);
    background-color: var(--ww-data-grid_footer-backgroundColor, transparent);
    border-top: 1px solid var(--ww-data-grid_footer-borderColor, transparent);
    gap: 12px;
    flex-wrap: wrap;

    &__info {
      color: var(--ww-data-grid_footer-info-color, inherit);
      font-size: var(--ww-data-grid_footer-info-fontSize, 13px);
      white-space: nowrap;
    }

    &__pagination {
      display: flex;
      align-items: center;
      gap: 4px;
    }
  }

  .ww-datagrid-page-ellipsis {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: var(--ww-data-grid_page-btn-size, 32px);
    height: var(--ww-data-grid_page-btn-size, 32px);
    font-size: 13px;
    color: var(--ww-data-grid_page-btn-color, inherit);
    user-select: none;
  }

  .ww-datagrid-page-btn {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: var(--ww-data-grid_page-btn-size, 32px);
    height: var(--ww-data-grid_page-btn-size, 32px);
    border-radius: var(--ww-data-grid_page-btn-borderRadius, 6px);
    border: 1px solid var(--ww-data-grid_page-btn-borderColor, transparent);
    background-color: var(--ww-data-grid_page-btn-backgroundColor, transparent);
    color: var(--ww-data-grid_page-btn-color, inherit);
    font-size: 13px;
    font-weight: 500;
    cursor: pointer;
    transition: background-color 0.15s ease, color 0.15s ease;
    padding: 0;
    line-height: 1;

    svg {
      width: 6px;
      height: 10px;
      flex-shrink: 0;

      // |< >|  icons have a square 10×10 viewBox
      &.nav-icon--wide {
        width: 10px;
        height: 10px;
      }

      // << >>  icons have a wide 14×10 viewBox
      &.nav-icon--xwide {
        width: 14px;
        height: 10px;
      }
    }

    &.active {
      background-color: var(--ww-data-grid_page-btn-active-backgroundColor, #3b82f6);
      color: var(--ww-data-grid_page-btn-active-color, #fff);
      border-color: transparent;
    }

    &--nav {
      &:disabled {
        opacity: 0.35;
        cursor: not-allowed;
      }
    }

    &:not(:disabled):not(.active):hover {
      opacity: 0.75;
    }
  }

  /* ── ROW CURSOR ──────────────────────────────────────────── */
  &.rows-clickable {
    :deep(.ag-row) {
      cursor: pointer !important;
    }
  }

  /* ── EDITABLE COLUMN HEADER ICON ────────────────────────── */
  :deep(.ag-header-cell.-editable) {
    .ag-header-cell-text::after {
      content: '';
      display: inline-block;
      width: 11px;
      height: 11px;
      margin-left: 5px;
      flex-shrink: 0;
      vertical-align: middle;
      opacity: 0.6;
      background-size: contain;
      background-repeat: no-repeat;
      background-position: center;
      // Pencil icon as inline SVG data URI
      background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 12 12' fill='none'%3E%3Cpath d='M8.5 1.5L10.5 3.5L4 10H2V8L8.5 1.5Z' stroke='currentColor' stroke-width='1.2' stroke-linecap='round' stroke-linejoin='round'/%3E%3C/svg%3E");
      filter: invert(1);
    }
  }

  /* ── EXISTING CELL / HEADER STYLES ───────────────────────── */
  :deep(.ag-cell-wrapper),
  :deep(.ag-cell-value) {
    height: 100%;
  }
  :deep(.ag-header-cell) {
    &.-center .ag-header-cell-label { justify-content: center; }
    &.-right {
      .ag-header-cell-label { justify-content: flex-end; }
      .ag-header-cell-filter-button { margin-left: 4px; }
    }
    &.-left .ag-header-cell-label { justify-content: flex-start; }
  }
  :deep(.ag-cell) {
    .ag-cell-value { display: flex; }
    &.-right .ag-cell-value { justify-content: flex-end; }
    &.-center .ag-cell-value { justify-content: center; }
    &.-left .ag-cell-value { justify-content: flex-start; }
  }

  /* wwEditor:start */
  &.editing {
    &::before {
      content: "";
      position: absolute;
      inset: 0;
      display: block;
      pointer-events: initial;
      z-index: 10;
    }
  }
  /* wwEditor:end */
}
</style>
