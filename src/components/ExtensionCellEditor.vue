<template>
  <input
    ref="inputRef"
    class="ww-extension-cell-editor"
    type="text"
    :value="currentValue"
    inputmode="numeric"
    @keydown="onKeydown"
    @blur="params.stopEditing()"
  />
</template>

<script>
import { ref, onMounted } from "vue";

export default {
  name: "ExtensionCellEditor",
  props: ["params"],
  setup(props) {
    const params = props.params;
    const inputRef = ref(null);

    const digitsOnly = (val) => String(val ?? "").replace(/\D/g, "");

    const currentValue = ref(digitsOnly(params.value));

    const onKeydown = (e) => {
      if (e.key === "Enter") { params.stopEditing(); }
      if (e.key === "Escape") { params.stopEditing(true); }

      // Allow: navigation, backspace, delete, tab
      if (["ArrowLeft","ArrowRight","ArrowUp","ArrowDown","Tab","Backspace","Delete"].includes(e.key)) return;

      // Block anything that isn't a digit
      if (!/^\d$/.test(e.key)) {
        e.preventDefault();
      }
    };

    const getValue = () => digitsOnly(inputRef.value?.value ?? currentValue.value);

    onMounted(() => {
      inputRef.value?.focus();
      inputRef.value?.select();
    });

    return { inputRef, currentValue, onKeydown, getValue };
  },
};
</script>

<style scoped>
.ww-extension-cell-editor {
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
