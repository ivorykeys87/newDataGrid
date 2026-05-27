<template>
  <select
    ref="selectRef"
    class="ww-dropdown-cell-editor"
    :value="currentValue"
    @change="onChange"
    @keydown="onKeydown"
    @blur="params.stopEditing()"
  >
    <option
      v-for="opt in options"
      :key="opt.value ?? opt"
      :value="opt.value ?? opt"
    >
      {{ opt.label ?? opt }}
    </option>
  </select>
</template>

<script>
import { ref, onMounted } from "vue";

export default {
  name: "DropdownCellEditor",
  props: ["params"],
  setup(props) {
    const params = props.params;
    const selectRef = ref(null);

    // editorOptions can be:
    // - array of strings: ["Active", "Inactive"]
    // - array of objects: [{ value: "a", label: "Active" }]
    const options = Array.isArray(params.editorOptions) ? params.editorOptions : [];

    const currentValue = ref(params.value ?? "");

    const onChange = (e) => {
      currentValue.value = e.target.value;
      params.stopEditing();
    };

    const onKeydown = (e) => {
      if (e.key === "Escape") params.stopEditing(true);
    };

    const getValue = () => currentValue.value;

    onMounted(() => {
      selectRef.value?.focus();
    });

    return { selectRef, options, currentValue, onChange, onKeydown, getValue };
  },
};
</script>

<style scoped>
.ww-dropdown-cell-editor {
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
  cursor: pointer;
}
</style>
