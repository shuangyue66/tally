<template>
	<view class="content">
        <view class="contentHead">
            <view class="contentHead-v">
                <view>总收入￥: 0</view>
                <view>总支出￥: 1000</view>
            </view>
            <view class="contentHead-v">
                <view>月收入￥: </view>
                <view>月支出￥: </view>
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
        <view>
            <view class="todayView">
                <view class="todayView-v" >本日总收￥:</view>
                <view class="todayView-v">本日总支出￥:</view>
            </view>
            <!-- <view v-for="(item, index) in sss" :key="item.id">{{index + ':' + item.name}}</view> -->
            <button @click="onForm">哈哈哈哈</button>
        </view>
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
                            <input name="input" type="number" placeholder="请输入金额" />
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
                nowtime: '',
                showForm: false,
                sss: [
                    {
                        id: 0,
                        name: "1"
                    },
                    {
                        id: 1,
                        name: "2"
                    }
                ],
                date: currentDate
			}
		},
		onLoad() {
            let date = new Date
            let Y = date.getFullYear() + '-'
            let M = date.getMonth() + 1 + '-'
            let D = date.getDate();
            let nowtime = Y + M + D
            this.nowtime = String(nowtime)
        },
        computed: {
            startDate() {
                console.log(this.getDate('start'), '11111')
            },
            endDate() {
                console.log(this.getDate('end'), '22222')
                return this.getDate('end');
            }
        },
		methods: {
            // 点击日期触发
			change(e) {
                console.log(e);
            },
            // 更换月份触发
            monthSwitch(e) {
                console.log(e)
            },
            onForm(e) {
                this.showForm = true
                this.$refs.popup.open()
            },
            bindDateChange: function(e) {
                console.log(e.target.value, '1123123')
                this.date = e.target.value
            },
            // 表单确认
            formSubmit(e) {
                let rule = [
                    {name:"picker", checkType: "notnull", checkRule:"",  errorMsg:"请选择日期"},
                    {name:"radio", checkType: "in", checkRule:"radio1,radio2",  errorMsg:"请选择方式"},
                    {name:"input", checkType: "therge",  errorMsg:"请输入金额"}
                ];
                console.log(e, 'eeee')
                console.log(e.detail.value, 'ee')
                let formData = e.detail.value;
                let checkRes = graceChecker.check(formData, rule);
                if(checkRes){
                    uni.showToast({title:"验证通过!", icon:"none"});
                } else {
                    uni.showToast({ title: graceChecker.error, icon: "none" });
                }
            },
            // 表单取消
            formReset(e) {
                this.showForm = false
                this.$refs.popup.close()
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
