<template>
	<div class="mapModelWrap">
		<div
			id="container"
			style="height: 800px; width: 1600px; margin: 0 auto; resize: both;"
		></div>
		<!-- <div class="input-card">
			<h4>轨迹回放控制</h4>
			<div class="input-item">
				<input
					type="button"
					class="btn"
					value="开始动画"
					id="start"
					onclick="startAnimation()"
				/>
				<input
					type="button"
					class="btn"
					value="暂停动画"
					id="pause"
					onclick="pauseAnimation()"
				/>
			</div>
			<div class="input-item">
				<input
					type="button"
					class="btn"
					value="继续动画"
					id="resume"
					onclick="resumeAnimation()"
				/>
				<input
					type="button"
					class="btn"
					value="停止动画"
					id="stop"
					onclick="stopAnimation()"
				/>
			</div>
		</div> -->
		<div class="controllerBtn">
			<div @click="showCar">car</div>
			<div @click="showWebcam">camera</div>
			<div @click="showSemaphore">light</div>
		</div>
	</div>
</template>

<script>
import AMapLoader from "@amap/amap-jsapi-loader";

export default {
	name: "controllerMap",
	props: {
		msg: String,
	},
	data() {
		return {
			map: null,
			carMarker: null, //
			carMarkerShow: false, // 车辆显示隐藏
			webcamMarker: null,
			webcamMarkerShow: false,
			semaphoreMarker: null,
			semaphoreMarkerShow: false,
		};
	},
	mounted() {
		this.initOrbit();
	},
	methods: {
		initOrbit() {
			AMapLoader.load({
				key: "10e903bafe1cff3b9f952d3ba5b3cbb7", // 申请好的Web端开发者Key，首次调用 load 时必填
				version: "2.0", // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
				plugins: [], // 需要使用的的插件列表，如比例尺'AMap.Scale'等
				AMapUI: {
					// 是否加载 AMapUI，缺省不加载
					version: "1.1", // AMapUI 缺省 1.1
					plugins: [], // 需要加载的 AMapUI ui插件
				},
				Loca: {
					// 是否加载 Loca， 缺省不加载
					version: "2.0", // Loca 版本，缺省 1.3.2
				},
			})
				.then((AMap) => {
					//初始化地图对象，加载地图
					this.map = new AMap.Map("container", {
						resizeEnable: true, //是否监控地图容器尺寸变化
						rotateEnable: true,
						pitchEnable: true,
						zoom: 17,
						pitch: 55, // 地图俯仰角度，有效范围 0 度- 83 度
						rotation: -15,
						viewMode: "3D", //开启3D视图,默认为关闭
						zooms: [2, 20],
						// center: [116.368904, 39.923423],
						center: [116.293828, 39.947945],
					});
					// 调用警告marker方法
					this.waringMarker(AMap, this.map);
					// 调用车辆信息marker方法
					// this.carMarker(AMap, this.map);
				})
				.catch((e) => {
					console.log(e);
				});
		},
		/*
		 * 绘制marker方法
		 * @param {*} AMap
		 * */
		waringMarker(AMap, map) {
			// 后台获取marker点信息
			var lnglats = [
				[116.368904, 39.923423],
				[116.382122, 39.921176],
				[116.387271, 39.922501],
				[116.398258, 39.9146],
			];

			var infoWindow = new AMap.InfoWindow({
				offset: new AMap.Pixel(0, -30),
			});

			// 定义icon
			var waringIcon = new AMap.Icon({
				size: new AMap.Size(25, 34),
				image: "../static/waring-marker.svg",
				imageSize: new AMap.Size(24, 26),
				imageOffset: new AMap.Pixel(0, 3),
			});

			for (var i = 0, marker; i < lnglats.length; i++) {
				marker = new AMap.Marker({
					position: lnglats[i],
					map: map,
					// 图标尺寸
					size: new AMap.Size(25, 34),
					// 将一张图片的地址设置为 icon
					icon: waringIcon,
					// 设置了 icon 以后，设置 icon 的偏移量，以 icon 的 [center bottom] 为原点
					offset: new AMap.Pixel(-13, -30),
				});

				marker.content = `<div class="infoWrap">
              <div>
                <h4>违法信息</h4>
                <div>京A23125 闯红灯 09:23:45</div>
				<h4>设备故障</h4>
                <div>玉龙大街与天义路 09:23:45</div>
				<h4>布控信息</h4>
                <div>京A23125 09:23:45</div>
				<h4>区间超速</h4>
                <div>京A23125 09:23:45</div>
              </div>
            </div>`;
				marker.on("click", markerClick);
				// 默认触发
				// marker.emit("click", { target: marker });
				// 将 markers 添加到地图
				map.add(marker);
			}

			function markerClick(e) {
				infoWindow.setContent(e.target.content);
				infoWindow.open(map, e.target.getPosition());
			}
			// 是否将地图缩放到可以看到全部marker的级别
			// map.setFitView();
		},
		/*
		 * 绘制车辆布局方法
		 * @param {*} AMap
		 * */
		// carMarker(AMap, map) {
		// 	// 后台获取marker点信息
		// 	var lnglats = [
		// 		[116.290932, 39.947654],
		// 		[116.292321, 39.947873],
		// 		[116.293493, 39.947865],
		// 		[116.295759, 39.947458],
		// 	];
		// 	const makeContentFn = (data) => {
		// 		console.log(data);
		// 		return `<div >
		// 			<div class="carInfoWrap">
		// 				<div class="carImgWrap">
		// 					<img src="../static/car-img.jpeg" alt="">
		// 				</div>
		// 				<section>
		// 					<div>
		// 					<span>车辆编号：</span>
		// 					<span>P12138</span>
		// 					</div>
		// 					<div>
		// 					<span>车辆号牌：</span>
		// 					<span>京A123434</span>
		// 					</div>
		// 					<div>
		// 					<span>平均车速：</span>
		// 					<span>30km/h</span>
		// 					</div>
		// 					<div>
		// 					<span>行车里程</span>
		// 					<span>16320.7km</span>
		// 					</div>
		// 					<div>
		// 					<span>自动驾驶级别：</span>
		// 					<span>L3</span>
		// 					</div>
		// 				</section>
		// 			</div>
		// 			<div class="operateImg">
		// 				<span onclick="(function () {
		// 					alert('跳转页面')
		// 				})();">轨迹</span>
		// 				<span onclick="(function () {
		// 					alert('根据框架弹出dialog')
		// 				})();">录像</span>
		// 			</div>
		// 		</div>`;
		// 	};
		// 	// infowidnow 的 innerHTML
		// 	var infoWindow = new AMap.InfoWindow({
		// 		offset: new AMap.Pixel(0, -30),
		// 	});

		// 	// 定义icon
		// 	var carIcon = new AMap.Icon({
		// 		size: new AMap.Size(25, 34),
		// 		image: "../static/car-marker.svg",
		// 		imageSize: new AMap.Size(24, 26),
		// 		imageOffset: new AMap.Pixel(0, 3),
		// 	});

		// 	for (var i = 0, marker; i < lnglats.length; i++) {
		// 		marker = new AMap.Marker({
		// 			position: lnglats[i],
		// 			map: map,
		// 			// 图标尺寸
		// 			size: new AMap.Size(25, 34),
		// 			// 将一张图片的地址设置为 icon
		// 			icon: carIcon,
		// 			// 设置了 icon 以后，设置 icon 的偏移量，以 icon 的 [center bottom] 为原点
		// 			offset: new AMap.Pixel(-13, -30),
		// 		});

		// 		marker.content = makeContentFn(lnglats);
		// 		marker.on("click", markerClick);
		// 		// 默认触发
		// 		// marker.emit("click", { target: marker });
		// 		// 将 markers 添加到地图
		// 		map.add(marker);
		// 	}

		// 	function markerClick(e) {
		// 		infoWindow.setContent(e.target.content);
		// 		infoWindow.open(map, e.target.getPosition());
		// 	}
		// 	// 将当前经纬度展示在 infowindow 的 input 中

		// 	// 是否将地图缩放到可以看到全部marker的级别
		// 	// map.setFitView();
		// },
		/*
		 * 点击显示车辆位置信息
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
						[116.290932, 39.947654],
						[116.292321, 39.947873],
						[116.293493, 39.947865],
						[116.295759, 39.947458],
					];
					// 后台获取car 提示信息
					var carTips = [
						[116.290837,39.947901],
						[116.292282,39.94812],
						[116.293682,39.948098]
					];
					// 车辆自定义弹出窗体
					const makeContentFn = (data) => {
						console.log(data);
						return `<div >
					<div class="carInfoWrap">
						<div class="carImgWrap">
							<img src="../static/car-img.jpeg" alt="">
						</div>
						<section>
							<div>
							<span>车辆编号：</span>
							<span>P12138</span>
							</div>
							<div>
							<span>车辆号牌：</span>
							<span>京A123434</span>
							</div>
							<div>
							<span>平均车速：</span>
							<span>30km/h</span>
							</div>
							<div>
							<span>行车里程：</span>
							<span>16320.7km</span>
							</div>
							<div>
							<span>自动驾驶级别：</span>
							<span>L3</span>
							</div>
						</section>
					</div>
					<div class="operateImg">
						<span onclick="(function () {
							alert('跳转页面')
						})();">轨迹</span>
						<span onclick="(function () {
							alert('根据框架弹出dialog')
						})();">录像</span>
					</div>
				</div>`;
					};
					// 车辆提示告警弹出窗体内容
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
						image: "../static/car-marker.svg",
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
						image: "../static/car-tips-marker.svg",
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
						console.log(e);
						infoWindow.setContent(e.target.content);
						infoWindow.open(thisMap, e.target.getPosition());
					}
					// 鼠标离开事件
					function markerHide(e) {
						console.log(e);
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
					const thisMap = this.map;
					// 后台获取marker点信息
					var lnglats = [
						[116.294837,39.949751],
						[116.295949,39.948457],
						[116.294944,39.946758],
						[116.295899,39.946678],
					];
					// 车辆自定义弹出窗体
					const makeContentFn = (data) => {
						console.log(data);
						return `<div >
					<div class="carInfoWrap">
						<div class="carImgWrap">
							<img src="../static/car-img.jpeg" alt="">
						</div>
						<section>
							<div>
							<span>车辆编号：</span>
							<span>P12138</span>
							</div>
							<div>
							<span>车辆号牌：</span>
							<span>京A123434</span>
							</div>
							<div>
							<span>平均车速：</span>
							<span>30km/h</span>
							</div>
							<div>
							<span>行车里程：</span>
							<span>16320.7km</span>
							</div>
							<div>
							<span>自动驾驶级别：</span>
							<span>L3</span>
							</div>
						</section>
					</div>
					<div class="operateImg">
						<span onclick="(function () {
							alert('跳转页面')
						})();">轨迹</span>
						<span onclick="(function () {
							alert('根据框架弹出dialog')
						})();">录像</span>
					</div>
				</div>`;
					};

					// infowidnow 窗体配置
					var infoWindow = new AMap.InfoWindow({
						offset: new AMap.Pixel(0, -30),
					});

					// 定义car-icon
					var carIcon = new AMap.Icon({
						size: new AMap.Size(25, 34),
						image: "../static/webcam-marker.svg",
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

				
					// 数组赋值
					this.webcamMarker = marker;
					// 窗体点击事件
					function markerClick(e) {
						console.log(e);
						infoWindow.setContent(e.target.content);
						infoWindow.open(thisMap, e.target.getPosition());
						// 直接右上角定位展示视频
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
					const thisMap = this.map;
					// 后台获取marker点信息
					var lnglats = [
						[116.294857,39.945535],
						[116.288192,39.946956],
						[116.298891,39.946304],
					];
					// 车辆自定义弹出窗体
					const makeContentFn = (data) => {
						console.log(data);
						return `<div >
					<div class="carInfoWrap">
						<div class="carImgWrap">
							<img src="../static/car-img.jpeg" alt="">
						</div>
						<section>
							<div>
							<span>车辆编号：</span>
							<span>P12138</span>
							</div>
							<div>
							<span>车辆号牌：</span>
							<span>京A123434</span>
							</div>
							<div>
							<span>平均车速：</span>
							<span>30km/h</span>
							</div>
							<div>
							<span>行车里程：</span>
							<span>16320.7km</span>
							</div>
							<div>
							<span>自动驾驶级别：</span>
							<span>L3</span>
							</div>
						</section>
					</div>
					<div class="operateImg">
						<span onclick="(function () {
							alert('跳转页面')
						})();">轨迹</span>
						<span onclick="(function () {
							alert('根据框架弹出dialog')
						})();">录像</span>
					</div>
				</div>`;
					};

					// infowidnow 窗体配置
					var infoWindow = new AMap.InfoWindow({
						offset: new AMap.Pixel(0, -30),
					});

					// 定义car-icon
					var carIcon = new AMap.Icon({
						size: new AMap.Size(25, 34),
						image: "../static/semaphore-marker.svg",
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

					// 数组赋值
					this.semaphoreMarker = marker;
					// 窗体点击事件
					function markerClick(e) {
						console.log(e);
						infoWindow.setContent(e.target.content);
						infoWindow.open(thisMap, e.target.getPosition());
					}

					// 已经显示了
					this.semaphoreMarkerShow = !this.semaphoreMarkerShow;
				})
				.catch((e) => {
					console.error(e);
				});
		},
	},
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less">
html,
body,
#container {
	height: 100%;
	width: 100%;
}

.input-card .btn {
	margin-right: 1.2rem;
	width: 9rem;
}

.input-card .btn:last-child {
	margin-right: 0;
}

.mapModelWrap {
	background: #f2f3f4;
	position: relative;
	/deep/.carInfoWrap {
		display: flex;
		height: 160px;
		.carImgWrap {
			width: 200px;
			img {
				display: block;
				width: 50%;
				margin: 0 auto;
			}
		}
	}
	/deep/.operateImg {
		height: 36px;
		display: flex;
		justify-content: space-around;
		span {
			display: inline-block;
			cursor: pointer;
		}
	}
	.controllerBtn {
		background: #666666;
		color: #fff;
		border-radius: 8px 8px 0 0;
		width: 360px;
		height: 66px;
		position: absolute;
		left: 50%;
		margin-left: -180px;
		bottom: 0;
		z-index: 9999;

		display: flex;
		justify-content: space-around;
		line-height: 66px;

		div {
			cursor: pointer;
		}
	}
	// 右上角总览
}
</style>
