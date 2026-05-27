<template>
  <input
    ref="inputRef"
    class="ww-phone-cell-editor"
    type="text"
    :value="displayValue"
    maxlength="14"
    placeholder="(___) ___-____"
    @input="onInput"
    @keydown="onKeydown"
    @blur="commitValue"
  />
</template>

<script>
import { ref, computed, onMounted } from "vue";

export default {
  name: "PhoneCellEditor",

  setup(props) {
    // AG Grid passes the initial cell value via props.params
    const params = props.params;
    const inputRef = ref(null);

    // Strip everything except digits from any incoming value
    const digitsOnly = (val) => String(val ?? "").replace(/\D/g, "").slice(0, 10);

    const rawDigits = ref(digitsOnly(params.value));

    // Format raw digits → (305) 123-4567
    const format = (digits) => {
      if (!digits) return "";
      const d = digits.slice(0, 10);
      if (d.length <= 3) return `(${d}`;
      if (d.length <= 6) return `(${d.slice(0, 3)}) ${d.slice(3)}`;
      return `(${d.slice(0, 3)}) ${d.slice(3, 6)}-${d.slice(6)}`;
    };

    const displayValue = computed(() => format(rawDigits.value));

    const onInput = (e) => {
      rawDigits.value = digitsOnly(e.target.value);
      // Keep cursor at end after reformat
      const input = e.target;
      const formatted = format(rawDigits.value);
      input.value = formatted;
      const pos = formatted.length;
      input.setSelectionRange(pos, pos);
    };

    const onKeydown = (e) => {
      if (e.key === "Enter") {
        commitValue();
        params.stopEditing();
      }
      if (e.key === "Escape") {
        params.stopEditing(true); // true = cancel
      }
    };

    const commitValue = () => {
      // getValue() is called by AG Grid — we store raw digits
    };

    // AG Grid calls this to get the committed value
    const getValue = () => rawDigits.value;

    // Focus the input when the editor mounts
    onMounted(() => {
      inputRef.value?.focus();
      inputRef.value?.select();
    });

    return { inputRef, displayValue, onInput, onKeydown, commitValue, getValue };
  },

  // AG Grid requires props.params
  props: ["params"],
};
</script>

<style scoped>
.ww-phone-cell-editor {
  width: 100%;
  height: 100%;
  border: none;
  outline: none;
  background: transparent;
  color: inherit;
  font-size: inherit;
  font-family: inherit;
  padding: 0 8px;
  box-sizing: border-box;
}
</style>
