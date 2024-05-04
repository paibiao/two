<template>
  <div id="cesiumContainer"></div>
  <div id="showDetailInfo" v-show="detailIsShow">
    <el-card style="width: 75vw; height: 80vh; padding: 20px">
      <span class="close">X</span>
      <el-table :data="tableData" style="width: 100%">
        <el-table-column prop="date" label="Date" />
        <el-table-column prop="name" label="Name" />
        <el-table-column prop="address" label="Address" />
      </el-table>
    </el-card>
  </div>
  <div class="mask" v-show="detailIsShow"></div>
</template>

<script setup>
import * as Cesium from "cesium";
import { onMounted, ref } from "vue";
import { load3dtiles, update3dtiles } from "/src/utils/load3D.js";
import build from "/src/assets/build.json";
let viewer, handle, lastPick;
let detailIsShow = ref(false);
const tableData = ref([
  {
    date: "2016-05-03",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles",
  },
  {
    date: "2016-05-02",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles",
  },
  {
    date: "2016-05-04",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles",
  },
  {
    date: "2016-05-01",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles",
  },
]);

Cesium.Ion.defaultAccessToken =
  "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI5ZWZlNmFmNS0yNTkxLTQ2NzEtYjk4NS1lNGY5NWM1OTc5MmYiLCJpZCI6MjA1MTk1LCJpYXQiOjE3MTE2NzU4OTJ9.O9J1G_tirf33A-GKVOhxk1h9lWH08IZTnMOf-y8XS_U";

function delAll(viewer) {
  viewer.entities.removeAll();
  handle.removeInputAction(Cesium.ScreenSpaceEventType.LEFT_CLICK);
  handle.removeInputAction(Cesium.ScreenSpaceEventType.MOUSE_MOVE);
}

function init(viewer) {
  highPolygonInit(viewer);
  lowerPolygonInit(viewer);
  setMouseMoveEvent(viewer);
  setMouseClickEvent(viewer);
}

function showDetailLowerPolygon(viewer) {
  delAll(viewer);
  let cameraOrtArr = build.lowerPolygon.view.cameraOrt.split(",");
  let cameraPositionArr = build.lowerPolygon.view.cameraPosition.split(",");
  cameraPositionArr = cameraPositionArr.map((item) => Number(item));
  cameraOrtArr = cameraOrtArr.map((item) => Number(item));
  //飞到细节视角
  viewer.camera.flyTo({
    destination: new Cesium.Cartesian3(...cameraPositionArr),
    orientation: {
      heading: cameraOrtArr[0], // 默认
      pitch: cameraOrtArr[1], // 默认
      roll: cameraOrtArr[2], // 默认
    },
    duration: 3,
  });
  let childArr = build.lowerPolygon.child;
  // 显示楼栋号;
  childArr.forEach((item) => {
    viewer.entities.add({
      id: item.name,
      position: Cesium.Cartesian3.fromDegrees(...item.position),
      label: {
        text: item.name,
        font: "16px sans-serif",
        fillColor: Cesium.Color.WHITE,
        showBackground: true,
        backgroundColor: Cesium.Color.fromCssColorString("ffffff"),
      },
    });
  });
}

function showDetailHighPolygon(viewer) {
  delAll(viewer);
  let cameraOrtArr = build.highPolygon.view.cameraOrt.split(",");
  let cameraPositionArr = build.highPolygon.view.cameraPosition.split(",");
  cameraPositionArr = cameraPositionArr.map((item) => Number(item));
  cameraOrtArr = cameraOrtArr.map((item) => Number(item));
  //飞到细节视角
  viewer.camera.flyTo({
    destination: new Cesium.Cartesian3(...cameraPositionArr),
    orientation: {
      heading: cameraOrtArr[0], // 默认
      pitch: cameraOrtArr[1], // 默认
      roll: cameraOrtArr[2], // 默认
    },
    duration: 3,
  });
  let childArr = build.highPolygon.child;
  // 显示楼栋号;
  childArr.forEach((item) => {
    viewer.entities.add({
      id: item.name,
      position: Cesium.Cartesian3.fromDegrees(...item.position),
      label: {
        text: item.name,
        font: "16px sans-serif",
        fillColor: Cesium.Color.WHITE,
        showBackground: true,
        backgroundColor: Cesium.Color.fromCssColorString("ffffff"),
      },
    });
  });
}

function setMouseClickEvent(viewer) {
  //添加鼠标点击事件
  handle.setInputAction((event) => {
    let pick = viewer.scene.pick(event.position);
    if (pick && pick.id && pick.id.id == "lowerPolygonRangeArae") {
      showDetailLowerPolygon(viewer);
    } else if (pick && pick.id && pick.id.id == "highPolygonRangeArae") {
      showDetailHighPolygon(viewer);
    }
  }, Cesium.ScreenSpaceEventType.LEFT_CLICK);
}

function setMouseMoveEvent(viewer) {
  //添加鼠标移动事件
  handle.setInputAction((event) => {
    if (lastPick) {
      let color = lastPick.polygon.material.color._value;
      lastPick.polygon.material = color.withAlpha(0);
    }
    let pick = viewer.scene.pick(event.endPosition);
    if (pick && pick.id && pick.id.id == "lowerPolygonRangeArae") {
      let color = pick.id.polygon.material.color._value;
      pick.id.polygon.material = color.withAlpha(0.1);
      lastPick = pick.id;
    } else if (pick && pick.id && pick.id.id == "highPolygonRangeArae") {
      let color = pick.id.polygon.material.color._value;
      pick.id.polygon.material = color.withAlpha(0.1);
      lastPick = pick.id;
    }
  }, Cesium.ScreenSpaceEventType.MOUSE_MOVE);
}

function lowerPolygonInit(viewer) {
  let lowerPolygonData = build.lowerPolygon;
  //添加区域文本
  viewer.entities.add({
    id: "lowerPolygonType",
    position: Cesium.Cartesian3.fromDegrees(...lowerPolygonData.center),
    label: {
      text: lowerPolygonData.type,
      fillColor: Cesium.Color.WHITE,
      showBackground: true,
      backgroundColor: Cesium.Color.fromCssColorString("ffffff"),
    },
  });
  // 添加区域范围线;
  viewer.entities.add({
    id: "lowerPolygonRange",
    polyline: {
      positions: Cesium.Cartesian3.fromDegreesArray(lowerPolygonData.range),
      material: new Cesium.PolylineGlowMaterialProperty({
        glowPower: 0.1,
        color: Cesium.Color.fromCssColorString(lowerPolygonData.color),
      }),
      width: 20,
      clampToGround: true,
    },
  });
  //添加区域面
  viewer.entities.add({
    id: "lowerPolygonRangeArae",
    polygon: {
      hierarchy: {
        positions: Cesium.Cartesian3.fromDegreesArray(lowerPolygonData.range),
      },
      material: Cesium.Color.fromCssColorString(
        lowerPolygonData.color
      ).withAlpha(0),
    },
  });
}

function highPolygonInit(viewer) {
  let highPolygonData = build.highPolygon;
  //添加区域文本
  viewer.entities.add({
    id: "highPolygonType",
    position: Cesium.Cartesian3.fromDegrees(...highPolygonData.center),
    label: {
      text: highPolygonData.type,
      fillColor: Cesium.Color.WHITE,
      showBackground: true,
      backgroundColor: Cesium.Color.fromCssColorString("ffffff"),
    },
  });
  // 添加区域范围线;
  viewer.entities.add({
    id: "highPolygonRange",
    polyline: {
      positions: Cesium.Cartesian3.fromDegreesArray(highPolygonData.range),
      material: new Cesium.PolylineGlowMaterialProperty({
        glowPower: 0.1,
        color: Cesium.Color.fromCssColorString(highPolygonData.color),
      }),
      width: 20,
      clampToGround: true,
    },
  });
  //添加区域面
  viewer.entities.add({
    id: "highPolygonRangeArae",
    polygon: {
      hierarchy: {
        positions: Cesium.Cartesian3.fromDegreesArray(highPolygonData.range),
      },
      material: Cesium.Color.fromCssColorString(
        highPolygonData.color
      ).withAlpha(0),
    },
  });
}

onMounted(() => {
  viewer = new Cesium.Viewer("cesiumContainer", {
    infoBox: false, //关闭右边显示内容
    selectionIndicator: false, //关闭实体选中
    baseLayerPicker: false, //是否显示图层选择控件
    animation: false, //是否显示动画控件
    timeline: false, //是否显示时间轴控件
    fullscreenButton: false, //是否显示全屏按钮
    geocoder: false, //是否显示搜索按钮
    homeButton: false, //是否显示主页按钮
    navigationHelpButton: false, //是否显示帮助提示按钮
    sceneModePicker: false, //是否显示投影方式按钮
  });
  handle = new Cesium.ScreenSpaceEventHandler(viewer.scene.canvas);
  load3dtiles(viewer, "/src/assets/b3dm/tileset.json", (tileset) => {
    update3dtiles(tileset);
    init(viewer);
    viewer.zoomTo(tileset);
  });
});
</script>

<style scoped>
#cesiumContainer {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}
#showDetailInfo {
  position: fixed;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  z-index: 999;
}
#container {
  padding: 100px;
}
.close {
  position: fixed;
  right: 20px;
  top: 10px;
}
.mask {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: #00000088;
  z-index: 998;
}
</style>
