<template>
	<view class="content">
		<uni-calendar 
            :insert="true"
            :lunar="true" 
            :start-date="'2019-1-1'"
            :end-date="'2019-5-20'"
            @change="change"
            @month-switch="monthSwitch"
        />
	</view>
</template>

<script>
    import {uniCalendar} from "@/components/uni-calendar/uni-calendar"
	export default {
        components: {
            uniCalendar
        },
		data() {
			return {
				nowtime: ''
			}
		},
		onLoad() {
            let date = new Date
            let Y = date.getFullYear() + '-'
            let M = date.getMonth() + 1 + '-'
            let D = date.getDate();
            let nowtime = Y + M + D
            this.nowtime = String(nowtime)
            console.log(String(nowtime), 'nowtime')
		},
		methods: {
			change(e) {
                console.log(e);
            },
            monthSwitch(e) {
                console.log(e)
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

</style>
