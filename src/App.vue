<template>
  <div>
    <Table :tableData="tableData" />
  </div>
</template>

<script>
import Table from "./components/Table.vue";

export default {
  name: "App",
  components: {
    Table,
  }
};
</script>