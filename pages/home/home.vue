<template>
	<view class="page">
		<view class="upper">
			<view class="upper-cell">
				<view class="butt" hover-class="butt-tap" @touchstart="start(1)" @touchend="end">
					<image src="../../static/up.png" mode="aspectFit"></image>
				</view>
				<view class="butt" hover-class="butt-tap" @touchstart="start(2)" @touchend="end">
					<image src="../../static/down.png" mode="aspectFit"></image>
				</view>
			</view>
			<view class="upper-cell">
				<view class="butt" hover-class="butt-tap" @touchstart="start(3)" @touchend="end">
					<image src="../../static/up.png" mode="aspectFit"></image>
				</view>
				<view class="butt" hover-class="butt-tap" @touchstart="start(4)" @touchend="end">
					<image src="../../static/down.png" mode="aspectFit"></image>
				</view>
			</view>
		</view>
		<view class="column">
			<view class="column-cell" hover-class="cellSelected" @tap="doManual">还原</view>
			<view :class="{cellSelected: memorise}" class="column-cell" @tap="doMemorise">{{memoriseState}}</view>
			<view class="column-cell" hover-calss="cellSelected" @tap="doPop">重复</view>
			<view :class="{cellSelected: doorOpen}" class="column-cell" @tap="doDoorOpen">{{doorState}}</view>
			<view :class="{cellSelected: lightsOn}" class="column-cell" @tap="doLightsOn">{{lightsState}}</view>
			<view class="column-cell" hover-calss="cellSelected" @tap="doAuto">自动</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				intervalId: 0,
				memorise: false,
				doorOpen: false,
				lightsOn: false,
				//pop: false,
				os: [], //操作栈
				memoriseState: "记忆",
				doorState: "开门",
				lightsState: "开灯",
				//popState:"自动",
			}
		},
		// onLoad: function(){
		// 	uni.getStorage({
		// 		key: "os",
		// 		success: (res) => {
		// 			this.os = res.data;
		// 		}
		// 	})
		// },
		methods: {
			start: function(index) {
				this.write(index);
				console.log("start");
				this.intervalID = setInterval(() => {
					this.write(index);

				}, 200);
				console.log(this.intervalID);
			},
			end: function() {
				console.log("end");
				clearInterval(this.intervalID);
			},
			write: function(index) {
				let buf = new ArrayBuffer(1);
				let dataView = new DataView(buf);
				dataView.setInt8(0, index);

				// let buf = Buffer.from("a");
				// let bufferstr = buf.buffer;

				var that = this;
				uni.writeBLECharacteristicValue({
					deviceId: "3C:A5:09:7A:D4:94",
					serviceId: "0000FFE0-0000-1000-8000-00805F9B34FB",
					characteristicId: "0000FFE1-0000-1000-8000-00805F9B34FB",
					value: buf,
					success: function(res) {
						console.log("数据发送成功");
						if (that.memorise) {
							that.os.push(index);
							console.log(that.os);
						}
						uni.hideLoading();
					},
					fail: function(res) {
						console.log("数据发送失败:" + JSON.stringify(res))
					},
					complete: function(res) {

					},
				})
			},
			doManual: function() {
				this.write(9);
				uni.showToast({
					title: "还原",
					icon: "none",
				})
			},
			doAuto: function() {
				this.write(6);
				uni.showToast({
					title: "自动",
					icon: "none",
				})
			},
			doMemorise: function() {
				console.log('do memorise');
				if (!this.memorise) {
					this.memorise = true;
					this.os = [];
					this.memoriseState = "完成";
					//this.write(5);
				} else {
					console.log(this.os);
					if (this.os.length != 0) {
						console.log('here');
						uni.setStorage({
							key: 'os',
							data: this.os,
							success: () => {
								uni.showToast({
									title: "保存成功",
									icon: "success",
								})
							}
						})

					}
					this.memorise = false;
					this.memoriseState = "记忆";
					//this.write(6);
				}

				// this.write(5);
				// uni.showToast({
				//  	title:"记忆模式",
				// 	icon:"none",
				// })
			},
			doPop: function() {
				console.log('do pop');
				uni.getStorage({
					key: "os",
					success: (res) => {
						this.os = res.data;
						console.log(this.os);
						if (this.os.length != 0) {
							let interval = setInterval(() => {
								console.log('write pop');
								// this.write(this.os[this.os.length - 1]);
								// this.os.pop();
								this.write(this.os[0]);
								this.os.shift();
								if (this.os.length == 0) {
									clearInterval(interval);
									console.log("操作全部弹出");
								}
							}, 100)
						
						}
					}
				})


				// let interval = setInterval(() => {
				// 	console.log('write pop');
				// 	console.log(this.os[this.os.length - 1]);
				// 	this.os.pop();
				// 	if (this.os.length == 0) clearInterval(interval);
				// }, 100)
				// console.log(this.os.length);
				// this.os.pop();
				// console.log(this.os.length);

				// if (this.os.length != 0) {
				// 	let interval = setInterval(() => {
				// 		console.log('write pop');
				// 		// this.write(this.os[this.os.length - 1]);
				// 		// this.os.pop();
				// 		this.write(this.os[0]);
				// 		this.os.shift();
				// 		if (this.os.length == 0) {
				// 			clearInterval(interval);
				// 			console.log("操作全部弹出");
				// 		}
				// 	}, 100)

				// }

				// this.write(6);
				// uni.showToast({
				//  	title:"自动模式",
				// 	icon:"none",
				// })
			},
			doDoorOpen: function() {
				if (this.doorOpen) { //关门
					this.write(7);
					this.doorOpen = false;
					this.doorState = "开门";
				} else { //开门
					this.write(7);
					this.doorOpen = true;
					this.doorState = "关门";
				}
			},
			doLightsOn: function() {
				if (this.lightsOn) { //关灯
					this.write(8);
					this.lightsOn = false;
					this.lightsState = "开灯";
				} else { //开灯
					this.write(8);
					this.lightsOn = true;
					this.lightsState = "关灯";
				}
			},
		}
	}
</script>

<style>
	page {
		height: 100%;
		width: 100%;
	}

	.page {
		height: 100%;
		width: 100%;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}

	.upper {
		height: 60vh;
		width: 100vw;
		display: flex;
		justify-content: space-around;
		align-items: center;
	}

	.upper-cell {
		height: 80%;
		width: 200rpx;
		background-color: #f3f2f4;
		border-radius: 50upx;
		display: flex;
		flex-direction: column;
		justify-content: space-around;
		align-items: center;
	}

	.butt {
		height: 100rpx;
		width: 100rpx;
		border-radius: 100%;
		box-shadow: 2rpx 3rpx 20rpx #dcdcdc;
		background-color: #ffffff;
		display: flex;
		justify-content: center;
		align-items: center;
	}

	.butt-tap {
		height: 100rpx;
		width: 100rpx;
		border-radius: 100%;
		box-shadow: 2rpx 3rpx 20rpx #dcdcdc;
		background-color: #f4f4f4;
		display: flex;
		justify-content: center;
		align-items: center;
	}

	image {
		height: 70%;
		width: 70%;
	}

	.column {
		height: 20vh;
		width: 80vw;
		margin: 30rpx;
		display: flex;
		justify-content: space-around;
		align-items: center;
		flex-wrap: wrap;
	}

	.column-cell {
		margin: 20rpx;
		font-size: 30rpx;
		height: 130rpx;
		width: 130rpx;
		border-radius: 100%;
		box-shadow: 2rpx 3rpx 20rpx #dcdcdc;
		color: #ffffff;
		background-color: #FA8072;
		display: flex;
		justify-content: space-around;
		align-items: center;
	}

	.cellSelected {
		margin: 20rpx;
		font-size: 30rpx;
		height: 130rpx;
		width: 130rpx;
		border-radius: 100%;
		box-shadow: 2rpx 3rpx 20rpx #dcdcdc;
		color: #FA8072;
		background-color: #ffffff;
		display: flex;
		justify-content: space-around;
		align-items: center;
	}
</style>
