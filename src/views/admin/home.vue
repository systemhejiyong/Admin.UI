<template>
  <section style="padding:20px 80px 0px 8px;">
    <el-timeline>
      <el-timeline-item timestamp="版本 1.1.0" placement="top">
        <el-card>
          <h4>升级 element-ui到 2.13.1 版本</h4>
          <h4>新增 文档管理界面，所见即所得编辑，支持自动保存</h4>
          <h4>新增 个人设置 - 头像上传，支持登录验证，上传配置支持热更新</h4>
          <h4>新增 首页选项卡支持卡片模式</h4>
          <h4>新增 登录日志和操作日志展开详情</h4>
          <h4>新增 分页组件、容器组件、MarkDown组件、布局-选项卡组件</h4>
          <h4>优化 布局样式全局修改为范围，导航菜单折叠后菜单项高度和图标间距不协调的问题</h4>
          <h4>优化 去除 slot="name" slot-scope="scope"语法，统一使用 v-slot:name="scope" 或 #name="{scope}"</h4>
          <h4>调整 所有分页界面使用容器组件布局，用户管理角色查询优化</h4>
          <h4>调整 移除布局视图滚动组件，实现容器组件表格满屏显示</h4>
          <h4>调整 将Element分页控件替换为Admin分页控件，小屏自动显示极简分页</h4>
          <p>小雪转中雪 提交于 2020-4-25</p>
        </el-card>
      </el-timeline-item>
      <el-timeline-item timestamp="版本 1.0.1" placement="top">
        <el-card>
          <h4>新增 日志管理-登录日志和操作日志</h4>
          <h4>优化 数据同步使用事物，数据同步SqlServer自增长处理。更新NuGet包到最新版本。</h4>
          <p>小雪转中雪 提交于 2020-3-28</p>
        </el-card>
      </el-timeline-item>
      <el-timeline-item timestamp="版本 1.0.0" placement="top">
        <el-card>
          <h4>新增 更新日志</h4>
          <h4>新增 缓存管理，手动清除缓存</h4>
          <h4>修复 接口管理-同步Api异常问题</h4>
          <p>小雪转中雪 提交于 2020-3-24</p>
        </el-card>
      </el-timeline-item>
    </el-timeline>
    <el-backtop target=".container .main" :visibility-height="200" />
  </section>
</template>

<script>
export default {
  name: 'Welcome',
  data() {
    return {

    }
  }
}
</script>

<style scoped>
h2,h3,h4,h5 {
    font-weight: 400;
    color: #1f2f3d;
    margin-top: 0px;
    margin-bottom: 10px;
}
p {
    font-size: 14px;
    color: #5e6d82;
    line-height: 1.5em;
    margin-bottom: 0px;
}
.el-timeline{
  padding-left: 15px;
}
</style>
