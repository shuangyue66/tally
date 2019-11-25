<template>
	<view class="content">
        <view class="contentHead">
            <view class="contentHead-v">
                <view>总收入￥: {{addincomeData}}</view>
                <view style="color: #red;">总支出￥: {{addexpendData}}</view>
                <view>总余额￥: {{addbalanceData}}</view>
            </view>
            <view class="contentHead-v">
                <view>月收入￥: {{monthIncomeData}}</view>
                <view>月支出￥: {{monthExpendData}}</view>
            </view>
        </view>
		<uni-calendar 
            :insert="true"
            :lunar="true" 
            :start-date="'2019-1-1'"
            :end-date="'2019-5-20'"
            @change="change"
            @month-switch="monthSwitch"
        />
        <view v-if="showDay">
            <view class="todayView">
                <view class="todayView-v" >本日总收￥: {{ incomeData }}</view>
                <view class="todayView-v">本日总支出￥: {{ '-' + expendData }}</view>
            </view>
            <view class="inventoryBox"
                    v-for="(item, index) in nowList"
                    :key="index">
                    <view class="inventoryBox-l">
                        <view>{{ item.picker }}</view>
                        <view>{{ item.radio }}</view>
                        <view>{{ item.input }}</view>
                    </view>
                    <view class="inventoryBox-r">
                        <view>{{ item.ramarks }}</view>
                    </view>
            </view>
        </view>
        <button @click="onForm">哈哈哈哈</button>
        <view>
            <uni-popup 
                ref="popup" 
                type="center"
                :maskClick="false">
                <view class="formBox">
                    <form @submit="formSubmit" @reset="formReset">
                        <view class="formView">
                            <text class="formView-t">时间:</text>
                            <view>
                                <picker mode="date"
                                        name="picker"
                                        :value="date"
                                        :start="startDate"
                                        :end="endDate"
                                        @change="bindDateChange">
                                    <view>{{date}}</view>
                                </picker>
                            </view>
                        </view>
                        <view class="formView">
                            <text class="formView-t">方式:</text>
                            <radio-group name="radio">
                                <label>
                                    <radio value="radio1" /><text>收入</text>
                                </label>
                                <label>
                                    <radio value="radio2" /><text>支出</text>
                                </label>
                            </radio-group>
                        </view>
                        <view class="formView">
                            <text class="formView-t">金额:</text>
                            <input name="input" type="number" placeholder="请输入金额，小数点最多两位" />
                        </view>
                        <view class="formView">
                            <view class="formView-t">备注:</view>
                            <textarea name="ramarks"
                                    style="height: 200rpx; background: #FAFAFA;"
                                    placeholder="请输入备注"
                                    maxlength="100"
                                    />
                        </view>
                        <view class="formBrn">
                            <button class="brn"
                                    type="warn"
                                    plain="true"
                                    form-type="reset">取消</button>
                            <button class="brn"
                                    type="primary" 
                                    plain="true"
                                    form-type="submit">确认</button>
                        </view>
                    </form>
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
                timeList: {
                    nowtime: '', // 今天日期(年-月-日)
                    cyear: '', // 年
                    cmonth: '', // 月
                    cdate: '', // 日
                },
                YMtime: '', // 年月
                adddateList: {}, // 总日期金额数组
                addincomeData: 0, //总收入数据
                addexpendData: 0, // 总支出数据
                addbalanceData: 0, // 总余额数据
                monthIncomeData: 0, // 月收入数据
                monthExpendData: 0, // 月支出数据
                incomeData: 0, // 本日收入数据
                expendData: 0, // 本日支出数据
                nowList: [], // 今天的数据
                date: currentDate,
                showDay: true
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
            this.YMtime = `${Y}-${M}`
            this.timeList = {
                ...newtime
            }
            if (newList) {
                this.adddateList = newList
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
                this.YMtime = YM 
                this.showDay = true
                this.timeList = {
                   ...newtime
                }
                let add = this.adddateList
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
                if (YM === this.YMtime) {
                    this.showDay = true
                } else {
                    this.showDay = false
                }
                this.timeList = {
                    ...this.timeList,
                    cyear: year,
                    cmonth: month
                }
                let add = this.adddateList
                this.monthCalculate(add, year, month)
            },
            onForm(e) {
                this.$refs.popup.open()
            },
            bindDateChange: function(e) {
                this.date = e.target.value
            },
            // 表单确认
            formSubmit(e) {
                let rule = [
                    {name:"picker", checkType: "notnull", checkRule:"",  errorMsg:"请选择日期"},
                    {name:"radio", checkType: "in", checkRule:"radio1,radio2",  errorMsg:"请选择方式"},
                    {name:"input", checkType: "therge",  errorMsg:"请输入金额"}
                ];
                let formData = e.detail.value;
                let checkRes = graceChecker.check(formData, rule);
                if(checkRes){
                    let time = formData.picker // 日期
                    let cyear = time.split('-')[0] // 年份
                    let cmonth = time.split('-')[1] // 月份
                    let cdate = time.split('-')[2] // 几号
                    let list = {...this.adddateList} // 总数据
                    
                    let asd = formData

                    if (formData.radio === 'radio1') {
                        asd = {
                            ...asd,
                            income: Number(formData.input),
                            expend: 0
                        }
                    } else {
                        asd = {
                            ...asd,
                            income: 0,
                            expend: Number(formData.input)
                        }
                    }

                    let newList = {}
                    if (JSON.stringify(list) === '{}') {
                        newList = {
                            addexpend: asd.expend, //总支出
                            addincome: asd.income, //总收入
                            [cyear]: {
                               yearexpend: asd.expend, //年支出
                               yearincome: asd.income, //年收入
                               [cmonth]: {
                                    monthexpend: asd.expend, //月支出
                                    monthincome: asd.income, //月收入
                                    day: {
                                       [cdate]: {
                                           dayexpend: asd.expend, //本日支出
                                           dayincome: asd.income, //本日收入
                                           data: [asd]
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
                                    list.addexpend = (Number(list.addexpend)*100 + Number(asd.expend)*100) /100 //总支出
                                    list.addincome = (Number(list.addincome)*100 + Number(asd.income)*100) /100 //总收入
                                    list[cyear].yearexpend = (Number(list[cyear].yearexpend)*100 + Number(asd.expend)*100) /100 //年支出
                                    list[cyear].yearincome = (Number(list[cyear].yearincome)*100 + Number(asd.income)*100) /100 //年收入
                                    list[cyear][cmonth].monthexpend = (Number(list[cyear][cmonth].monthexpend)*100 + Number(asd.expend)*100) /100 //月支出
                                    list[cyear][cmonth].monthincome = (Number(list[cyear][cmonth].monthincome)*100 + Number(asd.income)*100) /100 //月收入
                                    list[cyear][cmonth].day[cdate].dayexpend = (Number(list[cyear][cmonth].day[cdate].dayexpend)*100 + Number(asd.expend)*100) /100 //本日支出
                                    list[cyear][cmonth].day[cdate].dayincome = (Number(list[cyear][cmonth].day[cdate].dayincome)*100 + Number(asd.income)*100) /100 //本日收入
                                    list[cyear][cmonth].day[cdate].data.push(asd)
                                } else {
                                    newList = {
                                        [cdate]: {
                                            dayexpend: asd.expend, //本日支出
                                            dayincome: asd.income, //本日收入
                                            data: [asd]
                                        }
                                    }
                                    Object.assign(list[cyear][cmonth].day, newList)
                                    list.addexpend = (Number(list.addexpend)*100 + Number(asd.expend)*100) /100 //总支出
                                    list.addincome = (Number(list.addincome)*100 + Number(asd.income)*100) /100 //总收入
                                    list[cyear].yearexpend = (Number(list[cyear].yearexpend)*100 + Number(asd.expend)*100) /100 //年支出
                                    list[cyear].yearincome = (Number(list[cyear].yearincome)*100 + Number(asd.income)*100) /100 //年收入
                                    list[cyear][cmonth].monthexpend = (Number(list[cyear][cmonth].monthexpend)*100 + Number(asd.expend)*100) /100 //月支出
                                    list[cyear][cmonth].monthincome = (Number(list[cyear][cmonth].monthincome)*100 + Number(asd.income)*100) /100 //月收入
                                }
                            } else {
                                newList = {
                                    [cmonth]: {
                                        monthexpend: asd.expend, //月支出
                                        monthincome: asd.income, //月收入
                                        day:{
                                            [cdate]: {
                                                dayexpend: asd.expend, //本日支出
                                                dayincome: asd.income, //本日收入
                                                data: [asd]
                                            }
                                        }
                                    }
                                }
                                Object.assign(list[cyear], newList)
                                list.addexpend = (Number(list.addexpend)*100 + Number(asd.expend)*100) /100  //总支出
                                list.addincome = (Number(list.addincome)*100 + Number(asd.income)*100) /100  //总收入
                                list[cyear].yearexpend = (Number(list[cyear].yearexpend)*100 + Number(asd.expend)*100) /100 //年支出
                                list[cyear].yearincome = (Number(list[cyear].yearincome)*100 + Number(asd.income)*100) /100 //年收入
                            }
                        } else {   
                            newList = {
                                [cyear]: {
                                    yearexpend: asd.expend, //年支出
                                    yearincome: asd.income, //年收入
                                    [cmonth]: {
                                        monthexpend: asd.expend, //月支出
                                        monthincome: asd.income, //月收入
                                        day: {
                                            [cdate]: {
                                                dayexpend: asd.expend, //本日支出
                                                dayincome: asd.income, //本日收入
                                                data: [asd]
                                            }
                                        }
                                    }
                                }
                            }
                            Object.assign(list, newList)
                            list.addexpend = (Number(list.addexpend)*100 + Number(asd.expend)*100) /100, //总支出
                            list.addincome = (Number(list.addincome)*100 + Number(asd.income)*100) /100 //总收入
                        }
                    }

                    this.adddateList = {...list}
                    this.addCalculate(list) // 计算总金额
                    this.monthCalculate(list, cyear, cmonth) // 计算月金额
                    this.todayCalculate(list, cyear, cmonth, cdate) // 计算本日金额
                    uni.setStorageSync('list', list)
                    uni.showToast({title:"验证通过!", icon:"none"});
                    this.$refs.popup.close()
                } else {
                    uni.showToast({ title: graceChecker.error, icon: "none" });
                }
            },
            // 表单取消
            formReset(e) {
                this.$refs.popup.close()
            },
            // 计算总支出金额和总收入金额
            addCalculate(list) {
                let addIncome = list.addincome //总收入
                let addExpend = list.addexpend //总支出
                let addBalance = (Number(addIncome)*100 - Number(addExpend)*100) /100
                this.addincomeData = addIncome //总收入
                this.addexpendData = addExpend //总支出
                this.addbalanceData = addBalance //总余额
            },
            // 计算月支出金额和月收入金额
            monthCalculate(list, cyear, cmonth) {
                let nowyear = Number(this.timeList.cyear) //日历当前的年份
                let nowmonth = Number(this.timeList.cmonth) //日历当前的月份
                if (nowmonth == Number(cmonth) && nowyear == Number(cyear)) {
                    if (list[cyear] && list[cyear][cmonth]) {
                        let monthlistData = list[cyear][cmonth] //月的数据
                        let monthIncome = list[cyear][cmonth].monthincome //月收入
                        let monthExpend = list[cyear][cmonth].monthexpend //月支出
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
                        let income = list[cyear][cmonth].day[cdate].dayincome //本日收入
                        let expend = list[cyear][cmonth].day[cdate].dayexpend //本日支出
                        this.incomeData = income
                        this.expendData = expend
                        this.nowList = listData
                    } else {
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
        display: flex;
        width: 700rpx;
        margin-top: 10rpx;
    }

    .contentHead-v {
        width: 45%;
        text-align: left;
        margin-left: 10rpx;
    }
    
    .todayView {
        display: flex;
        width: 700rpx;
        margin: 20rpx 0;
        height: 60rpx;
        background: #F7F7F7;
    }

    .todayView-v {
        width: 45%;
        text-align: left;
        margin-left: 0rpx;
    }

    .inventoryBox {
        display: flex;
        width: 100%;
        border-bottom: 1px solid #EEE9E9;
        margin: 10rpx 0;
    }

    .inventoryBox-l {
        width: 30%;
        margin-right: 10rpx;
    }

    .formBox {
        background: #fff;
        width: 600rpx;
        /* height: 700rpx; */
        border-radius: 36rpx;
        padding: 20rpx;
    }

    .formView {
        margin: 10rpx 0;
        /* border: */
    }

    .formView-t {
        margin-bottom: 10rpx;
    }

    .formBrn {
        display: flex;
        justify-content: space-around;
        margin: 20rpx;
    }

    .brn {
        width: 45%;
    }
</style>
