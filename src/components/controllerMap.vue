<template>
	<div class="wrap">
		<div id="container-map"></div>
		<!-- <div class="serch_box">
      <input type="text" id="serch_input" ref="serch_input" @keydown.enter="autoInput" v-model="serch">
    </div> -->
		<!-- <div class="input-card">
        <div class="input-item">
          <input type="button" class="btn" value=""/>
          <input type="button" class="btn" value="摄像头"/>
          <input type="button" class="btn" value="雷达"/>
          <input type="button" class="btn" value="信号灯"/>
          <input type="button" class="btn" value="诱导屏"/>
        </div>
    </div> -->

		<div class="controllerBtn">
			<div @click="showCar">
				<span>无人机</span>
			</div>
			<div @click="showWebcam">
				
				<span>摄像头</span>
			</div>
			<div @click="showSemaphore">
				
				<span>灯</span>
			</div>
		</div>
		<!-- 实时视频弹出层 -->
		<div class="videoMask" :class="videoVisible ? 'videoShow' : ''">
			<div class="dialogBox">
				<div class="title">
					<span>实时视频</span>
					<i class="el-icon-close" @click="closeVideo">X</i>
				</div>
				<div class="content">
					<div>视频1</div>
					<div>视频2</div>
					<div>视频3</div>
					<div>视频4</div>
				</div>
			</div>
		</div>
		<!-- 摄像头弹出层 -->
		<div class="cameraMask" :class="cameraVisible ? 'cameraShow' : ''">
			<div class="dialogBox">
				<div class="title">
					<span>实时视频</span>
					<i class="el-icon-close" @click="cameraVideo"></i>
				</div>
				<div class="content">
					<div>视频1</div>
					<div>视频2</div>
					<div>视频3</div>
					<div>视频4</div>
				</div>
			</div>
		</div>
		<!-- 信号灯弹出层 -->
		<div class="lightMask" :class="lightVisible ? 'lightShow' : ''">
			<div class="dialogBox">
				<div class="title">
					<span>实时视频</span>
					<i class="el-icon-close" @click="lightClose"></i>
				</div>
				<div class="content">
					<div class="imgTable">
						<div class="img">
							<img src="static/fly_second.svg" alt="信号灯" />
						</div>
						<div class="table">
							<section class="info">
								<div class="tableTitle">
									<span class="label">相位名称</span>
									<span class="content">相位时长</span>
									<span class="status">状态</span>
								</div>

								<div>
									<span class="label">南向东</span>
									<span class="content">20</span>
									<span class="status" style="color: #e5574c;">红灯</span>
								</div>
								<div>
									<span class="label">东向南</span>
									<span class="content">20</span>
									<span class="status" style="color: #e38f1b;">黄</span>
								</div>
								<div>
									<span class="label">南向北</span>
									<span class="content">20</span>
									<span class="status">绿灯</span>
								</div>
								<div>
									<span class="label">北向南</span>
									<span class="content">20</span>
									<span class="status">绿灯</span>
								</div>
								<div>
									<span class="label">南向东</span>
									<span class="content">20</span>
									<span class="status">红灯</span>
								</div>
								<div>
									<span class="label">东向南</span>
									<span class="content">20</span>
									<span class="status">黄</span>
								</div>
								<div>
									<span class="label">南向北</span>
									<span class="content">20</span>
									<span class="status">绿灯</span>
								</div>
								<div>
									<span class="label">北向南</span>
									<span class="content">20</span>
									<span class="status">绿灯</span>
								</div>
							</section>
							<div class="switch">
								<span>锁定相位</span>
								<el-switch
									v-model="value"
									active-color="#585C62"
									inactive-color="#ff4949"
								>
								</el-switch>
							</div>
							<div class="switch">
								<span>全红控制</span>
								<el-switch
									v-model="value"
									active-color="#585C62"
									inactive-color="#ff4949"
								>
								</el-switch>
							</div>
						</div>
					</div>
					<el-divider></el-divider>
					<div class="footerBtn">
						<el-button type="primary">主要按钮</el-button>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
import AMapLoader from "@amap/amap-jsapi-loader";

export default {
	props: {},
	data() {
		return {
			// map: "",
			// clientHeight: "",
			// markers: [],
			// serch: "",

			value: "",
			map: null,
			carMarker: null, //
			carMarkerShow: false, // 显示隐藏
			webcamMarker: null,
			webcamMarkerShow: false,
			semaphoreMarker: null,
			semaphoreMarkerShow: false,
			videoVisible: false,
			cameraVisible: false,
			lightVisible: false,
			ws: null,
			timeout: 60000, //1分钟发一次心跳
			timeoutObj: null,
			serverTimeoutObj: null,
			lockReconnect: false, //避免ws重复连接
			wsUrl: "ws://192.168.8.100:8081/ws",
			historyInterval: null,
			historyIntervalFirst: null,
			historyIntervalSecond: null,
			historyIntervalThird: null,
		};
	},
	created() {},
	mounted() {
		window["videoShowFn"] = (param) => {
			this.videoShowFn(param);
		};
		window["cameraShowFn"] = (param) => {
			this.cameraShowFn(param);
		};
		window["lightShowFn"] = (param) => {
			this.lightShowFn(param);
		};
		// 跳转到轨迹回放页面
		window["toTrackPlaybackFn"] = (param) => {
			this.toTrackPlaybackFn(param);
		};

		this.initOrbit();
		// this.createWebSocket(this.wsUrl);
	},
	beforeDestroy() {
		clearInterval(this.historyInterval);
		clearInterval(this.historyIntervalFirst);
		clearInterval(this.historyIntervalSecond);
		clearInterval(this.historyIntervalThird);
		// this.timeoutObj = null
		// this.serverTimeoutObj = null
		// this.ws.close();
		this.historyInterval = null;
		this.historyIntervalFirst = null;
		this.historyIntervalSecond = null;
		this.historyIntervalThird = null;
	},
	methods: {
		videoShowFn() {
			this.videoVisible = true;
		},
		cameraShowFn() {
			this.cameraVisible = true;
		},
		lightShowFn() {
			this.lightVisible = true;
		},
		// 关闭弹出
		closeVideo() {
			this.videoVisible = false;
		},
		// 关闭弹出
		cameraVideo() {
			this.cameraVisible = false;
		},
		// 关闭弹出
		lightClose() {
			this.lightVisible = false;
		},
		// 跳转到轨迹回放
		toTrackPlaybackFn() {
			this.$router.push({
				path: "/trackPlayback/index",
			});
		},
		initOrbit() {
			AMapLoader.load({
				key: "10e903bafe1cff3b9f952d3ba5b3cbb7", // 申请好的Web端开发者Key，首次调用 load 时必填
				version: "2.0", // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
				plugins: ["AMap.ToolBar", "AMap.Scale"], // 需要使用的的插件列表，如比例尺'AMap.Scale'等
				AMapUI: {
					// 是否加载 AMapUI，缺省不加载
					version: "1.1", // AMapUI 缺省 1.1
					plugins: [], // 需要加载的 AMapUI ui插件
				},
				Loca: {
					// 是否加载 Loca， 缺省不加载
					version: "2.0", // Loca 版本，缺省 1.3.2
				},
			}).then((AMap) => {
				//初始化地图对象，加载地图
				this.map = new AMap.Map("container-map", {
					resizeEnable: true, //是否监控地图容器尺寸变化
					rotateEnable: true,
					pitchEnable: true,
					zoom: 17,
					pitch: 55, // 地图俯仰角度，有效范围 0 度- 83 度
					rotation: -15,
					viewMode: "3D", //开启3D视图,默认为关闭
					zooms: [2, 20],
					// center: [116.368904, 39.923423],
					// mapStyle:'amap://styles/blue',
					center: [116.515127, 39.781011],
				});
				this.map.addControl(new AMap.ToolBar());
				this.map.addControl(new AMap.Scale());
				// // 调用信息marker方法
				// this.carMarker(AMap, this.map);
				this.initHistory();
				// this.historyFirst();
				// this.historySecond();
				// this.historyThird();
			});
		},

		/*
		 * 点击显示位置信息
		 * @param {*} AMap
		 * */
		showCar() {
			if (this.carMarkerShow) {
				this.map.remove(this.carMarker);
				this.carMarkerShow = !this.carMarkerShow;
				return false;
			}

			AMapLoader.load({
				//可多次调用load
				// plugins: ["AMap.MapType"],
			})
				.then((AMap) => {
					const thisMap = this.map;
					// 后台获取marker点信息
					var lnglats = [
						[116.513383, 39.780172],
						[116.513935, 39.77933],
						[116.514275, 39.777622],
					];
					// 后台获取car 提示信息
					var carTips = [
						[116.514158, 39.780575],
						[116.51509, 39.781048],
						[116.516116, 39.779858],
					];
					// 自定义弹出窗体
					const makeContentFn = (data) => {
						console.log(data);
						return `<div class="carWrap">
              <div class="carTitle">信息</div>
              <div class="carInfoWrap">
                <div class="carImgWrap">
                  <img src="static/fly.svg" alt="">
                </div>
                <section class="info">
                  <div>
                    <span class="label">编号：</span>
                    <span class="content">P12138</span>
                  </div>
                  <div>
                    <span class="label">号牌：</span>
                    <span class="content">京 A3Q185</span>
                  </div>
                  <div>
                    <span class="label">平均车速：</span>
                    <span class="content">30km/h</span>
                  </div>
                  <div>
                    <span class="label">行车里程：</span>
                    <span class="content">16320.7km</span>
                  </div>
                  <div>
                    <span class="label">自动驾驶级别：</span>
                    <span class="content">L3</span>
                  </div>
                </section>
              </div>
              <div class="operateImg">
                <span onclick="(function () {
                  this.toTrackPlaybackFn();
                })();">
                <img src="static/fly.svg" alt="轨迹">
                </span>
                <span onclick="(function () {
                  this.videoShowFn(true);
                })();">
                  <img src="static/fly.svg" alt="录像">
                </span>
              </div>
            </div>`;
					};
					// 提示告警弹出窗体内容
					const carTipsContentFn = (data) => {
						console.log(data);
						return `<div >
							云端-紧急制动
						</div>`;
					};
					// infowidnow 窗体配置
					var infoWindow = new AMap.InfoWindow({
						offset: new AMap.Pixel(0, -30),
					});

					// 定义car-icon
					var carIcon = new AMap.Icon({
						size: new AMap.Size(25, 34),
						image: "static/fly.svg",
						imageSize: new AMap.Size(24, 26),
						imageOffset: new AMap.Pixel(0, 3),
					});

					// 覆盖物数组
					var marker = [];
					for (var i = 0; i < lnglats.length; i++) {
						marker[i] = new AMap.Marker({
							position: lnglats[i],
							map: this.map,
							// 图标尺寸
							size: new AMap.Size(25, 34),
							// 将一张图片的地址设置为 icon
							icon: carIcon,
							// 设置了 icon 以后，设置 icon 的偏移量，以 icon 的 [center bottom] 为原点
							offset: new AMap.Pixel(-13, -30),
						});

						marker[i].content = makeContentFn(lnglats);
						marker[i].on("click", markerClick);
						// marker[i].on("mouseover", markerClick);
						// 默认触发
						// marker.emit("click", { target: marker });
						// 将 markers 添加到地图
						this.map.add(marker[i]);
					}

					// 定义car-tips-icon
					var carTipsIcon = new AMap.Icon({
						size: new AMap.Size(25, 34),
						image: "static/fly_black.svg",
						imageSize: new AMap.Size(24, 26),
						imageOffset: new AMap.Pixel(0, 3),
					});
					var carMarkerTips = [];
					for (var j = 0; j < carTips.length; j++) {
						carMarkerTips[j] = new AMap.Marker({
							position: carTips[j],
							map: this.map,
							// 图标尺寸
							size: new AMap.Size(25, 34),
							// 将一张图片的地址设置为 icon
							icon: carTipsIcon,
							// 设置了 icon 以后，设置 icon 的偏移量，以 icon 的 [center bottom] 为原点
							offset: new AMap.Pixel(-13, -30),
						});

						carMarkerTips[j].content = carTipsContentFn(carTips);
						carMarkerTips[j].on("mouseover", markerClick);
						carMarkerTips[j].on("mouseout", markerHide);
						// 默认触发
						// marker.emit("click", { target: marker });
						// 将 markers 添加到地图
						this.map.add(carMarkerTips[j]);
					}
					// 数组赋值
					this.carMarker = marker.concat(carMarkerTips);
					// 窗体点击事件
					function markerClick(e) {
						// console.log(e);
						infoWindow.setContent(e.target.content);
						infoWindow.open(thisMap, e.target.getPosition());
					}
					// 鼠标离开事件
					function markerHide(e) {
						// console.log(e);
						infoWindow.setContent(e.target.content);
						infoWindow.open(thisMap, e.target.getPosition());
					}

					// 已经显示了
					this.carMarkerShow = !this.carMarkerShow;
				})
				.catch((e) => {
					console.error(e);
				});
		},
		/*
		 * 点击显示摄像头位置信息
		 * @param {*} AMap
		 * */
		showWebcam() {
			if (this.webcamMarkerShow) {
				this.map.remove(this.webcamMarker);
				this.webcamMarkerShow = !this.webcamMarkerShow;
				return false;
			}

			AMapLoader.load({
				//可多次调用load
				// plugins: ["AMap.MapType"],
			})
				.then((AMap) => {
					// const thisMap = this.map;
					// 后台获取marker点信息
					var lnglats = [
						[116.512874, 39.781626],
						[116.514222, 39.782176],
						[116.511633, 39.782538],
					];

					// 定义car-icon
					var carIcon = new AMap.Icon({
						size: new AMap.Size(25, 34),
						image: "static/fly_first.svg",
						imageSize: new AMap.Size(24, 26),
						imageOffset: new AMap.Pixel(0, 3),
					});

					// 覆盖物数组
					var marker = [];
					for (var i = 0; i < lnglats.length; i++) {
						marker[i] = new AMap.Marker({
							position: lnglats[i],
							map: this.map,
							// 图标尺寸
							size: new AMap.Size(25, 34),
							// 将一张图片的地址设置为 icon
							icon: carIcon,
							// 设置了 icon 以后，设置 icon 的偏移量，以 icon 的 [center bottom] 为原点
							offset: new AMap.Pixel(-13, -30),
						});

						// marker[i].content = makeContentFn(lnglats);
						marker[i].on("click", markerClick);
						// marker[i].on("mouseover", markerClick);
						// 默认触发
						// marker.emit("click", { target: marker });
						// 将 markers 添加到地图
						this.map.add(marker[i]);
					}

					// 数组赋值
					this.webcamMarker = marker;
					// 窗体点击事件
					function markerClick() {
						window.cameraShowFn(true);
					}

					// 已经显示了
					this.webcamMarkerShow = !this.webcamMarkerShow;
				})
				.catch((e) => {
					console.error(e);
				});
		},
		/*
		 * 点击显示信号灯位置信息
		 * @param {*} AMap
		 * */
		showSemaphore() {
			if (this.semaphoreMarkerShow) {
				this.map.remove(this.semaphoreMarker);
				this.semaphoreMarkerShow = !this.semaphoreMarkerShow;
				return false;
			}

			AMapLoader.load({
				//可多次调用load
				// plugins: ["AMap.MapType"],
			})
				.then((AMap) => {
					// const thisMap = this.map;
					// 后台获取marker点信息
					var lnglats = [
						[116.509486, 39.780772],
						[116.512249, 39.78137],
						[116.509055, 39.779877],
					];

					// 定义car-icon
					var carIcon = new AMap.Icon({
						size: new AMap.Size(25, 34),
						image: "static/fly_second.svg",
						imageSize: new AMap.Size(24, 26),
						imageOffset: new AMap.Pixel(0, 3),
					});

					// 覆盖物数组
					var marker = [];
					for (var i = 0; i < lnglats.length; i++) {
						marker[i] = new AMap.Marker({
							position: lnglats[i],
							map: this.map,
							// 图标尺寸
							size: new AMap.Size(25, 34),
							// 将一张图片的地址设置为 icon
							icon: carIcon,
							// 设置了 icon 以后，设置 icon 的偏移量，以 icon 的 [center bottom] 为原点
							offset: new AMap.Pixel(-13, -30),
						});

						// marker[i].content = makeContentFn(lnglats);
						marker[i].on("click", markerClick);
						// marker[i].on("mouseover", markerClick);
						// 默认触发
						// marker.emit("click", { target: marker });
						// 将 markers 添加到地图
						this.map.add(marker[i]);
					}

					// 数组赋值
					this.semaphoreMarker = marker;
					// 窗体点击事件
					function markerClick() {
						window.lightShowFn(true);
					}

					// 已经显示了
					this.semaphoreMarkerShow = !this.semaphoreMarkerShow;
				})
				.catch((e) => {
					console.error(e);
				});
		},
		createWebSocket(url) {
			try {
				if ("WebSocket" in window) {
					this.ws = new WebSocket(url);
				}
				this.initEventHandle();
			} catch (e) {
				this.reconnect(url);
				console.log(e);
			}
		},
		initEventHandle() {
			const self = this;
			this.ws.onclose = () => {
				// self.reconnect(this.wsUrl);
				console.log("ws连接关闭!" + new Date().toLocaleString());
			};
			this.ws.onerror = () => {
				self.reconnect(this.wsUrl);
				console.log("ws连接错误!");
			};
			this.ws.onopen = () => {
				self.reset().start(); //心跳检测重置
				console.log("ws连接成功!" + new Date().toLocaleString());
				// this.ws.send(1);
				this.getMarkerInMap();
			};
			this.ws.onmessage = (event) => {
				//如果获取到消息，心跳检测重置
				self.reset().start(); //拿到任何消息都说明当前连接是正常的
				console.log("ws收到消息啦:" + event.data);
			};
		},
		reconnect(url) {
			if (this.lockReconnect) {
				this.ws.close();
				return;
			}
			this.lockReconnect = true;
			setTimeout(() => {
				//没连接上会一直重连，设置延迟避免请求过多
				this.createWebSocket(url);
				this.lockReconnect = false;
			}, 2000);
		},
		reset() {
			clearTimeout(this.timeoutObj);
			clearTimeout(this.serverTimeoutObj);
			return this;
		},
		start() {
			var self = this;
			console.log(this.timeout);
			this.timeoutObj = setTimeout(() => {
				//这里发送一个心跳，后端收到后，返回一个心跳消息，
				//onmessage拿到返回的心跳就说明连接正常
				self.reconnect(self.wsUrl);
				self.ws.send("ping");
				console.log(self.timeout);
				// console.log("ping!")
				self.serverTimeoutObj = setTimeout(() => {
					//如果超过一定时间还没重置，说明后端主动断开了
					self.ws.close(); //如果onclose会执行reconnect，我们执行ws.close()就行了.如果直接执行reconnect 会触发onclose导致重连两次
				}, self.timeout);
			}, this.timeout);
		},
		// 点击获取对应类型覆盖物坐标
		getMarkerInMap() {
			const param = {
				deviceKeyList: [
					{
						deviceKey: "GEO_DEV_VEH_KEY:",
					},
					{
						deviceKey: "GEO_DEV_CAMERA_KEY:",
					},
				],
				longitude: "116.515127",
				dimension: "39.781011",
			};
			console.log(param);
			this.ws.send(JSON.stringify(param));
		},
		initHistory() {
			const self = this;
			AMapLoader.load({})
				.then((AMap) => {
					// JSAPI2.0 使用覆盖物动画必须先加载动画插件 ['AMap.ToolBar','AMap.Driving']
					AMap.plugin(
						["AMap.ToolBar", "AMap.MoveAnimation", "AMap.ControlBar"],
						function () {
							var marker,
								lineArr = [
									[116.514114, 39.780368],
									[116.514033, 39.780478],
									[116.514263, 39.780604],
									[116.515112, 39.781041],
									[116.516406, 39.779486],
									[116.518985, 39.778849],
									[116.519313, 39.779635],
									[116.519336, 39.779733],
									[116.519302, 39.780329],
									[116.519219, 39.781011],
									[116.51868, 39.781831],
									[116.517263, 39.783606],
									[116.516634, 39.783383],
									[116.512574, 39.781414],
									[116.513369, 39.780185],
									[116.513957, 39.780466],
								];

							// 3D 工具
							var controlBar = new AMap.ControlBar({
								position: {
									right: "10px",
									top: "10px",
								},
							});
							controlBar.addTo(self.map);

							var toolBar = new AMap.ToolBar({
								position: {
									right: "40px",
									top: "110px",
								},
							});
							toolBar.addTo(self.map);

							// 信息窗体
							// 自定义弹出窗体
							const makeContentFn = (data) => {
								console.log(data);
								return `<div class="carWrap">
                  <div class="carTitle">信息</div>
                  <div class="carInfoWrap">
                    <div class="carImgWrap">
                      <img src="img/mapMode/car-img.jpeg" alt="">
                    </div>
                    <section class="info">
                      <div>
                        <span class="label">编号：</span>
                        <span class="content">P12138</span>
                      </div>
                      <div>
                        <span class="label">号牌：</span>
                        <span class="content">京 A3Q185</span>
                      </div>
                      <div>
                        <span class="label">平均车速：</span>
                        <span class="content">30km/h</span>
                      </div>
                      <div>
                        <span class="label">行车里程：</span>
                        <span class="content">16320.7km</span>
                      </div>
                      <div>
                        <span class="label">自动驾驶级别：</span>
                        <span class="content">L3</span>
                      </div>
                    </section>
                  </div>
                  <div class="operateImg">
                    <span onclick="(function () {
                      this.toTrackPlaybackFn();
                    })();">
                    <img src="static/fly_second.svg" alt="轨迹">
                    </span>
                    <span onclick="(function () {
                      this.videoShowFn(true);
                    })();">
                      <img src="static/fly_second.svg" alt="录像">
                    </span>
                  </div>
                </div>`;
							};
							// infowidnow 窗体配置
							var infoWindow = new AMap.InfoWindow({
								offset: new AMap.Pixel(0, -30),
							});
							marker = new AMap.Marker({
								map: self.map,
								// position: [116.478935, 39.997761],
								position: [116.514114, 39.780368],
								icon: "static/fly_red.svg",
								offset: new AMap.Pixel(-30, -10),
							});
							const markerInfo = new AMap.Marker({
								map: self.map,
								position: [116.514255, 39.779813],
								icon: "",
								offset: new AMap.Pixel(-13, -26),
							});

							marker.content = makeContentFn(lineArr);
							marker.on("click", markerClick);
							markerInfo.content = makeContentFn(lineArr);
							markerInfo.on("click", markerClick);
							// 窗体点击事件
							function markerClick(e) {
								// console.log(e);
								infoWindow.setContent(e.target.content);
								infoWindow.open(self.map, e.target.getPosition());
							}
							// 绘制轨迹
							var polyline = new AMap.Polyline({
								map: self.map,
								path: lineArr,
								showDir: true,
								strokeColor: "transparent", //线颜色
								// strokeOpacity: 1,     //线透明度
								strokeWeight: 6, //线宽
								// strokeStyle: "solid"  //线样式
							});

							var passedPolyline = new AMap.Polyline({
								map: self.map,
								strokeColor: "transparent", //线颜色
								strokeWeight: 6, //线宽
							});

							marker.on("moving", function (e) {
								// 绘制轨迹
								polyline;
								passedPolyline.setPath(e.passedPath);
							});

							// map.setFitView();

							window.startAnimationHistory = function startAnimation() {
								marker.moveAlong(lineArr, {
									// 每一段的时长
									duration: 2000,
									// JSAPI2.0 是否延道路自动设置角度在 moveAlong 里设置
									autoRotation: true,
								});
							};

							window.pauseAnimation = function () {
								marker.pauseMove();
							};

							window.resumeAnimation = function () {
								marker.resumeMove();
							};

							window.stopAnimation = function () {
								marker.stopMove();
							};
							// 自动播放
							window.startAnimationHistory();
							// 定时器触发循环
							self.historyInterval = setInterval(() => {
								window.startAnimationHistory();
							}, 32000);
						}
					);
				})
				.catch((e) => {
					console.log(e);
				});
		},
		historyFirst() {
			const self = this;
			AMapLoader.load({})
				.then((AMap) => {
					// JSAPI2.0 使用覆盖物动画必须先加载动画插件 ['AMap.ToolBar','AMap.Driving']
					AMap.plugin(
						["AMap.ToolBar", "AMap.MoveAnimation", "AMap.ControlBar"],
						function () {
							var markerFirst,
								lineArr = [
									[116.506124, 39.781449],
									[116.509328, 39.780741],
									[116.509705, 39.78081],
									[116.511772, 39.781229],
									[116.512498, 39.781405],
									[116.514194, 39.782182],
									[116.51492, 39.782516],
									[116.516743, 39.783396],
									[116.518802, 39.784447],
									[116.519959, 39.785015],
									[116.52149, 39.785718],
									[116.523725, 39.786701],
								];

							// var map = new AMap.Map("container", {
							// 	rotateEnable: true,
							// 	pitchEnable: true,
							// 	zoom: 17,
							// 	pitch: 50,
							// 	rotation: -15,
							// 	viewMode: "3D", //开启3D视图,默认为关闭
							// 	zooms: [2, 20],
							// 	resizeEnable: true,
							// 	// center: [116.397428, 39.90923],
							// 	center: [116.306338, 39.975639],
							// });

							// 3D 工具
							var controlBar = new AMap.ControlBar({
								position: {
									right: "10px",
									top: "10px",
								},
							});
							console.log(self);
							controlBar.addTo(self.map);

							var toolBar = new AMap.ToolBar({
								position: {
									right: "40px",
									top: "110px",
								},
							});

							toolBar.addTo(self.map);
							// 自定义弹出窗体
							const makeContentFn = (data) => {
								console.log(data);
								return `<div class="carWrap">
                  <div class="carTitle">信息</div>
                  <div class="carInfoWrap">
                    <div class="carImgWrap">
                      <img src="static/fly_second.svg" alt="">
                    </div>
                    <section class="info">
                      <div>
                        <span class="label">编号：</span>
                        <span class="content">P12138</span>
                      </div>
                      <div>
                        <span class="label">号牌：</span>
                        <span class="content">京 A3Q185</span>
                      </div>
                      <div>
                        <span class="label">平均车速：</span>
                        <span class="content">30km/h</span>
                      </div>
                      <div>
                        <span class="label">行车里程：</span>
                        <span class="content">16320.7km</span>
                      </div>
                      <div>
                        <span class="label">自动驾驶级别：</span>
                        <span class="content">L3</span>
                      </div>
                    </section>
                  </div>
                  <div class="operateImg">
                    <span onclick="(function () {
                      this.toTrackPlaybackFn();
                    })();">
                    <img src="static/fly_second.svg" alt="轨迹">
                    </span>
                    <span onclick="(function () {
                      this.videoShowFn(true);
                    })();">
                      <img src="static/fly_second.svg" alt="录像">
                    </span>
                  </div>
                </div>`;
							};
							// infowidnow 窗体配置
							var infoWindow = new AMap.InfoWindow({
								offset: new AMap.Pixel(0, -30),
							});
							markerFirst = new AMap.Marker({
								map: self.map,
								position: [116.506124, 39.781449],
								icon: "img/mapMode/car.png",
								offset: new AMap.Pixel(-13, -26),
							});
							markerFirst.content = makeContentFn(lineArr);
							markerFirst.on("click", markerClick);
							// 窗体点击事件
							function markerClick(e) {
								// console.log(e);
								infoWindow.setContent(e.target.content);
								infoWindow.open(self.map, e.target.getPosition());
							}
							// 绘制轨迹
							var polyline = new AMap.Polyline({
								map: self.map,
								path: lineArr,
								showDir: true,
								strokeColor: "#28F", //线颜色
								// strokeOpacity: 1,     //线透明度
								strokeWeight: 6, //线宽
								// strokeStyle: "solid"  //线样式
							});

							var passedPolyline = new AMap.Polyline({
								map: self.map,
								strokeColor: "#AF5", //线颜色
								strokeWeight: 6, //线宽
							});

							markerFirst.on("moving", function (e) {
								// 绘制轨迹
								polyline;
								passedPolyline.setPath(e.passedPath);
							});

							// map.setFitView();

							window.startAnimationFirst = function startAnimation() {
								markerFirst.moveAlong(lineArr, {
									// 每一段的时长
									duration: 2000,
									// JSAPI2.0 是否延道路自动设置角度在 moveAlong 里设置
									autoRotation: true,
								});
							};

							// window.pauseAnimation = function () {
							// 	markerFirst.pauseMove();
							// };

							// window.resumeAnimation = function () {
							// 	markerFirst.resumeMove();
							// };

							// window.stopAnimation = function () {
							// 	markerFirst.stopMove();
							// };
							// 自动播放
							window.startAnimationFirst();
							// 定时器触发循环
							self.historyIntervalFirst = setInterval(() => {
								window.startAnimationFirst();
							}, 24000);
						}
					);
				})
				.catch((e) => {
					console.log(e);
				});
		},
		historySecond() {
			const self = this;
			AMapLoader.load({})
				.then((AMap) => {
					// JSAPI2.0 使用覆盖物动画必须先加载动画插件 ['AMap.ToolBar','AMap.Driving']
					AMap.plugin(
						["AMap.ToolBar", "AMap.MoveAnimation", "AMap.ControlBar"],
						function () {
							var markerSecond,
								lineArr = [
									[116.523725, 39.786701],
									[116.52149, 39.785718],
									[116.519959, 39.785015],
									[116.518802, 39.784447],
									[116.516743, 39.783396],
									[116.51492, 39.782516],
									[116.514194, 39.782182],
									[116.512498, 39.781405],
									[116.511772, 39.781229],
									[116.509705, 39.78081],
									[116.509328, 39.780741],
									[116.508802, 39.779257],
									[116.508212, 39.777606],
								];

							// 3D 工具
							var controlBar = new AMap.ControlBar({
								position: {
									right: "10px",
									top: "10px",
								},
							});
							console.log(self);
							controlBar.addTo(self.map);

							var toolBar = new AMap.ToolBar({
								position: {
									right: "40px",
									top: "110px",
								},
							});

							toolBar.addTo(self.map);
							// 自定义弹出窗体
							const makeContentFn = (data) => {
								console.log(data);
								return `<div class="carWrap">
                  <div class="carTitle">信息</div>
                  <div class="carInfoWrap">
                    <div class="carImgWrap">
                      <img src="img/mapMode/car-img.jpeg" alt="">
                    </div>
                    <section class="info">
                      <div>
                        <span class="label">编号：</span>
                        <span class="content">P12138</span>
                      </div>
                      <div>
                        <span class="label">号牌：</span>
                        <span class="content">京 A3Q185</span>
                      </div>
                      <div>
                        <span class="label">平均车速：</span>
                        <span class="content">30km/h</span>
                      </div>
                      <div>
                        <span class="label">行车里程：</span>
                        <span class="content">16320.7km</span>
                      </div>
                      <div>
                        <span class="label">自动驾驶级别：</span>
                        <span class="content">L3</span>
                      </div>
                    </section>
                  </div>
                  <div class="operateImg">
                    <span onclick="(function () {
                      this.toTrackPlaybackFn();
                    })();">
                    <img src="img/mapMode/gray-track.svg" alt="轨迹">
                    </span>
                    <span onclick="(function () {
                      this.videoShowFn(true);
                    })();">
                      <img src="img/mapMode/gray-video.svg" alt="录像">
                    </span>
                  </div>
                </div>`;
							};
							// infowidnow 窗体配置
							var infoWindow = new AMap.InfoWindow({
								offset: new AMap.Pixel(0, -30),
							});
							markerSecond = new AMap.Marker({
								map: self.map,
								position: [116.523725, 39.786701],
								icon: "img/mapMode/car.png",
								offset: new AMap.Pixel(-13, -26),
							});
							markerSecond.content = makeContentFn(lineArr);
							markerSecond.on("click", markerClick);
							// 窗体点击事件
							function markerClick(e) {
								// console.log(e);
								infoWindow.setContent(e.target.content);
								infoWindow.open(self.map, e.target.getPosition());
							}
							// 绘制轨迹
							var polyline = new AMap.Polyline({
								map: self.map,
								path: lineArr,
								showDir: true,
								strokeColor: "#28F", //线颜色
								// strokeOpacity: 1,     //线透明度
								strokeWeight: 6, //线宽
								// strokeStyle: "solid"  //线样式
							});

							var passedPolyline = new AMap.Polyline({
								map: self.map,
								strokeColor: "#AF5", //线颜色
								strokeWeight: 6, //线宽
							});

							markerSecond.on("moving", function (e) {
								// 绘制轨迹
								polyline;
								passedPolyline.setPath(e.passedPath);
							});

							// map.setFitView();

							window.startAnimationSecond = function startAnimation() {
								markerSecond.moveAlong(lineArr, {
									// 每一段的时长
									duration: 2000,
									// JSAPI2.0 是否延道路自动设置角度在 moveAlong 里设置
									autoRotation: true,
								});
							};

							// window.pauseAnimation = function () {
							// 	markerSecond.pauseMove();
							// };

							// window.resumeAnimation = function () {
							// 	markerSecond.resumeMove();
							// };

							// window.stopAnimation = function () {
							// 	markerSecond.stopMove();
							// };
							// 自动播放
							window.startAnimationSecond();
							// 定时器触发循环
							self.historyIntervalSecond = setInterval(() => {
								window.startAnimationSecond();
							}, 26000);
						}
					);
				})
				.catch((e) => {
					console.log(e);
				});
		},
		historyThird() {
			const self = this;
			AMapLoader.load({})
				.then((AMap) => {
					// JSAPI2.0 使用覆盖物动画必须先加载动画插件 ['AMap.ToolBar','AMap.Driving']
					AMap.plugin(
						["AMap.ToolBar", "AMap.MoveAnimation", "AMap.ControlBar"],
						function () {
							var markerThird,
								lineArr = [
									[116.514255, 39.777521],
									[116.514623, 39.778373],
									[116.516237, 39.777982],
									[116.518233, 39.77744],
									[116.518639, 39.777317],
									[116.519355, 39.779185],
									[116.520329, 39.780518],
									[116.521526, 39.781317],
									[116.522735, 39.782116],
									[116.521448, 39.783667],
									[116.520327, 39.78519],
									[116.518639, 39.787237],
									[116.51876, 39.787087],
									[116.51783, 39.788079],
									[116.51636, 39.787466],
									[116.514846, 39.786741],
									[116.513077, 39.785901],
									[116.511284, 39.78499],
									[116.510104, 39.784497],
									[116.511523, 39.782688],
									[116.513374, 39.78018],
									[116.513965, 39.779293],
									[116.514602, 39.778383],
									[116.514128, 39.777275],
								];

							// 3D 工具
							var controlBar = new AMap.ControlBar({
								position: {
									right: "10px",
									top: "10px",
								},
							});
							console.log(self);
							controlBar.addTo(self.map);

							var toolBar = new AMap.ToolBar({
								position: {
									right: "40px",
									top: "110px",
								},
							});

							toolBar.addTo(self.map);
							// 自定义弹出窗体
							const makeContentFn = (data) => {
								console.log(data);
								return `<div class="carWrap">
                  <div class="carTitle">信息</div>
                  <div class="carInfoWrap">
                    <div class="carImgWrap">
                      <img src="img/mapMode/car-img.jpeg" alt="">
                    </div>
                    <section class="info">
                      <div>
                        <span class="label">编号：</span>
                        <span class="content">P12138</span>
                      </div>
                      <div>
                        <span class="label">号牌：</span>
                        <span class="content">京 A3Q185</span>
                      </div>
                      <div>
                        <span class="label">平均车速：</span>
                        <span class="content">30km/h</span>
                      </div>
                      <div>
                        <span class="label">行车里程：</span>
                        <span class="content">16320.7km</span>
                      </div>
                      <div>
                        <span class="label">自动驾驶级别：</span>
                        <span class="content">L3</span>
                      </div>
                    </section>
                  </div>
                  <div class="operateImg">
                    <span onclick="(function () {
                      this.toTrackPlaybackFn();
                    })();">
                    <img src="img/mapMode/gray-track.svg" alt="轨迹">
                    </span>
                    <span onclick="(function () {
                      this.videoShowFn(true);
                    })();">
                      <img src="img/mapMode/gray-video.svg" alt="录像">
                    </span>
                  </div>
                </div>`;
							};
							// infowidnow 窗体配置
							var infoWindow = new AMap.InfoWindow({
								offset: new AMap.Pixel(0, -30),
							});
							markerThird = new AMap.Marker({
								map: self.map,
								position: [116.523725, 39.786701],
								icon: "img/mapMode/car.png",
								offset: new AMap.Pixel(-13, -26),
							});

							markerThird.content = makeContentFn(lineArr);
							markerThird.on("click", markerClick);
							// 窗体点击事件
							function markerClick(e) {
								// console.log(e);
								infoWindow.setContent(e.target.content);
								infoWindow.open(self.map, e.target.getPosition());
							}

							// 绘制轨迹
							var polyline = new AMap.Polyline({
								map: self.map,
								path: lineArr,
								showDir: true,
								strokeColor: "#28F", //线颜色
								// strokeOpacity: 1,     //线透明度
								strokeWeight: 6, //线宽
								// strokeStyle: "solid"  //线样式
							});

							var passedPolyline = new AMap.Polyline({
								map: self.map,
								strokeColor: "#AF5", //线颜色
								strokeWeight: 6, //线宽
							});

							markerThird.on("moving", function (e) {
								// 绘制轨迹
								polyline;
								passedPolyline.setPath(e.passedPath);
							});

							// map.setFitView();

							window.startAnimationThird = function startAnimation() {
								markerThird.moveAlong(lineArr, {
									// 每一段的时长
									duration: 2000,
									// JSAPI2.0 是否延道路自动设置角度在 moveAlong 里设置
									autoRotation: true,
								});
							};

							// window.pauseAnimation = function () {
							// 	markerSecond.pauseMove();
							// };

							// window.resumeAnimation = function () {
							// 	markerSecond.resumeMove();
							// };

							// window.stopAnimation = function () {
							// 	markerSecond.stopMove();
							// };
							// 自动播放
							window.startAnimationThird();
							// 定时器触发循环
							self.historyIntervalThird = setInterval(() => {
								window.startAnimationThird();
							}, 48000);
						}
					);
				})
				.catch((e) => {
					console.log(e);
				});
		},
	},
};
</script>

<style scoped lang="less">
.wrap {
	width: 100%;
	height: 100%;
	/*隐藏高德地图*/
	/deep/ .amap-logo {
		display: none !important;
	}
	/deep/ .amap-copyright {
		display: none !important;
	}

	// 地图弹出框样式
	/deep/.amap-info-content {
		background: #41464d;
		color: #ffffff;
	}
	/deep/.bottom-center .amap-info-sharp {
		border-top: 8px solid #41464d;
	}

	/deep/.carWrap {
		.carTitle {
			margin-bottom: 12px;
			text-indent: 8px;
		}
		.carInfoWrap {
			display: flex;
			height: 160px;
			.info {
				div {
					border: 1px solid #7a7d82;
					border-bottom: none;
					line-height: 24px;
					font-size: 12px;
					&:last-child {
						border-bottom: 1px solid #7a7d82;
					}
				}
				.label {
					display: inline-block;
					width: 122px;
					border-right: 1px solid #7a7d82;
					text-indent: 8px;
				}
				.content {
					display: inline-block;
					width: 138px;
					margin-left: -5px;
					text-indent: 8px;
				}
			}
			.carImgWrap {
				width: 200px;
				img {
					display: block;
					width: 64%;
					margin: 0 auto;
				}
			}
		}
		.operateImg {
			height: 36px;
			width: 160px;
			margin: 0 auto;
			display: flex;
			justify-content: space-around;
			span {
				display: inline-block;
				cursor: pointer;
				img {
					width: 20px;
					cursor: pointer;
				}
			}
		}
	}
}

#container-map {
	width: 100%;
	min-height: 800px;
	height: 100%;
}

.map_box {
	width: 100%;
}

h4 {
	font-family: inherit;
	line-height: 1.8;
	font-weight: 300;
	color: inherit;
	font-size: 1.1rem;
	margin-top: 0;
	margin-bottom: 0.5rem;
}

.serch_box {
	position: fixed;
	top: 0;
}

// 底部控制台样式
.controllerBtn {
	// background: #666666;
	color: #fff;
	width: 420px;
	height: 66px;
	border-bottom: 66px solid #666666;
	border-right: 12px solid;
	border-right-color: transparent;
	border-left: 12px solid;
	border-left-color: transparent;
	position: fixed;
	left: 50%;
	margin-left: -180px;
	bottom: 0;
	z-index: 9999;

	display: flex;
	justify-content: space-around;

	div {
		cursor: pointer;
		margin-top: 90px;
		min-width: 72px;
		text-align: center;
		svg {
			width: 72px;
			height: 72px;
		}
		span {
			display: block;
		}
	}
}

// 实时视频弹出层
.videoShow {
	display: block !important;
	transition: all 0.25s;
}
.videoMask {
	width: 100%;
	height: 100%;
	position: fixed;
	top: 64px;
	background: transparent;
	display: none;
	transition: all 0.25s;
	.dialogBox {
		width: 800px;
		height: 492px;
		background: #ffffff;
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		border-radius: 4px;
		.title {
			width: inherit;
			height: 40px;
			background: #bbbdbf;
			color: #222222;
			line-height: 40px;
			text-indent: 8px;
			display: flex;
			justify-content: space-between;
			span {
				font-weight: 500;
			}
			i {
				line-height: 36px;
				padding-right: 12px;
				cursor: pointer;
				&:hover {
					color: #0065cc;
					transition: all 0.25s;
				}
			}
		}
		.content {
			display: flex;
			justify-content: center;
			flex-wrap: wrap;
			margin-right: -4px;
			div {
				width: 48%;
				height: 214px;
				margin: 8px 6px 0 2px;
				background: #fac;
			}
		}
	}
}

// 摄像头弹出层
.cameraShow {
	display: block !important;
	transition: all 0.25s;
}
.cameraMask {
	width: 100%;
	height: 100%;
	position: fixed;
	top: 64px;
	background: transparent;
	display: none;
	transition: all 0.25s;
	.dialogBox {
		width: 402px;
		min-height: 492px;
		background: #ffffff;
		position: fixed;
		top: 148px;
		right: 30px;
		border-radius: 4px;
		.title {
			width: inherit;
			height: 40px;
			background: #bbbdbf;
			color: #222222;
			line-height: 40px;
			text-indent: 8px;
			display: flex;
			justify-content: space-between;
			span {
				font-weight: 500;
			}
			i {
				line-height: 36px;
				padding-right: 12px;
				cursor: pointer;
				&:hover {
					color: #0065cc;
					transition: all 0.25s;
				}
			}
		}
		.content {
			height: 678px;
			overflow-y: auto;
			div {
				width: 99%;
				height: 220px;
				margin: 4px 4px 0 2px;
				background: #fac;
			}
		}
	}
}

//
// 信号灯弹出层
.lightShow {
	display: block !important;
	transition: all 0.25s;
}
.lightMask {
	width: 100%;
	height: 100%;
	position: fixed;
	top: 64px;
	background: transparent;
	display: none;
	transition: all 0.25s;
	.dialogBox {
		width: 660px;
		min-height: 522px;
		background: #41464d;
		position: fixed;
		top: 146px;
		right: 30px;
		border-radius: 4px;
		color: #ffffff;
		.title {
			width: inherit;
			height: 40px;
			background: #bbbdbf;
			color: #222222;
			line-height: 40px;
			text-indent: 8px;
			display: flex;
			justify-content: space-between;
			span {
				font-weight: 500;
			}
			i {
				line-height: 36px;
				padding-right: 12px;
				cursor: pointer;
				&:hover {
					color: #0065cc;
					transition: all 0.25s;
				}
			}
		}
		.content {
			.imgTable {
				display: flex;
				justify-content: space-around;
				.img {
					width: 370px;
					height: 320px;
					margin: 14px;
					img {
						width: inherit;
					}
				}
				.table {
					margin: 14px 14px 0 0;
					.info {
						.tableTitle {
							background: #7a7d82;
						}
						div {
							border: 1px solid #7a7d82;
							border-bottom: none;
							line-height: 32px;
							font-size: 12px;
							display: flex;
							justify-content: center;
							&:last-child {
								border-bottom: 1px solid #7a7d82;
							}
						}
						.label {
							display: inline-block;
							width: 90px;
							border-right: 1px solid #7a7d82;
							text-indent: 8px;
						}
						.content {
							display: inline-block;
							width: 80px;
							border-right: 1px solid #7a7d82;
							text-indent: 8px;
						}
						.status {
							display: inline-block;
							width: 80px;
							text-indent: 8px;
						}
					}
					.switch {
						padding: 12px;
						span {
							padding-right: 12px;
						}
					}
				}
			}
			/deep/ .el-divider--horizontal {
				margin: 14px 0;
				background: #ffffff;
				opacity: 0.2;
			}
			.footerBtn {
				display: flex;
				justify-content: flex-end;
				padding-right: 20px;
			}
		}
	}
}
/deep/.amap-icon {
  overflow: visible!important;
}
</style>
