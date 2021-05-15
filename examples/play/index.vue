<template>
  <div style="margin: 20px;">
    <el-checkbox v-model="checked">备选项</el-checkbox>
    <el-checkbox-group v-model="checkList">
      <el-checkbox label="复选框 A"></el-checkbox>
      <el-checkbox label="复选框 B"></el-checkbox>
      <el-checkbox label="复选框 C"></el-checkbox>
      <el-checkbox label="禁用" disabled></el-checkbox>
      <el-checkbox label="选中且禁用" disabled></el-checkbox>
    </el-checkbox-group>
  </div>
</template>

<script>
export default {
  data() {
    return {
      checked: true,
      checkList: ['选中且禁用', '复选框 A'],
    };
  },
};
</script>

<style lang="scss" scoped></style>
