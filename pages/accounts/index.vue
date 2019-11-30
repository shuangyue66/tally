<template>
	<view class="content">
        <view class="contentHead">
            <view class="contentHead-box">
                <view class="contentHead-v">
                    <view>
                        <view class="head-prefix">总收入￥</view>
                        <view>{{addIncomeData}}</view>
                    </view>
                    <view style="margin-top: 10rpx;">
                        <view class="head-prefix">月收入￥</view>
                        <view>{{monthIncomeData}}</view>
                    </view>
                </view>
                <view class="contentHead-v">
                    <view>
                        <view class="head-prefix">总支出￥</view>
                        <view v-if="addExpendData == 0">{{addExpendData}}</view>
                        <view v-else class="head-prefix-r">{{'-'+ addExpendData}}</view>
                    </view>
                    <view style="margin-top: 10rpx;">
                        <view class="head-prefix">月支出￥</view>
                        <view v-if="monthExpendData == 0">{{monthExpendData}}</view>
                        <view v-else class="head-prefix-r">{{'-'+monthExpendData}}</view>
                    </view>
                </view>
                <view class="contentHead-v">
                    <view class="head-prefix">总余额￥</view>
                    <view>{{addBalanceData}}</view>
                </view>
            </view>
        </view>
        <view>
		<uni-calendar 
            :insert="true"
            :lunar="true" 
            :start-date="'2019-1-1'"
            :end-date="'2019-5-20'"
            @change="change"
            @month-switch="monthSwitch"
        />
        </view>
        <view v-if="showDay">
            <view class="todayView">
                <view class="todayView-l">
                    <view v-if="incomeData == 0" class="todayView-v">本日收入￥{{ incomeData }}</view>
                    <view v-else class="todayView-v" >
                        本日收入￥
                        <text class="greenStyle">{{ '+' +incomeData }}</text>
                    </view>
                    <view v-if="expendData == 0"
                        class="todayView-v">本日支出￥{{ expendData }}</view>
                    <view v-else class="todayView-v">
                        本日支出￥
                        <text class="todayView-v-prefix">{{'-'+ expendData }}</text>
                    </view>
                </view>
                <view class="todayView-r">
                    <button @click="onForm" type="primary" plain="true">添加</button>
                </view>
            </view>
            <view class="inventoryBox"
                    v-for="(item, index) in nowList"
                    :key="index">
                    <view class="inventoryBox-content">
                        <view class="inventoryBox-l">
                            <view class="inventoryItem">{{ item.picker }}</view>
                            <view class="inventoryItem">{{ item.radio }}</view>
                            <view v-if="item.radio === '收入'"
                                class="greenStyle inventoryItem">{{ '+'+ item.input }}</view>
                            <view v-else class="redStyle inventoryItem">{{ '-' + item.input }}</view>
                        </view>
                        <view class="inventoryBox-z"></view>
                        <view class="inventoryBox-r">
                            <view v-if="item.ramarks">备注:{{ item.ramarks }}</view>
                            <view v-else></view>
                        </view>
                    </view>
            </view>
        </view>
        <view v-else class="brnStyle">
            <button class="brnStyle-b" @click="onForm">添加</button>
        </view>
        <view>
            <uni-popup 
                ref="popup" 
                type="center"
                :maskClick="false">
                <view class="formBox">
                    <view>
                        <view class="formView-t">时间:</view>
                        <view>
                            <picker mode="date"
                                    :value="formDetail.picker"
                                    :start="startDate"
                                    :end="endDate"
                                    @change="bindDateChange">
                                <view>{{date}}</view>
                            </picker>
                        </view>
                    </view>
                    <view class="formView">
                        <view class="formView-t">方式:</view>
                        <radio-group @change="formRadio"
                                    :value="formDetail.radio"
                                    >
                            <label>
                                <radio value="radio1" :checked="radioData.radio1"/><text>收入</text>
                            </label>
                            <label style="margin-left: 20rpx;">
                                <radio value="radio2" :checked="radioData.radio2"/><text>支出</text>
                            </label>
                        </radio-group>
                    </view>
                    <view class="formView">
                        <view class="formView-t">金额:</view>
                        <input type="digit"
                                class="formView-input"
                                :value="formDetail.input"
                                @input="formInput"
                                maxlength='12'
                                placeholder="请输入金额，小数点最多两位" />
                    </view>
                    <view class="formView">
                        <view class="formView-t">备注:</view>
                        <textarea :value="formDetail.ramarks" 
                                @input="formRamarks"
                                class="formView-text"
                                placeholder="请输入备注"
                                maxlength="100"
                                />
                    </view>
                    <view class="formBrn">
                        <button class="brn"
                                type="warn"
                                plain="true"
                                ref="resetB"
                                @click="formReset">取消</button>
                        <button class="brn"
                                type="primary" 
                                plain="true"
                                @click="formSubmit">确认</button>
                    </view>
                </view>
            </uni-popup>
        </view>
	</view>
</template>

<script>
    import {uniCalendar} from "@/components/uni-calendar/uni-calendar"
    import uniPopup from "@/components/uni-popup/uni-popup.vue"
    var graceChecker = require("../../js_sdk/graceui-dataChecker/graceChecker.js")
	export default {
        components: {
            uniCalendar,
            uniPopup
        },
		data() {
            const currentDate = this.getDate({
                format: true
            })
			return {
                formDetail: {
                    picker: '', //时间
                    radio: '', //方式
                    input: '', //金额
                    ramarks: '', //备注 
                },
                timeList: {
                    nowtime: '', // 今天日期(年-月-日)
                    cyear: '', // 年
                    cmonth: '', // 月
                    cdate: '', // 日
                },
                YMtime: '', // 年月
                addDateList: {}, // 总日期金额数组
                addIncomeData: 0, //总收入数据
                addExpendData: 0, // 总支出数据
                addBalanceData: 0, // 总余额数据
                monthIncomeData: 0, // 月收入数据
                monthExpendData: 0, // 月支出数据
                incomeData: 0, // 本日收入数据
                expendData: 0, // 本日支出数据
                nowList: [], // 今天的数据
                date: currentDate,
                showDay: false,
                radioData: {
                    radio1: false,
                    radio2: false
                }
			}
		},
		onLoad() {
            let newList = uni.getStorageSync('list');
            let date = new Date
            let Y = date.getFullYear()
            let M = date.getMonth() + 1
            let D = date.getDate();
            let newtime = {
                nowtime: `${Y}-${M}-${D}`,
                cyear: Y,
                cmonth: M,
                cdate: D
            }
            this.formDetail.picker = `${Y}-${M}-${D}`
            this.YMtime = `${Y}-${M}`
            this.timeList = {
                ...newtime
            }
            if (newList) {
                this.showDay = true
                this.addDateList = newList
                this.addCalculate(newList) //总数据
                this.monthCalculate(newList, Y, M) //月数据
                this.todayCalculate(newList, Y, M, D) //本日数据
            }
        },
        computed: {
            startDate() {
                console.log(this.getDate('start'), '11111')
            },
            endDate() {
                return this.getDate('end');
            }
        },
		methods: {
            // 点击日期触发
			change(e) {
                // let year = e.year
                // let month = e.month
                // let day = e.date
                // let nowtime = `${year}-${month}-${day}`
                let newtime = {
                    nowtime: `${e.year}-${e.month}-${e.date}`,
                    cyear: e.year,
                    cmonth: e.month,
                    cdate: e.date
                }
                let YM = `${e.year}-${e.month}`
                this.date = `${e.year}-${e.month}-${e.date}`
                this.formDetail.picker = `${e.year}-${e.month}-${e.date}`
                this.YMtime = YM 
                this.showDay = true
                this.timeList = {
                   ...newtime
                }
                let add = this.addDateList
                this.todayCalculate(add, newtime.cyear, newtime.cmonth, newtime.cdate)
            },
            // 更换月份触发
            monthSwitch(e) {
                let year = e.year
                let month = e.month
                if (Number(month) < 9) {
                    month = `0${month}`
                }
                let YM = `${year}-${month}`
                let add = this.addDateList
                if (YM === this.YMtime) {
                    // this.showDay = true
                    this.todayCalculate(add, year, month, this.timeList.cdate)
                } else {
                    this.showDay = false
                }
                this.timeList = {
                    ...this.timeList,
                    cyear: year,
                    cmonth: month
                }
                this.monthCalculate(add, year, month)
            },
            onForm(e) {
                this.$refs.popup.open()
            },
            // 表单时间
            bindDateChange: function(e) {
                this.date = e.detail.value
                this.formDetail.picker = e.detail.value
            },
            // 表单方式
            formRadio(e) {
                if (e.detail.value === 'radio1') {
                    this.radioData.radio1 = true
                } else {
                    this.radioData.radio2 = true
                }
                this.formDetail.radio = e.detail.value
            },
            // 表单金额
            formInput(e) {
                this.formDetail.input = e.detail.value
            },
            // 表单备注
            formRamarks(e) {
                this.formDetail.ramarks = e.detail.value
            },
            // 表单确认
            formSubmit(e) {
                let formParams = {...this.formDetail} // 表单参数
                let rule = [
                    {name:"picker", checkType: "notnull", checkRule:"",  errorMsg:"请选择日期"},
                    {name:"radio", checkType: "in", checkRule:"radio1,radio2",  errorMsg:"请选择方式"},
                    {name:"input", checkType: "therge",  errorMsg:"请输入金额"}
                ];
                let checkRes = graceChecker.check(formParams, rule);
                if(checkRes){
                    let time = formParams.picker // 日期
                    let cyear = time.split('-')[0] // 年份
                    let cmonth = time.split('-')[1] // 月份
                    let cdate = time.split('-')[2] // 几号
                    let list = {...this.addDateList} // 总数据

                    let dataParams = {}                    

                    if (formParams.radio === 'radio1') {
                        dataParams = {
                            ...formParams,
                            radio: '收入',
                            income: Number(formParams.input),
                            expend: 0
                        }
                    } else {
                        dataParams = {
                            ...formParams,
                            radio: '支出',
                            income: 0,
                            expend: Number(formParams.input)
                        }
                    }
                    let newList = {}
                    if (JSON.stringify(list) === '{}') {
                        newList = {
                            addExpend: dataParams.expend, //总支出
                            addIncome: dataParams.income, //总收入
                            [cyear]: {
                               yearExpend: dataParams.expend, //年支出
                               yearIncome: dataParams.income, //年收入
                               [cmonth]: {
                                    monthExpend: dataParams.expend, //月支出
                                    monthIncome: dataParams.income, //月收入
                                    day: {
                                       [cdate]: {
                                           dayExpend: dataParams.expend, //本日支出
                                           dayIncome: dataParams.income, //本日收入
                                           data: [dataParams]
                                       }
                                    }
                                }
                            }
                        }
                        list = {...newList}
                    } else {
                        if (list[cyear]) {
                            if (list[cyear][cmonth]) {
                                if (list[cyear][cmonth].day[cdate]) {
                                    list.addExpend = (Number(list.addExpend)*100 + Number(dataParams.expend)*100) /100 //总支出
                                    list.addIncome = (Number(list.addIncome)*100 + Number(dataParams.income)*100) /100 //总收入
                                    list[cyear].yearExpend = (Number(list[cyear].yearExpend)*100 + Number(dataParams.expend)*100) /100 //年支出
                                    list[cyear].yearIncome = (Number(list[cyear].yearIncome)*100 + Number(dataParams.income)*100) /100 //年收入
                                    list[cyear][cmonth].monthExpend = (Number(list[cyear][cmonth].monthExpend)*100 + Number(dataParams.expend)*100) /100 //月支出
                                    list[cyear][cmonth].monthIncome = (Number(list[cyear][cmonth].monthIncome)*100 + Number(dataParams.income)*100) /100 //月收入
                                    list[cyear][cmonth].day[cdate].dayExpend = (Number(list[cyear][cmonth].day[cdate].dayExpend)*100 + Number(dataParams.expend)*100) /100 //本日支出
                                    list[cyear][cmonth].day[cdate].dayIncome = (Number(list[cyear][cmonth].day[cdate].dayIncome)*100 + Number(dataParams.income)*100) /100 //本日收入
                                    list[cyear][cmonth].day[cdate].data.push(dataParams)
                                } else {
                                    newList = {
                                        [cdate]: {
                                            dayExpend: dataParams.expend, //本日支出
                                            dayIncome: dataParams.income, //本日收入
                                            data: [dataParams]
                                        }
                                    }
                                    Object.assign(list[cyear][cmonth].day, newList)
                                    list.addExpend = (Number(list.addExpend)*100 + Number(dataParams.expend)*100) /100 //总支出
                                    list.addIncome = (Number(list.addIncome)*100 + Number(dataParams.income)*100) /100 //总收入
                                    list[cyear].yearExpend = (Number(list[cyear].yearExpend)*100 + Number(dataParams.expend)*100) /100 //年支出
                                    list[cyear].yearIncome = (Number(list[cyear].yearIncome)*100 + Number(dataParams.income)*100) /100 //年收入
                                    list[cyear][cmonth].monthExpend = (Number(list[cyear][cmonth].monthExpend)*100 + Number(dataParams.expend)*100) /100 //月支出
                                    list[cyear][cmonth].monthIncome = (Number(list[cyear][cmonth].monthIncome)*100 + Number(dataParams.income)*100) /100 //月收入
                                }
                            } else {
                                newList = {
                                    [cmonth]: {
                                        monthExpend: dataParams.expend, //月支出
                                        monthIncome: dataParams.income, //月收入
                                        day:{
                                            [cdate]: {
                                                dayExpend: dataParams.expend, //本日支出
                                                dayIncome: dataParams.income, //本日收入
                                                data: [dataParams]
                                            }
                                        }
                                    }
                                }
                                Object.assign(list[cyear], newList)
                                list.addExpend = (Number(list.addExpend)*100 + Number(dataParams.expend)*100) /100  //总支出
                                list.addIncome = (Number(list.addIncome)*100 + Number(dataParams.income)*100) /100  //总收入
                                list[cyear].yearExpend = (Number(list[cyear].yearExpend)*100 + Number(dataParams.expend)*100) /100 //年支出
                                list[cyear].yearIncome = (Number(list[cyear].yearIncome)*100 + Number(dataParams.income)*100) /100 //年收入
                            }
                        } else {   
                            newList = {
                                [cyear]: {
                                    yearExpend: dataParams.expend, //年支出
                                    yearIncome: dataParams.income, //年收入
                                    [cmonth]: {
                                        monthExpend: dataParams.expend, //月支出
                                        monthIncome: dataParams.income, //月收入
                                        day: {
                                            [cdate]: {
                                                dayExpend: dataParams.expend, //本日支出
                                                dayIncome: dataParams.income, //本日收入
                                                data: [dataParams]
                                            }
                                        }
                                    }
                                }
                            }
                            Object.assign(list, newList)
                            list.addExpend = (Number(list.addExpend)*100 + Number(dataParams.expend)*100) /100, //总支出
                            list.addIncome = (Number(list.addIncome)*100 + Number(dataParams.income)*100) /100 //总收入
                        }
                    }

                    this.addDateList = {...list}
                    this.addCalculate(list) // 计算总金额
                    this.monthCalculate(list, cyear, cmonth) // 计算月金额
                    this.todayCalculate(list, cyear, cmonth, cdate) // 计算本日金额
                    uni.setStorageSync('list', list)
                    uni.showToast({title:"验证通过!", icon:"none"});
                    this.formReset()
                } else {
                    uni.showToast({ title: graceChecker.error, icon: "none" });
                }
            },
            // 表单取消
            formReset(e) {
                this.formDetail = {
                    ...this.formDetail,
                    radio: '', //方式
                    input: '', //金额
                    ramarks: '', //备注 
                }
                this.radioData = {
                    radio1: false,
                    radio2: false
                }
                this.getDate('end')
                this.$refs.popup.close()
            },
            // 计算总支出金额和总收入金额
            addCalculate(list) {
                let addIncome = list.addIncome //总收入
                let addExpend = list.addExpend //总支出
                let addBalance = (Number(addIncome)*100 - Number(addExpend)*100) /100
                this.addIncomeData = addIncome //总收入
                this.addExpendData = addExpend //总支出
                this.addBalanceData = addBalance //总余额
            },
            // 计算月支出金额和月收入金额
            monthCalculate(list, cyear, cmonth) {
                let nowyear = Number(this.timeList.cyear) //日历当前的年份
                let nowmonth = Number(this.timeList.cmonth) //日历当前的月份
                if (nowmonth == Number(cmonth) && nowyear == Number(cyear)) {
                    if (list[cyear] && list[cyear][cmonth]) {
                        let monthlistData = list[cyear][cmonth] //月的数据
                        let monthIncome = list[cyear][cmonth].monthIncome//月收入
                        let monthExpend = list[cyear][cmonth].monthExpend //月支出
                        this.monthIncomeData = monthIncome //月收入
                        this.monthExpendData = monthExpend //月支出
                    } else {
                        this.monthIncomeData = 0 //月收入
                        this.monthExpendData = 0 //月支出
                    }
                }
            },
            // 计算本日支出金额和本日收入金额
            todayCalculate(list, cyear, cmonth, cdate) {
                let nowTime = this.timeList.nowtime //当前的时间
                let newTime = `${cyear}-${cmonth}-${cdate}`
                if (nowTime === newTime) {
                    if (list[cyear] && list[cyear][cmonth] && list[cyear][cmonth].day[cdate]) {
                        let listData = list[cyear][cmonth].day[cdate].data //本日的数据
                        let income = list[cyear][cmonth].day[cdate].dayIncome //本日收入
                        let expend = list[cyear][cmonth].day[cdate].dayExpend //本日支出
                        this.incomeData = income
                        this.expendData = expend
                        this.nowList = listData
                        this.showDay = true
                    } else {
                        this.nowList = []
                        this.showDay = false
                    }
                }
            },
            getDate(type) {
                const date = new Date();
                let year = date.getFullYear();
                let month = date.getMonth() + 1;
                let day = date.getDate();
                if (type === 'start') {
                    year = year - 60;
                } else if (type === 'end') {
                    year = year;
                }
                month = month > 9 ? month : '0' + month;;
                day = day > 9 ? day : '0' + day;
                // if (type === 'end') {
                //     this.formDetail.picker = `${year}-${month}-${day}`
                //     this.date = `${year}-${month}-${day}`
                // }
                return `${year}-${month}-${day}`;
            }
        }
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}
    
    .contentHead {
        width: 750rpx;
        background: #5CACEE;
        color: #fff;
    }

    .contentHead-box {
        display: flex;
        margin: 20rpx;
    }

    .contentHead-v {
        width: 45%;
        text-align: left;
        margin-left: 10rpx;
        font-size: 30rpx;
    }

    .head-prefix {
        margin-bottom: 10rpx;
        color: #DEDEDE;
        opacity: 0.8;
        font-size: 36rpx;
    }

    .head-prefix-r {
        color: #EEEE00;
    }

    .todayView {
        display: flex;
        justify-content: space-between;
        align-items: center;
        width: 750rpx;
        margin: 10rpx 0;
        text-align: left;
        background: #F7F7F7;
        color: #8B8386;
    }

    .todayView-l{
        margin: 20rpx 20rpx;
    }

    .todayView-v {
        text-align: left;
        font-size: 32rpx;
        margin: 10rpx 0;
    }

    .todayView-v-prefix {
        color: red;
    }

    .todayView-r {
        margin-right: 30rpx;
    }

    .inventoryBox {
        width: 100%;
    }

    .inventoryBox-content {
        display: flex;
        border-bottom: 1px solid #EEE9E9;
        margin: 20rpx;
    }

    .inventoryBox-l {
        width: 30%;
        margin-right: 10rpx;
    }

    .inventoryBox-z{
        width: 1px;
        background-color: #EEE9E9;
        margin: 20rpx 0;
    }

    .inventoryBox-r {
        font-size: 30rpx;
        padding-left: 20rpx;
    }

    .inventoryItem {
        margin-bottom: 10rpx;
        font-size: 30rpx;
    }

    .greenStyle{
        color: green;
    }

    .redStyle{
        color: red;
    }

    .brnStyle {
        margin: 60rpx 20rpx;
        width: 100%;
    }

    .brnStyle-b {
        width: 50%;
        background: #5CACEE;
        color: #fff;
    }

    .formBox {
        background: #fff;
        width: 600rpx;
        border-radius: 36rpx;
        padding: 40rpx;
    }

    .formView {
        margin: 20rpx 0;
        /* border: */
    }

    .formView-t {
        margin-bottom: 20rpx;
        color: #CD9B9B;
    }

    .formView-input{
        width: 100%;
    }

    .formView-text {
        width: 100%;
        height: 200rpx;
        background: #FAFAFA;
    }
    .formBrn {
        display: flex;
        justify-content: space-around;
        margin: 20rpx 20rpx 0;
    }

    .brn {
        width: 45%;
    }

</style>
