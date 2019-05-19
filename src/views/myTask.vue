<template>
  <div class="main">
    <!-- 我的任务 -->
    <div class="my-task bra">
      <div class="my-task-title flex">
        <text class="f28 fw5 c0">我的任务</text>
        <text class="f24 c153 fw4 pl20 pt10 pb10" @click="mytaskMoreEvent">全部</text>
      </div>
      <div class="my-task-content">
        <div class="" v-for="(item, index) in mytaskArr" :key='index' @click='mytaskUserEvent'>
          <div class="flex-dr task-item flex-ac">
            <div v-if='item.level == 1' class="task-line task-level-one"></div>
            <div v-if='item.level == 2' class="task-line task-level-two"></div>
            <div v-if='item.level == 3' class="task-line task-level-three"></div>
            <div class="task-text flex">
              <text class="f28 c0 fw4 prl27 lines1 task-width">门户小部件原型设计111111111111111111111111111111111111111</text>
              <text class="f24 c153 fw4">05-20 周一</text>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        mytaskArr: [{
          level: 1
        }, {
          level: 2
        }, {
          level: 3
        }, {
        level: 3
        }, {
        level: 3
        }]
      }
    },
    methods: {
      // 更多
      mytaskMoreEvent() {
        this.$alert()
      },
      // 常用联系人
      mytaskUserEvent() {
        this.$alert()
      }
    }
  }
</script>

<style lang="css" src="../css/common.css"></style>
<style>
  .main {
    flex: 1;
    padding-top: 100px;
    background-color: #666;
  }

  .my-task {
    background-color: #fff;
  }

  .my-task-title {
    height: 40px;
    margin: 18px 23px 20px 25px;
  }

  .my-task-content {
    padding: 0 24px 19px 0;
  }

  .task-item {
    height: 95px;
  }

  .task-line {
    width: 5px;
    height: 92px;
    margin-right: 19px;
  }

  .task-width {
    width: 550px;
  }

  .task-text {
    width: 702px;
    border-bottom: 1px solid rgba(232, 232, 232, 1);
  }

  .task-level-one {
    background: rgba(254, 91, 91, 1);
  }

  .task-level-two {
    background: rgba(255, 148, 59, 1);
  }

  .task-level-three {
    background: rgba(53, 196, 134, 1);
  }
</style>