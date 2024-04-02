<!-- pages/show.vue -->
<template>
	<view class="app flex-rw" :style="'background-color:'+bgc">
		<movable-area :class="'map-area '+mapmode">
			<movable-view direction="all" :class="'map '+refresh+' '+mapmode">
				<view v-for="(e,idx) in events[locIdx][timeIdx[locIdx]]" @click="viewMarker($event)"
					:id="(e.type=='multil'?'event'+e.id:'person'+e.pid)" :class="'marker flex-ct '+e.type" :key="e.id"
					:style="'left:calc('+e.pos[0]+' * 100% / 1440 - 22px);top: calc('+e.pos[1]+' * 100% / 781 - 39px)'">
					<block v-if="e.type=='multil'">
						<view :class="'marker-round '+(e.id==viewId?'':'hide')"></view>
						<text class="marker-count">{{e.persons.length}}</text>
						<view v-if="e.id==viewId&&offsets" v-for="(s,index) in e.persons" @click="viewMarker($event)"
							:id="'person'+s.pid" :key="s.pid" :class="'submarker flex-ct '+s.type+' '+subMarkers"
							:style="'top:'+offsets[index][0]+'px;left:'+offsets[index][1]+'px;'">
							<text class="submarker-txt">{{s.name}}</text>
						</view>
					</block>
					<text class="marker-txt" v-else>{{e.name}}</text>
				</view>
				<image mode="heightFix" :src="'@/static/maps/'+locs[locIdx].id+timeIdx[locIdx]+'.webp'"
					style="filter: brightness(0.95);height: 100%;width: 100%;z-index: 10;" @load="mapLoad"></image>
				<view @click="viewMarker({currentTarget:{id: 'event0'}})" v-show="viewId!=0" class="mask"
					style="z-index: 40;cursor: default;position: absolute;top: 0px;"></view>
			</movable-view>
		</movable-area>
		<view class="timebar flex-rw">
			<view :class="'time flex-ct '+(timeIdx[locIdx]==idx?'selected':'')" @click.stop="timeSelect($event)"
				v-for="(t,idx) in locs[locIdx].time" :key="locs[locIdx].time[idx].name" :id="'time'+idx"
				:style="'width:calc(100% * '+timeLong(locIdx,idx)+' / '+locLong(locIdx)+')'">
				<text :class="refresh">{{t.name}}</text>
				<text
					:class="'time-info '+(timeIdx[locIdx]==idx?'':'hide')+' '+refresh">{{timeShowCalc(t.start)}}-{{timeShowCalc(t.end)}}</text>
			</view>
		</view>
		<view :class="'sidebar mask '+(locMenu||infoMenu?'':'hide')"></view>
		<view :class="'sidebar flex-rw '+(locMenu||infoMenu?'':'hide')">
			<view v-if="!infoId" :class="'list flex-cl '+(locMenu?'':'hide')">
				<text class="title">地区列表</text>
				<view v-for="(loc,idx) in locs" :id="'opt'+idx" :key="loc.name"
					:class="'card flex-rw '+(locIdx==idx?'selected':'')" @click="locMenuShow($event)">
					<view class="cl-gp flex-cl" style="margin-left: 15px;">
						<text class="card-txt">{{loc.name}}</text>
						<text v-if="locIdx==idx" style="color: #c87800">当前</text>
					</view>
				</view>
			</view>
			<view v-else :class="'info flex-cl '+(infoMenu?'':'hide')">
				<view class="info-card flex-rw">
					<image v-if="infoId" mode="aspectFit" class="info-photo"
						:src="'@/static/photos/'+infoId+'.webp'"></image>
					<view class="flex-cl">
						<text class="info-name">{{persons[infoId].name}}</text>
						<text>{{locs[locIdx].name}}</text>
						<text>{{persons[infoId].txt1}}</text>
						<text>{{persons[infoId].txt2}}</text>
					</view>
				</view>
				<rich-text class="info-txt" :nodes="persons[infoId].desc"></rich-text>
			</view>
		</view>
		<view :class="'toolbar '+refresh" v-show="!locMenu&&!infoMenu">
			<view class="toolbar-item flex-ct" @click="feedback">
				
				<text class="toolbar-txt">反馈建议</text>
			</view>
		</view>
		<view class="loc-info flex-rw" v-show="!locMenu&&!infoMenu">
			<view :class="'cl-gp flex-cl '+(locEvent?'hide ':'')+refresh" style="margin-right: 15px;transition: 0.25s;">
				<text style="color: #FFFFFF;width: 100%;text-align: right;font-size: 20px;">{{locs[locIdx].name}}</text>
				<text style="color: #F8F8F8;">{{"探索度 "}}<text
						:style="'color: '+(visited==100?'#e5d309':'#F8F8F8')">{{visited+"%"}}</text></text>
			</view>
			<image @click="locMenuShow()" class="loc-icon" src="@/static/icons/compass.svg"></image>
		</view>
		<view :class="'event-area flex-cl '+refresh">
			<view @click="eventSelect($event)" :id="'locEvent'+idx" v-for="(e,idx) in locEvents[locIdx]" :key="e.name"
				class="event-unit flex-rw">
				<view class="event-card flex-ct">
					<image :src="'@/static/icons/events-'+e.type+'-icon.svg'" class="event-icon"></image>
					<view class="event-txt-box flex-ct">
						<text class="event-txt">{{e.name}}</text>
					</view>
				</view>
				<view class="event-cardend"></view>
			</view>
		</view>
		<view @click="eventSelect()" :class="'locEvent-area flex-ct '+(locEvent?'':'hide')">
			<view @click.stop class="locEvent flex-ct flex-cl">
				<text class="locEvent-title">{{locEvents[locIdx][locEventIdx].name}}</text>
				<text class="locEvent-subtext">{{locEvents[locIdx][locEventIdx].subtext}}</text>
				<view class="locEvent-txt">
					<image class="locEvent-pic" mode="aspectFill" :src="'@/static/pictures/'+locIdx+locEventIdx+'.webp'">
					</image>
					<rich-text class="locEvent-desc" :nodes="locEvents[locIdx][locEventIdx].text"
						:style="'max-height: calc(100% - '+(locEvents[locIdx][locEventIdx].subtext?110:90)+'px);'">
					</rich-text>
				</view>
			</view>
		</view>
		<view @click="locMenuShow()" v-show="locMenu||infoMenu" class="mask"></view>
	</view>
</template>

<script>
	const version = "22w32e"
	var visitedP = [{}, {}, {}],
		visitedE = [{}, {}, {}]
	import {
		countData,
		personData,
		locData,
		eventData,
		locEventData
	} from '@/static/data.js'
	var timer, timetick = 0;
	function setTimer(t) {
		clearTimer();
		timer = setInterval(() => timetick += 50, 50);
	}
	function clearTimer() {
		clearTimeout(timer);
		timetick = 0;
	}
	export default {
		onLoad() {
			window.oncontextmenu = () => {
				this.locMenuShow()
				return false
			}
			if (uni.getStorageSync("version") != version) {
				//uni.clearStorageSync()
				uni.setStorageSync("version", version)
			}
			this.locIdx = uni.getStorageSync("locIdx") ? uni.getStorageSync("locIdx") : this.locIdx
			this.timeIdx = uni.getStorageSync("timeIdx") ? uni.getStorageSync("timeIdx") : this.timeIdx
			visitedE = uni.getStorageSync("visitedE") ? uni.getStorageSync("visitedE") : visitedE
			visitedP = uni.getStorageSync("visitedP") ? uni.getStorageSync("visitedP") : visitedP
			this.exploreCalc()
			this.windowResizeCallback({
				size: {
					windowHeight: uni.getWindowInfo().windowHeight,
					windowWidth: uni.getWindowInfo().windowWidth
				}
			})
			uni.onWindowResize(this.windowResizeCallback)
		},
		data() {
			return {
				viewId: 0,
				locIdx: 0,
				timeIdx: [0, 0, 0],
				infoMenu: false,
				infoId: '',
				visited: 0,
				locMenu: false,
				locEvent: false,
				mapmode: "a",
				locEventIdx: "0",
				subMarkers: 'hide',
				bgc: '#FFF',
				refresh: 'hide',
				offsets: []
			}
		},
		computed: {
			persons() {
				return personData
			},
			locs() {
				return locData
			},
			events() {
				return eventData
			},
			locEvents() {
				return locEventData
			}
		},
		methods: {
			viewMarker(e) {
				var id = Number(e.currentTarget.id.replace("event", "")),
					count, i
				if (id) {
					this.offsets = []
					let eventList = this.events[this.locIdx][this.timeIdx[this.locIdx]]
					for (i in eventList)
						if (eventList[i].id == id) {
							count = eventList[i].persons.length
							break
						}
					var angle = 360 / count;
					var radius = 50;
					for (i = 0; i < count; i++) {
						this.offsets.push([Math.round(Math.cos(angle * i * Math.PI / 180) * radius) - 15, Math.round(Math
							.sin(angle * i * Math.PI / 180) * radius)]);
					}
					this.viewId = id
					setTimeout(() => this.subMarkers = "", 100)
				} else if (id == 0) {
					this.subMarkers = "hide"
					setTimeout(() => this.viewId = id, 250)
				} else {
					id = e.currentTarget.id.replace("person", "")
					this.infoMenu = true
					console.log(id)
					visitedP[this.locIdx][id] = true
					this.exploreCalc()
					this.infoId = id
				}
			},
			eventSelect(e) {
				if (e) {
					var id = e.currentTarget.id.replace("locEvent", "")
					this.locEventIdx = id
					this.locEvent = true
					visitedE[this.locIdx][id] = true
					setTimeout(() => this.exploreCalc(), 250)
				} else {
					this.locEvent = false
				}
			},
			timeSelect(e) {
				var id = Number(e.currentTarget.id.replace("time", ""))
				if (e && this.timeIdx[this.locIdx] != id) {
					this.refresh = 'hide'
					setTimer()
					setTimeout(() => {
						this.timeIdx[this.locIdx] = id;
						this.$forceUpdate()
					}, 150)
				}
			},
			locMenuShow(e) {
				if (e) {
					var id = Number(e.currentTarget.id.replace("opt", ""))
					if (this.locIdx != id) {
						this.refresh = 'hide'
						this.locEventIdx = "0"
						setTimer()
						setTimeout(() => {
							this.locIdx = id;
							this.exploreCalc()
						}, 150)

					}
				} else
				if (this.locMenu)
					this.locMenu = false
				else
				if (this.infoMenu) {
					this.infoMenu = false
					setTimeout(() => this.infoId = "", 250)
				} else
					setTimeout(() => this.locMenu = true, 100)
			},
			exploreCalc() {
				uni.setStorageSync("visitedE", visitedE)
				uni.setStorageSync("visitedP", visitedP)
				let a = Object.keys(visitedE[this.locIdx]).length + Object.keys(visitedP[this.locIdx]).length,
					b = countData[this.locIdx] + locEventData[this.locIdx].length
				this.visited = Math.round(a * 100 / b);
			},
			locLong(i) {
				return this.locs[i].time.slice(-1)[0].end - this.locs[i].time[0].start
			},
			timeLong(i, t) {
				return this.locs[i].time[t].end - this.locs[i].time[t].start
			},
			feedback() {
				window.open("https://support.qq.com/product/422563")
			},
			timeShowCalc(t) {
				return t < 0 ? -t + 'BC' : t
			},
			mapLoad() {
				if (timetick < 300)
					setTimeout(() => {
						this.refresh = '';
					}, 300 - timetick)
				else
					this.refresh = ''
				clearTimer()
				uni.setStorageSync("locIdx", this.locIdx)
				uni.setStorageSync("timeIdx", this.timeIdx)
			},
			windowResizeCallback(res) {
				if (res.size.windowWidth / res.size.windowHeight <= 1440 / 781)
					this.mapmode = "a"
				else
					this.mapmode = "b"
			}
		}
	}
</script>

<style>
	.toolbar {
		position: absolute;
		top: 12.5px;
		right: 20px;
	}

	.toolbar-item {
		background-color: #48484850;
		border-radius: 20px;
		padding: 5px;
		padding-top: 3px;
		padding-bottom: 3px;
		transition: 0.25s;
	}

	.toolbar-item:hover {
		opacity: 0.8;
	}

	.toolbar-txt {
		color: #FFFFFF;
		font-size: 15px;
		font-weight: 100;
	}

	.locEvent-subtext {
		color: #484848EE;
		font-weight: 50;
	}
	.locEvent-desc {
		line-height: 1.7em;
	}
	.locEvent-txt::-webkit-scrollbar {
		width: 5px;
	}
	.locEvent-txt::-webkit-scrollbar-thumb {
		border-radius: 10px;
		box-shadow: inset 0 0 5px rgba(200, 200, 200, 0.8);
	}
	.locEvent-pic {
		margin: 5px;
		width: 110px;
		height: 140px;
		overflow: hidden;
		float: left;
		/* margin-right: 10px; */
	}
	.locEvent-txt {
		width: calc(100% - 30px);
		margin: 15px;
		overflow-y: auto;
		margin-top: 5px;
	}
	.locEvent-title {
		font-size: 20px;
		margin: 15px;
		margin-bottom: 5px;
	}
	.locEvent {
		background-color: #FFFFFFDD;
		width: 650px;
		position: relative;
		bottom: 30px;
		height: 360px;
		border-radius: 10px;
		justify-content: flex-start;
	}
	.locEvent-area {
		position: absolute;
		top: 0px;
		left: 0px;
		width: 100vw;
		height: 100vh;
		z-index: 150;
		transition: opacity 0.25s;
	}
	.event-icon {
		height: 40px;
		width: 40px;
	}
	.event-txt-box {
		height: 40px;
	}
	.event-cardend {
		margin-top: 12.5px;
		width: 0px;
		height: 0px;
		border: 20px solid transparent;
		border-left: 15px solid #FFFFFF60;
		cursor: pointer;
	}
	.event-card {
		margin-top: 12.5px;
		background-color: #FFFFFF60;
		padding: 5px;
		height: 30px;
		width: 170px;
		justify-content: flex-start;
		cursor: pointer;
	}
	.event-unit {
		transition: 0.25s;
	}
	.event-unit:hover {
		transform: scale(1.05);
	}
	.event-area {
		position: absolute;
		left: 0px;
		top: 0px;
		transition: 0.25s;
	}
	.info-txt {
		max-height: calc(100% - 170px);
		overflow-y: auto;
	}
	.info-txt::-webkit-scrollbar {
		width: 5px;
	}
	.info-txt::-webkit-scrollbar-thumb {
		border-radius: 10px;
		box-shadow: inset 0 0 5px rgba(255, 255, 255, 0.8);
	}
	.info-photo {
		height: 108px;
		width: 90px;
		margin-right: 20px;
	}
	.info-card {
		margin-top: 20px;
		padding: 10px;
	}

	.info-name {
		font-size: 25px;
	}

	.info {
		width: calc(100% - 50px);
	}

	.marker.litterateur {
		background-image: url(@/static/icons/person-litterateur-marker.svg);
	}

	.submarker.litterateur {
		background-image: url(@/static/icons/person-litterateur-submarker.svg);
	}

	.marker.artist {
		background-image: url(@/static/icons/person-artist-marker.svg);
	}

	.submarker.artist {
		background-image: url(@/static/icons/person-artist-submarker.svg);
	}

	.marker.naturalist {
		background-image: url(@/static/icons/person-naturalist-marker.svg);
	}

	.submarker.naturalist {
		background-image: url(@/static/icons/person-naturalist-submarker.svg);
	}

	.marker.industry {
		background-image: url(@/static/icons/person-industry-marker.svg);
	}

	.marker.politician {
		background-image: url(@/static/icons/person-politician-marker.svg);
	}

	.submarker.politician {
		background-image: url(@/static/icons/person-politician-submarker.svg);
	}

	.marker.military {
		background-image: url(@/static/icons/person-military-marker.svg);
	}

	.submarker.military {
		background-image: url(@/static/icons/person-military-submarker.svg);
	}

	.marker.multil {
		background-image: url(@/static/icons/mutil-markers.svg);
	}

	.marker-round {
		height: 140px;
		width: 140px;
		position: absolute;
		border-radius: 50%;

		background: radial-gradient(hsla(0, 0%, 100%, .26) 0, rgba(255, 255, 255, .23) 96%);
		border: 1px solid rgba(255, 255, 255, .33);
		transition: all .3s ease;
		z-index: 40;
		/* cursor: grab; */
	}

	.marker-count {
		color: #FFF;
		position: absolute;
		top: 9px;
		font-weight: 50;
	}

	.submarker-txt {
		position: absolute;
		top: 45px;
		background-color: #FFFFFF60;
		font-size: 12.5px;
		padding: 5px;
		border-radius: 5px;
		transition: 0.5s;
		white-space: nowrap;
	}

	.marker-txt {
		position: absolute;
		top: 45px;
		background-color: #FFFFFF60;
		font-size: 12.5px;
		padding: 5px;
		border-radius: 5px;
		transition: 0.5s;
		white-space: nowrap;
	}

	.submarker {
		z-index: 60;
	}

	.marker.multil:hover,
	.marker.multil:active {
		opacity: 1;
		filter: brightness(1);
	}

	.marker:active,
	.submarker:active {
		filter: brightness(0.95);
		/* opacity: 0.6; */
	}

	.marker.multil:hover {
		opacity: 1;
	}

	.marker:hover,
	.submarker:hover {
		opacity: 0.8;
	}

	.marker,
	.submarker {
		width: 44px;
		height: 50px;
		position: absolute;
		background-position: 50%;
		z-index: 50;
		cursor: default;
		transition: 0.25s;
	}

	.mask {
		opacity: 0;
	}

	.sidebar.hide {
		opacity: 1;
	}

	.sidebar.mask.hide {
		opacity: 0.5;
	}

	.sidebar.hide,
	.sidebar.mask.hide {
		right: -350px;
		/* display: none; */
	}

	.loc-info {
		position: absolute;
		right: 20px;
		bottom: 20px;
		cursor: pointer;
	}

	.loc-icon {
		height: 40px;
		width: 40px;
	}

	.card.selected:hover {
		padding: 10px;
		background-color: #FFFFFF54;
	}

	.card.selected {
		padding: 10px;
		background-color: #FFFFFF64;
	}

	.card:hover {
		padding: 7px;
		background-color: #FFFFFF54;
	}

	.card {
		margin: 7px;
		width: 100%;
		border-radius: 7.5px;
		background-color: #FFFFFF44;
		cursor: pointer;
		height: 60px;
		padding: 5px;
		transition: 0.25s;
	}

	.card-txt {
		font-size: 25px;
		opacity: 0.8;
	}

	.icon {
		height: 100px;
		width: 100px;
	}

	.list {
		position: relative;
		width: calc(100% - 90px);
		height: calc(100% - 100px);
		top: 10px;
		justify-content: flex-start;
		align-items: center;
		padding: 40px;
		transition: 0.5s;
	}

	.map-area {
		position: absolute;
		background-color: #FFF;
	}

	.map-area.a {
		left: calc((100vw - (100vh * 1440 / 781)));
		height: 100vh;
		width: calc(2 * (100vh * 1440 / 781) - 100vw);
	}

	.map-area.b {
		top: calc((100vh - (100vw / 1440 * 781)));
		width: 100vw;
		height: calc(2 * (100vw / 1440 * 781) - 100vh);
	}

	.map {
		position: absolute;
		transition: 0.25s;
	}

	.map.a {
		left: calc(((100vh * 1440 / 781) - 100vw) / 2);
		height: 100vh;
		width: calc(100vh * 1440 / 781);
	}

	.map.b {
		top: calc(((100vw / 1440 * 781) - 100vh) / 2);
		width: 100vw;
		height: calc(100vw / 1440 * 781);
	}

	.time-info {
		position: absolute;
		bottom: 60px;
		background-color: #FFFFFF60;
		font-size: 12.5px;
		padding: 5px;
		border-radius: 5px;
		transition: 0.5s;
	}

	.time.selected {
		background-color: #FFFFFF90;
	}

	.time:hover {
		background-color: #FFFFFF60;
	}

	.time {
		height: 100%;
		transition: 0.25s;
	}

	.timebar {
		position: absolute;
		height: 50px;
		background-color: #FFFFFFA0;
		width: 75%;
		bottom: 15px;
	}

	.sidebar.mask {
		opacity: 0.5;
	}

	.sidebar {
		position: absolute;
		justify-content: center;
		height: 100vh;
		width: 350px;
		right: 0px;
		opacity: 1;
		transition: 0.1s;
		z-index: 100;
	}

	.title {
		font-size: 30px;
		margin: 10px;
	}

	.cl-gp {
		height: 100%;
		justify-content: center;
	}

	p {
		color: #484848;
		padding: 5px;
		font-size: 17.5px;
	}
</style>
