<template>
  <div class="ww-currency-cell-editor">
    <span class="ww-currency-cell-editor__symbol">$</span>
    <input
      ref="inputRef"
      type="text"
      :value="displayValue"
      inputmode="decimal"
      @input="onInput"
      @keydown="onKeydown"
      @blur="commitValue"
    />
  </div>
</template>

<script>
import { ref, computed, onMounted } from "vue";

export default {
  name: "CurrencyCellEditor",
  props: ["params"],
  setup(props) {
    const params = props.params;
    const inputRef = ref(null);

    // Strip anything that isn't a digit or decimal point
    const parseRaw = (val) => {
      const str = String(val ?? "").replace(/[^0-9.]/g, "");
      // Keep only first decimal point
      const parts = str.split(".");
      if (parts.length > 2) return parts[0] + "." + parts.slice(1).join("");
      return str;
    };

    const rawValue = ref(parseRaw(params.value));

    const displayValue = computed(() => rawValue.value);

    const onInput = (e) => {
      rawValue.value = parseRaw(e.target.value);
      e.target.value = rawValue.value;
    };

    const onKeydown = (e) => {
      if (e.key === "Enter") { commitValue(); params.stopEditing(); }
      if (e.key === "Escape") { params.stopEditing(true); }
    };

    const commitValue = () => {
      // Round to 2 decimal places on commit
      const num = parseFloat(rawValue.value);
      if (!isNaN(num)) rawValue.value = num.toFixed(2);
    };

    const getValue = () => {
      const num = parseFloat(rawValue.value);
      return isNaN(num) ? null : parseFloat(num.toFixed(2));
    };

    onMounted(() => {
      inputRef.value?.focus();
      inputRef.value?.select();
    });

    return { inputRef, displayValue, onInput, onKeydown, commitValue, getValue };
  },
};
</script>

<style scoped>
.ww-currency-cell-editor {
  display: flex;
  align-items: center;
  width: 100%;
  height: 100%;
  padding: 0 8px;
  box-sizing: border-box;
  gap: 2px;
}
.ww-currency-cell-editor__symbol {
  color: inherit;
  font-size: inherit;
  font-family: inherit;
  flex-shrink: 0;
}
.ww-currency-cell-editor input {
  flex: 1;
  min-width: 0;
  border: none;
  outline: none;
  background: transparent;
  color: inherit;
  font-size: inherit;
  font-family: inherit;
  padding: 0;
}
</style>
