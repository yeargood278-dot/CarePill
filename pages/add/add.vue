<template>
	<view class="container">
		<view class="form-group">
			<view class="input-card">
				<text class="label">💊 药品信息</text>
				<input class="main-input" v-model="form.name" placeholder="请输入药品名称" placeholder-class="p-style" />
				<input class="main-input" v-model="form.dosage" placeholder="请输入剂量 (如: 1片/20ml)" placeholder-class="p-style" />
			</view>

			<view class="input-card">
				<text class="label">⏰ 时间与周期设置</text>
				
				<view class="picker-group">
					<view class="picker-row-main">
						<text class="row-left">提醒时间</text>
						<picker mode="time" :value="form.time" @change="e => form.time = e.detail.value" class="full-picker">
							<view class="time-display-box">
								<text class="time-val">{{ form.time }}</text>
								<text class="arrow">></text>
							</view>
						</picker>
					</view>
					<text class="tip-text">点击大号时间字体即可调整时、分</text>
				</view>

				<view class="divider"></view>

				<view class="date-section">
					<view class="picker-row">
						<text class="row-left">开始日期</text>
						<picker mode="date" :value="form.startDate" @change="e => form.startDate = e.detail.value">
							<view class="date-val-box">{{ form.startDate }}</view>
						</picker>
					</view>
					<view class="picker-row">
						<text class="row-left">结束日期</text>
						<picker mode="date" :value="form.endDate" @change="e => form.endDate = e.detail.value">
							<view class="date-val-box">{{ form.endDate }}</view>
						</picker>
					</view>
				</view>
			</view>

			<view class="input-card">
				<text class="label">🔔 提醒模式</text>
				<checkbox-group class="methods-grid" @change="e => form.methods = e.detail.value">
					<label class="method-cell" :class="{active: form.methods.includes('voice')}">
						<checkbox value="voice" :checked="form.methods.includes('voice')" hidden />
						<text>📢 语音播报</text>
					</label>
					<label class="method-cell" :class="{active: form.methods.includes('vibrate')}">
						<checkbox value="vibrate" :checked="form.methods.includes('vibrate')" hidden />
						<text>📳 震动提醒</text>
					</label>
					<label class="method-cell" :class="{active: form.methods.includes('popup')}">
						<checkbox value="popup" :checked="form.methods.includes('popup')" hidden />
						<text>📱 弹窗提示</text>
					</label>
				</checkbox-group>
			</view>
		</view>

		<view class="footer-safe">
			<button class="save-btn" @click="savePlan">开启服药提醒计划</button>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		const d = new Date();
		const today = d.toISOString().split('T')[0];
		d.setMonth(d.getMonth() + 1);
		const nextMonth = d.toISOString().split('T')[0];
		return {
			form: {
				id: '',
				name: '',
				dosage: '',
				time: '08:00', // 默认时间
				startDate: today,
				endDate: nextMonth,
				methods: ['voice', 'vibrate', 'popup'],
				history: {}
			}
		}
	},
	methods: {
		savePlan() {
			if(!this.form.name || !this.form.dosage) {
				return uni.showToast({ title: '药名和剂量不能为空', icon: 'none' });
			}
			this.form.id = 'ID_' + Date.now();
			let list = uni.getStorageSync('medicine_list') || [];
			list.push(JSON.parse(JSON.stringify(this.form)));
			uni.setStorageSync('medicine_list', list);
			
			uni.showToast({ title: '计划已启动', icon: 'success' });
			setTimeout(() => { 
				uni.reLaunch({ url: '/pages/index/index' }); 
			}, 1000);
		}
	}
}
</script>

<style>
.container { 
	padding: 20px; 
	background: #F8F9FB; 
	min-height: 100vh; 
	/* 关键：确保底部内容不被遮挡 */
	padding-bottom: 120px; 
}

.input-card { 
	background: #fff; 
	border-radius: 24px; 
	padding: 20px; 
	margin-bottom: 20px; 
	box-shadow: 0 4px 12px rgba(0,0,0,0.03);
}

.label { 
	font-size: 16px; 
	font-weight: bold; 
	color: #333; 
	margin-bottom: 15px; 
	display: block; 
}

.main-input { 
	height: 55px; 
	border-bottom: 1px solid #F0F0F0; 
	font-size: 18px; 
	margin-bottom: 10px; 
}

.p-style { color: #CCC; }

/* 优化时间选择器布局 */
.picker-group {
	padding: 10px 0;
}

.picker-row-main {
	display: flex;
	justify-content: space-between;
	align-items: center;
}

.full-picker {
	flex: 1;
	display: flex;
	justify-content: flex-end;
}

.time-display-box {
	display: flex;
	align-items: center;
	background: #F0F4FF;
	padding: 8px 15px;
	border-radius: 12px;
}

.time-val { 
	font-size: 32px; 
	font-weight: 800; 
	color: #2C5FF7; 
	font-family: 'Courier New', Courier, monospace;
}

.arrow {
	margin-left: 10px;
	color: #2C5FF7;
	font-weight: bold;
	opacity: 0.5;
}

.tip-text {
	font-size: 12px;
	color: #999;
	margin-top: 8px;
	display: block;
}

.divider { 
	height: 1px; 
	background: #F5F5F5; 
	margin: 15px 0; 
}

.date-section .picker-row {
	display: flex;
	justify-content: space-between;
	align-items: center;
	height: 45px;
}

.row-left { font-size: 15px; color: #666; }

.date-val-box {
	font-size: 16px;
	color: #333;
	font-weight: 500;
	background: #F8F9FB;
	padding: 4px 10px;
	border-radius: 6px;
}

/* 提醒模式选择 */
.methods-grid { 
	display: grid; 
	grid-template-columns: 1fr 1fr; 
	gap: 12px; 
}

.method-cell { 
	background: #F8F9FB; 
	padding: 15px; 
	border-radius: 12px; 
	text-align: center; 
	border: 2px solid transparent; 
	transition: 0.2s; 
}

.method-cell.active { 
	background: #EEF2FF; 
	border-color: #2C5FF7; 
	color: #2C5FF7; 
	font-weight: bold; 
}

/* 底部按钮固定 */
.footer-safe {
	position: fixed;
	bottom: 0;
	left: 0;
	right: 0;
	padding: 20px 20px 40px;
	background: linear-gradient(to top, #F8F9FB 80%, rgba(248, 249, 251, 0));
	z-index: 99;
}

.save-btn { 
	background: #2C5FF7; 
	color: #fff; 
	height: 60px; 
	line-height: 60px; 
	border-radius: 30px; 
	font-size: 18px; 
	font-weight: bold; 
	box-shadow: 0 10px 20px rgba(44,95,247,0.3); 
}
</style>