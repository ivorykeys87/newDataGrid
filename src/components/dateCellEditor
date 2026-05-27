<template>
  <input
    ref="inputRef"
    class="ww-date-cell-editor"
    type="text"
    :value="displayValue"
    maxlength="10"
    placeholder="MM-DD-YYYY"
    @keydown="onKeydown"
    @blur="commitValue"
  />
</template>

<script>
import { ref, onMounted } from "vue";

export default {
  name: "DateCellEditor",
  props: ["params"],
  setup(props) {
    const params = props.params;
    const inputRef = ref(null);

    // Accept MM-DD-YYYY or MM/DD/YYYY or raw digits, normalise to MM-DD-YYYY
    const normalise = (val) => {
      const digits = String(val ?? "").replace(/\D/g, "").slice(0, 8);
      return format(digits);
    };

    const format = (digits) => {
      if (digits.length <= 2) return digits;
      if (digits.length <= 4) return `${digits.slice(0, 2)}-${digits.slice(2)}`;
      return `${digits.slice(0, 2)}-${digits.slice(2, 4)}-${digits.slice(4)}`;
    };

    const displayValue = ref(normalise(params.value));

    const onKeydown = (e) => {
      const input = e.target;
      const key = e.key;

      // Always allow: navigation, backspace, delete, tab, enter, escape
      if (["ArrowLeft","ArrowRight","ArrowUp","ArrowDown","Tab","Backspace","Delete","Enter","Escape"].includes(key)) {
        if (key === "Enter") { commitValue(); params.stopEditing(); }
        if (key === "Escape") { params.stopEditing(true); }
        return;
      }

      // Only allow digits
      if (!/^\d$/.test(key)) {
        e.preventDefault();
        return;
      }

      e.preventDefault();

      // Build updated digit string from current display value
      const currentDigits = input.value.replace(/\D/g, "");
      if (currentDigits.length >= 8) return; // max 8 digits

      const newDigits = currentDigits + key;
      const formatted = format(newDigits);
      input.value = formatted;
      displayValue.value = formatted;

      // Move cursor to end
      const pos = formatted.length;
      input.setSelectionRange(pos, pos);
    };

    const commitValue = () => {
      // no-op — getValue handles it
    };

    const getValue = () => {
      // Return the formatted MM-DD-YYYY string as-is
      return displayValue.value || null;
    };

    onMounted(() => {
      inputRef.value?.focus();
      // Place cursor at end
      const len = displayValue.value.length;
      inputRef.value?.setSelectionRange(len, len);
    });

    return { inputRef, displayValue, onKeydown, commitValue, getValue };
  },
};
</script>

<style scoped>
.ww-date-cell-editor {
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
