<template>
    <div ref="wrap">
        <!-- 我的任务 -->
        <div class="my-task">
            <div class="my-task-title flex">
                <text class="f28 fw5 c0">{{i18n.MyMission}}</text>
                <text class="f24 c153 fw4 pl20 pt10 pb10" @click="mytaskMoreEvent">{{i18n.All}}</text>
            </div>
            <div class="my-task-content" v-if="isShow">
                <div v-if='mytaskArr.length!=0' v-for="(item, index) in mytaskArr" :key='index'
                    @click='mytaskUserEvent(item.id)'>
                    <div class="flex-dr task-item flex-ac">
                        <div v-if='item.marked == true' class="task-line task-level-one"></div>
                        <div v-if='item.marked == false' class="task-line task-level-three"></div>
                        <div class="flex1"
                            :class="[index == (mytaskArr.length-1)? 'border-no-bottom' : 'border-bottom']">
                            <div class="flex-dr flex-ac flex-sb ptb27">
                                <text class="f28 c0 fw4 lines1 flex1">{{item.name}}</text>
                                <text class="f24 c153 fw4 pl20">{{item.showtime}}</text>
                                <!-- <div class="task-time-width flex flex-dr">
                                    <text class="f24 c153 fw4">{{item.time}}</text>
                                    <text class="f24 c153 fw4">{{item.week}}</text>
                                </div> -->
                            </div>
                        </div>
                    </div>
                </div>
                <div class="no-content flex1  flex-ac flex-jc" v-if='mytaskArr.length==0'>
                    <div class="flex-dr flex-jc">
                        <bui-image src="/image/sleep.png" width="42px" height="39px"></bui-image>
                        <text class="f26 c51 fw4 pl15 center-height">{{isError?i18n.NoneData:i18n.ErrorLoadData}}</text>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    const link = weex.requireModule("LinkModule");
    const linkapi = require('linkapi');
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
                isError: true,
                channel: new BroadcastChannel('WidgetsMessage'),
                i18n: ''
            }
        },
        methods: {
            mytaskMoreEvent() {
                link.launchLinkService(['[OpenApp] \n appCode=crm \n appUrl=LinkOl/Modular/other/taskList.html'], (res) => { },
                    (err) => { });
            },
            mytaskUserEvent(id) {
                link.launchLinkService(['[OpenApp] \n appCode=crm \n appUrl=LinkOl/Modular/other/taskHome.html \n id=' + id], (
                    res) => { },
                    (err) => { });
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
            getTask(startTime, endTime) {
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

                    let index = 0;
                    let promiseOne;
                    let promiseTwo;

                    // 内部数据
                    linkapi.get({
                        url: params.specialUri + '/webCommon/getList',
                        data: objData
                    }).then((res) => {
                        if (res.success == true) {
                            index++
                            promiseOne = res
                            if (index == 2) {
                                this.getTaskData(promiseOne, promiseTwo)
                            }
                        }
                    }, () => {
                        this.error()
                    })
                    // 星标数据
                    linkapi.get({
                        url: params.specialUri + '/webCommon/getList',
                        data: reqObjData
                    }).then((res) => {
                        if (res.success == true) {
                            index++
                            promiseTwo = res
                            if (index == 2) {
                                this.getTaskData(promiseOne, promiseTwo)
                            }
                        }
                    }, () => {
                        this.error()
                    })
                }, () => {
                    this.error()
                });
            },
            getTaskData(promiseOne, promiseTwo) {
                this.isError = true
                this.isShow = true
                this.broadcastWidgetHeight()
                let task = []
                if (JSON.stringify(promiseOne.data) != '[]') {
                    task = promiseOne.data
                }
                if (JSON.stringify(promiseTwo.data) != '[]') {
                    task = task.concat(promiseTwo.data)
                }
                let taskObj = {}
                let taskItem = []
                for (let index = 0; index < task.length; index++) {
                    let newDate = new Date(Date.parse(task[index].createdTimeDisplayValue.replace(/\-/g,
                        "/")));
                    // let getMonthWeek = this.getNowFormatDate(1, newDate)
                    let getDay = newDate.getDay()
                    // taskObj['time'] = getMonthWeek
                    // taskObj['week'] = this.week[getDay]
                    taskObj['showtime'] = this.formatStartEndTime(task[index])
                    taskObj['name'] = task[index].name
                    taskObj['id'] = task[index].id
                    taskObj['marked'] = task[index].marked
                    taskItem.push(JSON.parse(JSON.stringify(taskObj)))
                }
                this.mytaskArr = taskItem
            },
            format(ts, fmt) {
                if (!ts) return '';
                if (!fmt) fmt = 'yyyy-MM-dd hh:mm:ss'
                var dt = new Date(ts)
                var o = {
                    'M+': dt.getMonth() + 1, // 月份
                    'd+': dt.getDate(), // 日
                    'h+': dt.getHours(), // 小时
                    'm+': dt.getMinutes(), // 分
                    's+': dt.getSeconds(), // 秒
                    'q+': Math.floor((dt.getMonth() + 3) / 3), // 季度
                    'S': dt.getMilliseconds() // 毫秒
                }
                if (/(y+)/.test(fmt)) fmt = fmt.replace(RegExp.$1, (dt.getFullYear() + '').substr(4 - RegExp.$1.length))
                for (var k in o) {
                    if (new RegExp('(' + k + ')').test(fmt)) {
                        fmt = fmt.replace(RegExp.$1,
                            (RegExp.$1.length === 1) ? (o[k]) : (('00' + o[k]).substr(('' + o[k]).length)))
                    }
                }
                return fmt
            },
            formatStartEndTime(data) {
                var tYear = new Date().getFullYear();
                var sDate,
                    eDate,
                    sf,
                    ef;
                if (data.startTime) {
                    sDate = new Date(data.startTime);
                    sf = data.isAllDay ? (sDate.getFullYear() == tYear ? 'MM-dd' : 'yyyy-MM-dd') : (sDate.getFullYear() == tYear ? 'MM-dd hh:mm' : 'yyyy-MM-dd hh:mm');
                    sDate = this.format(sDate, sf);
                }
                if (data.endTime) {
                    eDate = new Date(data.endTime);
                    ef = data.isAllDay ? (eDate.getFullYear() == tYear ? 'MM-dd' : 'yyyy-MM-dd') : (eDate.getFullYear() == tYear ? 'MM-dd hh:mm' : 'yyyy-MM-dd hh:mm');
                    eDate = this.format(eDate, ef);
                }
                if (sDate && eDate) {
                    data.showTime = sDate + '~' + eDate;
                } else if (sDate && !eDate) {
                    data.showTime = this.i18n.Date_Begin + sDate;
                } else if (!sDate && eDate) {
                    data.showTime = this.i18n.Date_End + eDate;
                } else {
                    data.showTime = '';
                }
                if (data.isAllDay) data.showTime += ' ' + this.i18n.Date_ALLDay;
                return data.showTime ? data.showTime : ''
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
                        this.channel.postMessage({
                            widgetHeight: ret.size.height,
                            id: _params.id
                        });
                    });
                }, 200)
            },
            getData() {
                let searchTime = this.getNowFormatDate(2, null);
                let start = searchTime + ' 00:00:00'
                let startDate = new Date(start)
                let end = searchTime + ' 23:59:59'
                let endDate = new Date(end)
                this.getTask(startDate.getTime() / 1000, endDate.getTime() / 1000)
            }
        },
        created() {
            this.$fixViewport();
            linkapi.getLanguage((res) => {
                this.i18n = this.$window[res]
            })
        },
        mounted() {
            this.channel.onmessage = (event) => {
                if (event.data.action === 'RefreshData') {
                    this.getData()
                }
            }
            this.getData()
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

    .task-time-width {
        width: 220px;
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
        border-bottom: 1px solid #e8e8e8;
    }

    .border-no-bottom {
        border-bottom: 1px solid #fff;
    }

    .no-content {
        height: 166px;
    }

    .center-height {
        line-height: 40px;
    }
</style>