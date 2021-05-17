<template>
  <div style="margin: 20px;">
    <el-input placeholder="请输入内容" suffix-icon="el-icon-date" v-model="input1"></el-input>
  </div>
</template>

<script>
export default {
  data() {
    return {
      input1: 'test',
    };
  },
};
</script>

<style lang="scss" scoped></style>
