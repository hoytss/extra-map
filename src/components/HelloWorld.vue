<template>
	<div class="">
		<div
			id="container"
			style="height: 800px; width: 1600px; margin: 0 auto; resize: both;"
		></div>
		<div class="input-card">
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
		</div>
	</div>
</template>

<script>
import AMapLoader from "@amap/amap-jsapi-loader";

export default {
	name: "HelloWorld",
	props: {
		msg: String,
	},
	data() {
		return {};
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
					// var map = new AMap.Map("container", {
					// 	rotateEnable: true,
					// 	pitchEnable: true,
					// 	zoom: 17,
					// 	pitch: 50,
					// 	rotation: -15,
					// 	viewMode: "3D", //开启3D视图,默认为关闭
					// 	zooms: [2, 20],
					// 	center: [116.333926, 39.997245],
					// });
					// map;
					// JSAPI2.0 使用覆盖物动画必须先加载动画插件 ['AMap.ToolBar','AMap.Driving']
					// AMap.plugin(
					// 	["AMap.ToolBar", "AMap.MoveAnimation", "AMap.ControlBar"],
					// 	function () {
					// 		var marker,
					// 			// lineArr = [
					// 			// 	[116.478935, 39.997761],
					// 			// 	[116.478939, 39.997825],
					// 			// 	[116.478912, 39.998549],
					// 			// 	[116.478912, 39.998549],
					// 			// 	[116.478998, 39.998555],
					// 			// 	[116.478998, 39.998555],
					// 			// 	[116.479282, 39.99856],
					// 			// 	[116.479658, 39.998528],
					// 			// 	[116.480151, 39.998453],
					// 			// 	[116.480784, 39.998302],
					// 			// 	[116.480784, 39.998302],
					// 			// 	[116.481149, 39.998184],
					// 			// 	[116.481573, 39.997997],
					// 			// 	[116.481863, 39.997846],
					// 			// 	[116.482072, 39.997718],
					// 			// 	[116.482362, 39.997718],
					// 			// 	[116.483633, 39.998935],
					// 			// 	[116.48367, 39.998968],
					// 			// 	[116.484648, 39.999861],
					// 			// ];
					// 			lineArr = [
					// 				[116.306339, 39.975654],
					// 				[116.306302, 39.976091],
					// 				[116.306278, 39.976396],
					// 				[116.306233, 39.976904],
					// 				[116.306153, 39.977671],
					// 				[116.306094, 39.978244],
					// 				[116.306061, 39.978569],
					// 				[116.306028, 39.978953],
					// 				[116.306124, 39.978984],
					// 				[116.306647, 39.978992],
					// 				[116.307616, 39.979032],
					// 				[116.308294, 39.979032],
					// 				[116.308456, 39.978744],
					// 				[116.308728, 39.978182],
					// 				[116.308887, 39.977388],
					// 				[116.308966, 39.975903],
					// 				[116.308682, 39.975826],
					// 				[116.307657, 39.975781],
					// 				[116.306918, 39.97571],
					// 			];

					// 		var map = new AMap.Map("container", {
					// 			rotateEnable: true,
					// 			pitchEnable: true,
					// 			zoom: 17,
					// 			pitch: 50,
					// 			rotation: -15,
					// 			viewMode: "3D", //开启3D视图,默认为关闭
					// 			zooms: [2, 20],
					// 			resizeEnable: true,
					// 			// center: [116.397428, 39.90923],
					// 			center: [116.306338, 39.975639],
					// 		});

					// 		// 3D 工具
					// 		var controlBar = new AMap.ControlBar({
					// 			position: {
					// 				right: "10px",
					// 				top: "10px",
					// 			},
					// 		});
					// 		controlBar.addTo(map);

					// 		var toolBar = new AMap.ToolBar({
					// 			position: {
					// 				right: "40px",
					// 				top: "110px",
					// 			},
					// 		});
					// 		toolBar.addTo(map);

					// 		marker = new AMap.Marker({
					// 			map: map,
					// 			// position: [116.478935, 39.997761],
					// 			position: [116.306338, 39.975639],
					// 			icon:
					// 				"https://a.amap.com/jsapi_demos/static/demo-center-v2/car.png",
					// 			offset: new AMap.Pixel(-13, -26),
					// 		});

					// 		// 绘制轨迹
					// 		var polyline = new AMap.Polyline({
					// 			map: map,
					// 			path: lineArr,
					// 			showDir: true,
					// 			strokeColor: "#28F", //线颜色
					// 			// strokeOpacity: 1,     //线透明度
					// 			strokeWeight: 6, //线宽
					// 			// strokeStyle: "solid"  //线样式
					// 		});

					// 		var passedPolyline = new AMap.Polyline({
					// 			map: map,
					// 			strokeColor: "#AF5", //线颜色
					// 			strokeWeight: 6, //线宽
					// 		});

					// 		marker.on("moving", function (e) {
					// 			// 绘制轨迹
					// 			polyline;
					// 			passedPolyline.setPath(e.passedPath);
					// 		});

					// 		map.setFitView();

					// 		window.startAnimation = function startAnimation() {
					// 			marker.moveAlong(lineArr, {
					// 				// 每一段的时长
					// 				duration: 600,
					// 				// JSAPI2.0 是否延道路自动设置角度在 moveAlong 里设置
					// 				autoRotation: true,
					// 			});
					// 		};

					// 		window.pauseAnimation = function () {
					// 			marker.pauseMove();
					// 		};

					// 		window.resumeAnimation = function () {
					// 			marker.resumeMove();
					// 		};

					// 		window.stopAnimation = function () {
					// 			marker.stopMove();
					// 		};

					// 		console.log(map.getCenter());
					// 		// 绘制点
					// 		var markerPoint = new AMap.Marker({
					// 			position: map.getCenter(),
					// 			icon:
					// 				"//a.amap.com/jsapi_demos/static/demo-center/icons/poi-marker-default.png",
					// 			anchor: "bottom-center",
					// 			offset: new AMap.Pixel(0, 0),
					// 		});

					// 		markerPoint.setMap(map);

					// 		// 设置鼠标划过点标记显示的文字提示
					// 		markerPoint.setTitle("我是marker的title");

					// 		// 设置label标签
					// 		// label默认蓝框白底左上角显示，样式className为：amap-marker-label
					// 		markerPoint.setLabel({
					// 			direction: "right",
					// 			offset: new AMap.Pixel(10, 0), //设置文本标注偏移量
					// 			content:
					// 				"<div class='info'>我是 marker 的 label 标签，我的样式可更改</div>", //设置文本标注内容
					// 		});
					// 点的点击事件
					// }
					// );

					//初始化地图对象，加载地图
					var map = new AMap.Map("container", {
						resizeEnable: true, //是否监控地图容器尺寸变化
						rotateEnable: true,
						pitchEnable: true,
						zoom: 17,
						pitch: 55, // 地图俯仰角度，有效范围 0 度- 83 度
						rotation: -15,
						viewMode: "3D", //开启3D视图,默认为关闭
						zooms: [2, 20],
						center: [116.368904, 39.923423],
					});
          // 调用警告marker方法
					this.waringMarker(AMap, map);
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
				image: "../static/waring.svg",
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
                <h5>违法信息</h5>
                <div>京A23125 闯红灯 09:23:45</div>
              </div>
            </div>`;
				marker.on("click", markerClick);
				marker.emit("click", { target: marker });
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
.amap-icon img {
	position: relative !important;
}
</style>
