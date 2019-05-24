<template>
  <div ref="wrap">
    <!-- 我的任务 -->
    <div class="my-task">
      <div class="my-task-title flex">
        <text class="f28 fw5 c0">我的任务</text>
        <text class="f24 c153 fw4 pl20 pt10 pb10" @click="mytaskMoreEvent">全部</text>
      </div>
      <div class="my-task-content" v-if="isShow">
        <div v-if='mytaskArr.length!=0' v-for="(item, index) in mytaskArr" :key='index' @click='mytaskUserEvent(item.id)'>
          <div class="flex-dr task-item flex-ac">
            <div v-if='item.marked == false' class="task-line task-level-one"></div>
            <!-- <div v-if='item.marked == true' class="task-line task-level-two"></div> -->
            <div v-if='item.marked == true' class="task-line task-level-three"></div>
            <div :class="[index == (mytaskArr.length-1)? 'border-no-bottom' : 'border-bottom']">
              <div class="task-text flex">
                <text class="f28 c0 fw4 prl27 lines1 task-width">{{item.name}}</text>
                <div class="task-time-width flex">
                  <text class="f24 c153 fw4">{{item.time}}</text>
                  <text class="f24 c153 fw4">{{item.week}}</text>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="no-content flex-ac flex-jc" v-if='mytaskArr.length==0'>
          <div class="flex-dr flex-jc">
            <bui-image src="/image/sleep.png" width="42px" height="39px"></bui-image>
            <text class="f26 c51 fw4 pl15 center-height">{{isError?'暂无任务':'加载失败'}}</text>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  const link = weex.requireModule("LinkModule");
  const dom = weex.requireModule('dom');
  export default {
    data() {
      return {
        mytaskArr: [],
        week: {
          '0': '周日',
          '1': '周一',
          '2': '周二',
          '3': '周三',
          '4': '周四',
          '5': '周五',
          '6': '周六',
        },
        isShow: false,
        isError: true
      }
    },
    methods: {
      mytaskMoreEvent() {
        link.launchLinkService(['[OpenApp] \n appCode=crm \n appUrl=LinkOl/Modular/other/taskList.html'], (res) => {},
        (err) => {});
      },
      mytaskUserEvent(id) {
        link.launchLinkService(['[OpenApp] \n appCode=crm \n appUrl=LinkOl/Modular/other/taskHome.html \n id='+ id], (res) => {},
        (err) => {
        });
      },
      getNowFormatDate(type, dat) {
        let date = new Date()
        if (dat) {
          date = dat
        }
        let year = date.getFullYear(),
          month = date.getMonth() + 1,
          strDate = date.getDate(),
          currentdate = ''
        if (month >= 1 && month <= 9) {
          month = "0" + month;
        }
        if (strDate >= 0 && strDate <= 9) {
          strDate = "0" + strDate;
        }
        if (type == 1) {
          currentdate = year + month + strDate;
        } else {
          currentdate = year + '/' + month + '/' + strDate;
        }
        if (dat) {
          currentdate = month + '-' + strDate
        }
        return currentdate;
      },
      getToken(success, error) {
        return new Promise((resolve, reject) => {
          link.getToken([], res => {
            resolve(res);
            success && success(res);
          }, err => {
            reject(err);
            error && error(err);
          });
        });
      },
      getTaskData(url, data, token, success, error) {
        return new Promise((resolve, reject) => {
          this.$get({
            url: url,
            headers: {
              'Authorization': 'Bearer ' + token
            },
            data: data
          }).then((res) => {
            resolve(res);
            success && success(res);
          }).catch((reason) => {
            reject(reason);
            error && error(reason);
          })
        });
      },
      getTask(startTime, endTime) {
        this.getToken((token) => {
          link.getServerConfigs([], (params) => {
            let whereFilter = '(UNIX_TIMESTAMP(startTime) <= ' + endTime +
              ' OR startTime IS NULL) AND (UNIX_TIMESTAMP(endTime)>= ' + startTime + ' OR endTime IS NULL)'
            let objData = {
              whereFilter,
              entityName: 'ExtendWorktask',
              searchType: 0,
              keyWord: '',
              currentStatus: 0,
              orderBy: 'IF(ISNULL(IFNULL(startTime,endTime)),1,0),IF(ISNULL(startTime),endTime,startTime),IF(ISNULL(endTime),1,0),endTime',
              marked: false,
              parentId: '',
              endTime: '',
              startTime: '',
            }

            let reqObjData = JSON.parse(JSON.stringify(objData))
            reqObjData['whereFilter'] = '(UNIX_TIMESTAMP(startTime) <= ' + endTime + ' OR startTime IS NULL)'
            reqObjData['marked'] = true
            // 内部数据
            let promiseOne = this.getTaskData(params.uamUri + '/webCommon/getList', objData, token.accessToken, (res) =>
            {}, (err) => {})
            // 星标数据
            let promiseTwo = this.getTaskData(params.uamUri + '/webCommon/getList', reqObjData, token.accessToken,
              (res) => {}, (err) => {})
            Promise.all([
              promiseOne, promiseTwo
            ]).then(() => {
              this.isError = true
              this.isShow = true
              this.broadcastWidgetHeight()
              let task = []
              if (promiseOne._v.success == true && promiseTwo._v.success == true) {
                if (JSON.stringify(promiseOne._v.data) != '[]') {
                  task = promiseOne._v.data
                }
                if (JSON.stringify(promiseTwo._v.data) != '[]') {
                  task = task.concat(promiseTwo._v.data)
                }

                let taskObj = {}
                let taskItem = []
                for (let index = 0; index < task.length; index++) {
                  let newDate = new Date(Date.parse(task[index].createdTimeDisplayValue.replace(/\-/g,
                    "/")));
                  let getMonthWeek = this.getNowFormatDate(1, newDate)
                  let getDay = newDate.getDay()
                  taskObj['time'] = getMonthWeek
                  taskObj['week'] = this.week[getDay]
                  taskObj['name'] = task[index].name
                  taskObj['id'] = task[index].id
                  taskObj['marked'] = task[index].marked
                  taskItem.push(JSON.parse(JSON.stringify(taskObj)))
                }
                this.mytaskArr = taskItem
              }
            }).catch(() => {
              this.error()
            })
          }, () => {
            this.error()
          });
        }, (err) => {
          this.error()
        })
      },
      error() {
        this.isShow = true
        this.isError = false
        this.broadcastWidgetHeight()
      },
      broadcastWidgetHeight() {
        let _params = this.$getPageParams();
        setTimeout(() => {
          dom.getComponentRect(this.$refs.wrap, (ret) => {
            var channel = new BroadcastChannel('WidgetsMessage')
            channel.postMessage({
              widgetHeight: ret.size.height,
              id: _params.id
            });
            channel.close();
          });
        }, 100)
      }
    },
    mounted() {
      let searchTime = this.getNowFormatDate(2, null);
      let start = searchTime + ' 00:00:00'
      let startDate = new Date(start)
      let end = searchTime + ' 23:59:59'
      let endDate = new Date(end)
      this.getTask(startDate.getTime(), endDate.getTime())
    }
  }
</script>

<style lang="css" src="../css/common.css"></style>
<style>
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

  .task-time-width {
    width: 137px;
  }

  .task-text {
    width: 702px;
    /* border-bottom: 1px solid rgba(232, 232, 232, 1); */
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

  .border-bottom {
    border-bottom: 1px solid #E8E8E8;
  }

  .border-no-bottom {
    border-bottom: 1px solid #fff;
  }

  .no-content {
    height: 166px;
    width: 750px
  }

  .center-height {
    line-height: 40px;
  }
</style>