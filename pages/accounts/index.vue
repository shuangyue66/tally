<template>
	<view class="content">
        <view class="contentHead">
            <view class="contentHead-v">
                <view>总收入: 0</view>
                <view>总支出: 1000</view>
            </view>
            <view class="contentHead-v">
                <view>月收入: </view>
                <view>月支出: </view>
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
            <view>
                <view>本日总收:</view>
                <view>本日总支出:</view>
            </view>
            <view v-for="(item, index) in sss" :key="item.id">{{index + ':' + item.name}}</view>
            <button @click="onForm">哈哈哈哈</button>
        </view>
        <view>
            <uni-popup 
                ref="popup" 
                type="center"
                :maskClick="false">
                <view style="background: #fff; width: 600rpx; height: 700rpx;"> 
                    <form @submit="formSubmit" @reset="formReset">
                        <view>
                            <text>时间:</text>
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
                        <view>
                            <view>方式:</view>
                            <radio-group name="radio">
                                <label>
                                    <radio value="radio1" /><text>收入</text>
                                </label>
                                <label>
                                    <radio value="radio2" /><text>支出</text>
                                </label>
                            </radio-group>
                        </view>
                        <view>
                            <view>金额:</view>
                            <input name="input" placeholder="请输入金额" />
                        </view>
                        <view>
                            <view>备注: </view>
                            <textarea name="ramarks" placeholder="请输入备注"/>
                        </view>
                        <view>
                            <button type="default" form-type="reset">取消</button>
                            <button form-type="submit">确认</button>
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
                console.log(e.detail.value, 'ee')
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
    
</style>
