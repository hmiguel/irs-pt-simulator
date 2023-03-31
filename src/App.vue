<template>
  <div>
    <div class="github-link">
      <a href="https://github.com/hmiguel/irs-pt-simulator" target="_blank">View on GitHub</a>
    </div>
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