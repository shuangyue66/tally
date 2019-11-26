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
                    <view>
                        <text class="formView-t">时间:</text>
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
                        <text class="formView-t">方式:</text>
                        <radio-group @change="formRadio"
                                    :value="formDetail.radio"
                                    >
                            <label>
                                <radio value="radio1" :checked="radioData.radio1"/><text>收入</text>
                            </label>
                            <label>
                                <radio value="radio2" :checked="radioData.radio2"/><text>支出</text>
                            </label>
                        </radio-group>
                    </view>
                    <view class="formView">
                        <text class="formView-t">金额:</text>
                        <input type="digit"
                                :value="formDetail.input"
                                @input="formInput"
                                placeholder="请输入金额，小数点最多两位" />
                    </view>
                    <view class="formView">
                        <view class="formView-t">备注:</view>
                        <textarea :value="formDetail.ramarks" 
                                @input="formRamarks"
                                style="height: 200rpx; background: #FAFAFA;"
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
                showDay: true,
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
                console.log(this.formDetail, 'formDetail')
                this.$refs.popup.open()
            },
            // 表单时间
            bindDateChange: function(e) {
                console.log(e, 'eaeea')
                this.date = e.detail.value
                this.formDetail.picker = e.detail.value
            },
            // 表单方式
            formRadio(e) {
                console.log(e, 'eeeeee')
                if (e.detail.value === 'radio1') {
                    this.radioData.radio1 = true
                } else {
                    this.radioData.radio2 = true
                }
                this.formDetail.radio = e.detail.value
            },
            // 表单金额
            formInput(e) {
                console.log(e.detail.value, 'weq')
                this.formDetail.input = e.detail.value
            },
            // 表单备注
            formRamarks(e) {
                console.log(e.detail.value, 'formRamarks')
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
                    let list = {...this.adddateList} // 总数据

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
                            addexpend: dataParams.expend, //总支出
                            addincome: dataParams.income, //总收入
                            [cyear]: {
                               yearexpend: dataParams.expend, //年支出
                               yearincome: dataParams.income, //年收入
                               [cmonth]: {
                                    monthexpend: dataParams.expend, //月支出
                                    monthincome: dataParams.income, //月收入
                                    day: {
                                       [cdate]: {
                                           dayexpend: dataParams.expend, //本日支出
                                           dayincome: dataParams.income, //本日收入
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
                                    list.addexpend = (Number(list.addexpend)*100 + Number(dataParams.expend)*100) /100 //总支出
                                    list.addincome = (Number(list.addincome)*100 + Number(dataParams.income)*100) /100 //总收入
                                    list[cyear].yearexpend = (Number(list[cyear].yearexpend)*100 + Number(dataParams.expend)*100) /100 //年支出
                                    list[cyear].yearincome = (Number(list[cyear].yearincome)*100 + Number(dataParams.income)*100) /100 //年收入
                                    list[cyear][cmonth].monthexpend = (Number(list[cyear][cmonth].monthexpend)*100 + Number(dataParams.expend)*100) /100 //月支出
                                    list[cyear][cmonth].monthincome = (Number(list[cyear][cmonth].monthincome)*100 + Number(dataParams.income)*100) /100 //月收入
                                    list[cyear][cmonth].day[cdate].dayexpend = (Number(list[cyear][cmonth].day[cdate].dayexpend)*100 + Number(dataParams.expend)*100) /100 //本日支出
                                    list[cyear][cmonth].day[cdate].dayincome = (Number(list[cyear][cmonth].day[cdate].dayincome)*100 + Number(dataParams.income)*100) /100 //本日收入
                                    list[cyear][cmonth].day[cdate].data.push(dataParams)
                                } else {
                                    newList = {
                                        [cdate]: {
                                            dayexpend: dataParams.expend, //本日支出
                                            dayincome: dataParams.income, //本日收入
                                            data: [dataParams]
                                        }
                                    }
                                    Object.assign(list[cyear][cmonth].day, newList)
                                    list.addexpend = (Number(list.addexpend)*100 + Number(dataParams.expend)*100) /100 //总支出
                                    list.addincome = (Number(list.addincome)*100 + Number(dataParams.income)*100) /100 //总收入
                                    list[cyear].yearexpend = (Number(list[cyear].yearexpend)*100 + Number(dataParams.expend)*100) /100 //年支出
                                    list[cyear].yearincome = (Number(list[cyear].yearincome)*100 + Number(dataParams.income)*100) /100 //年收入
                                    list[cyear][cmonth].monthexpend = (Number(list[cyear][cmonth].monthexpend)*100 + Number(dataParams.expend)*100) /100 //月支出
                                    list[cyear][cmonth].monthincome = (Number(list[cyear][cmonth].monthincome)*100 + Number(dataParams.income)*100) /100 //月收入
                                }
                            } else {
                                newList = {
                                    [cmonth]: {
                                        monthexpend: dataParams.expend, //月支出
                                        monthincome: dataParams.income, //月收入
                                        day:{
                                            [cdate]: {
                                                dayexpend: dataParams.expend, //本日支出
                                                dayincome: dataParams.income, //本日收入
                                                data: [dataParams]
                                            }
                                        }
                                    }
                                }
                                Object.assign(list[cyear], newList)
                                list.addexpend = (Number(list.addexpend)*100 + Number(dataParams.expend)*100) /100  //总支出
                                list.addincome = (Number(list.addincome)*100 + Number(dataParams.income)*100) /100  //总收入
                                list[cyear].yearexpend = (Number(list[cyear].yearexpend)*100 + Number(dataParams.expend)*100) /100 //年支出
                                list[cyear].yearincome = (Number(list[cyear].yearincome)*100 + Number(dataParams.income)*100) /100 //年收入
                            }
                        } else {   
                            newList = {
                                [cyear]: {
                                    yearexpend: dataParams.expend, //年支出
                                    yearincome: dataParams.income, //年收入
                                    [cmonth]: {
                                        monthexpend: dataParams.expend, //月支出
                                        monthincome: dataParams.income, //月收入
                                        day: {
                                            [cdate]: {
                                                dayexpend: dataParams.expend, //本日支出
                                                dayincome: dataParams.income, //本日收入
                                                data: [dataParams]
                                            }
                                        }
                                    }
                                }
                            }
                            Object.assign(list, newList)
                            list.addexpend = (Number(list.addexpend)*100 + Number(dataParams.expend)*100) /100, //总支出
                            list.addincome = (Number(list.addincome)*100 + Number(dataParams.income)*100) /100 //总收入
                        }
                    }

                    this.adddateList = {...list}
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
                console.log(e, '3ee')
                this.formDetail = {
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
                console.log('qwwww')
                let nowTime = this.timeList.nowtime //当前的时间
                let newTime = `${cyear}-${cmonth}-${cdate}`
                if (nowTime === newTime) {
                    if (list[cyear] && list[cyear][cmonth] && list[cyear][cmonth].day[cdate]) {
                        console.log('hhhh')
                        let listData = list[cyear][cmonth].day[cdate].data //本日的数据
                        let income = list[cyear][cmonth].day[cdate].dayincome //本日收入
                        let expend = list[cyear][cmonth].day[cdate].dayexpend //本日支出
                        this.incomeData = income
                        this.expendData = expend
                        this.nowList = listData
                    } else {
                        console.log('llll')
                        this.nowList = []
                        this.showDay = false
                    }
                }
            },
            getDate(type) {
                console.log(type, 'type')
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
                if (type === 'end') {
                    this.formDetail.picker = `${year}-${month}-${day}`
                    this.date = `${year}-${month}-${day}`
                }
                console.log(this.formDetail, '最后一天')
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
