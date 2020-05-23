<template>
    <div>
        <el-button type="info" @click="quitLogin">退出登录</el-button>
    </div>
</template>

<script>
export default {
  methods: {
    quitLogin () {
      window.sessionStorage.clear()
      this.$router.push('/login')
      this.$message.success({ message: '退出成功！😊', center: true })
    }
  }
}
</script>

<style lang="less" scoped>

</style>
