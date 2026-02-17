<template>
	<view class="container">
		<view class="header-box">
			<view class="status-indicator" :class="{active: isRunning}">
				<view class="pulse"></view>
				<text>{{ isRunning ? '智能监护中' : '服务暂停' }}</text>
			</view>
			<view class="welcome-row">
				<text class="welcome-text">您好，记得按时吃药</text>
				<view class="report-entry" @click="navTo('/pages/report/report')">
					<text>健康报告</text>
					<text class="icon">📊</text>
				</view>
			</view>
		</view>

		<scroll-view scroll-y class="content-scroll">
			<view class="section-head">
				<text class="section-title">今日提醒</text>
				<text class="section-date">{{ todayStr }}</text>
			</view>

			<view v-if="todayList.length === 0" class="empty-state">
				<image src="/static/empty_meds.png" mode="aspectFit" class="empty-img"></image>
				<text>今天暂无服药计划</text>
			</view>

			<view v-for="item in todayList" :key="item.id" class="med-card" :class="{completed: item.isTaken}">
				<view class="card-left">
					<text class="med-time">{{ item.time }}</text>
					<view class="timeline-dot"></view>
				</view>
				<view class="card-body">
					<text class="med-name">{{ item.name }}</text>
					<view class="med-meta">
						<text class="dosage">剂量：{{ item.dosage }}</text>
						<view class="method-tags">
							<text v-if="item.methods.includes('voice')" class="tag">🗣️ 语音</text>
							<text v-if="item.methods.includes('vibrate')" class="tag">📳 震动</text>
						</view>
					</view>
				</view>
				<view class="card-right">
					<button class="check-btn" :disabled="item.isTaken" @click="confirmTake(item.id)">
						{{ item.isTaken ? '已完成' : '去打卡' }}
					</button>
				</view>
			</view>
			<view style="height: 120px;"></view>
		</scroll-view>

		<view class="fab-btn" @click="navTo('/pages/add/add')">
			<text class="plus">+</text>
			<text class="fab-text">加药</text>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			todayList: [],
			todayStr: new Date().toISOString().split('T')[0],
			isRunning: false,
			timer: null
		}
	},
	onShow() {
		this.init();
		this.startMonitor();
	},
	onHide() { if(this.timer) clearInterval(this.timer); },
	methods: {
		init() {
			const list = uni.getStorageSync('medicine_list') || [];
			const now = new Date().toISOString().split('T')[0];
			this.todayStr = now;
			this.todayList = list.filter(m => now >= m.startDate && now <= m.endDate)
				.map(m => ({ ...m, isTaken: !!(m.history && m.history[now]) }))
				.sort((a, b) => a.time.localeCompare(b.time));
		},
		startMonitor() {
			this.isRunning = true;
			if(this.timer) clearInterval(this.timer);
			this.timer = setInterval(() => {
				const now = new Date();
				const time = `${String(now.getHours()).padStart(2,'0')}:${String(now.getMinutes()).padStart(2,'0')}`;
				const date = now.toISOString().split('T')[0];
				this.todayList.forEach(med => {
					if (med.time === time && !med.isTaken) this.triggerAlarm(med);
				});
			}, 10000);
		},
		triggerAlarm(med) {
			if(this._last === med.id + med.time) return;
			this._last = med.id + med.time;
			if(med.methods.includes('vibrate')) uni.vibrateLong();
			// 语音逻辑在此处扩展...
			uni.showModal({
				title: '🔔 服药提醒',
				content: `药品：${med.name}\n剂量：${med.dosage}`,
				confirmText: '确定服药',
				success: (res) => { if(res.confirm) this.confirmTake(med.id); }
			});
		},
		confirmTake(id) {
			let list = uni.getStorageSync('medicine_list') || [];
			const idx = list.findIndex(m => m.id === id);
			if (idx !== -1) {
				if (!list[idx].history) list[idx].history = {};
				list[idx].history[this.todayStr] = true;
				uni.setStorageSync('medicine_list', list);
				uni.vibrateShort();
				this.init();
			}
		},
		navTo(url) { uni.navigateTo({ url }); }
	}
}
</script>

<style>
.container { background: #F8F9FB; min-height: 100vh; }
.header-box { background: #fff; padding: 60px 20px 20px; border-radius: 0 0 30px 30px; box-shadow: 0 4px 20px rgba(0,0,0,0.05); }
.status-indicator { display: flex; align-items: center; font-size: 12px; color: #999; margin-bottom: 10px; }
.status-indicator.active { color: #52c41a; }
.pulse { width: 8px; height: 8px; background: currentColor; border-radius: 50%; margin-right: 6px; animation: blink 1.5s infinite; }
@keyframes blink { 0% { opacity: 1; transform: scale(1); } 50% { opacity: 0.4; transform: scale(1.2); } 100% { opacity: 1; transform: scale(1); } }
.welcome-row { display: flex; justify-content: space-between; align-items: center; }
.welcome-text { font-size: 24px; font-weight: bold; color: #333; }
.report-entry { background: #EEF2FF; padding: 6px 12px; border-radius: 20px; color: #2C5FF7; font-size: 14px; font-weight: 500; }

.section-head { padding: 25px 20px 15px; display: flex; justify-content: space-between; align-items: baseline; }
.section-title { font-size: 18px; font-weight: bold; color: #444; }
.section-date { font-size: 13px; color: #999; }

.med-card { background: #fff; margin: 0 15px 15px; border-radius: 20px; padding: 20px; display: flex; align-items: center; transition: all 0.3s; }
.med-card.completed { opacity: 0.6; filter: grayscale(1); }
.card-left { display: flex; flex-direction: column; align-items: center; width: 60px; border-right: 1px dashed #EEE; margin-right: 15px; }
.med-time { font-size: 20px; font-weight: 800; color: #2C5FF7; }
.card-body { flex: 1; }
.med-name { font-size: 18px; font-weight: bold; color: #333; display: block; margin-bottom: 5px; }
.dosage { font-size: 14px; color: #666; }
.method-tags { display: flex; margin-top: 8px; }
.tag { font-size: 10px; background: #F0F2F5; color: #888; padding: 2px 6px; border-radius: 4px; margin-right: 6px; }
.check-btn { background: #2C5FF7; color: #fff; font-size: 13px; border-radius: 20px; border: none; padding: 0 15px; }
.check-btn[disabled] { background: #E0E0E0; color: #999; }

.fab-btn { position: fixed; bottom: 40px; left: 50%; transform: translateX(-50%); width: 140px; height: 56px; background: #2C5FF7; border-radius: 30px; display: flex; align-items: center; justify-content: center; color: #fff; box-shadow: 0 10px 25px rgba(44,95,247,0.4); }
.plus { font-size: 28px; font-weight: 300; margin-right: 8px; }
</style>