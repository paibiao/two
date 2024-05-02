<template>
  <div id="cesiumContainer"></div>
</template>

<script setup>
import * as Cesium from "cesium";
import { onMounted } from "vue";
import { load3dtiles, update3dtiles } from "/src/utils/load3D.js";
import build from "/src/assets/build.json";
let viewer, handle, lastPick;
Cesium.Ion.defaultAccessToken =
  "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI5ZWZlNmFmNS0yNTkxLTQ2NzEtYjk4NS1lNGY5NWM1OTc5MmYiLCJpZCI6MjA1MTk1LCJpYXQiOjE3MTE2NzU4OTJ9.O9J1G_tirf33A-GKVOhxk1h9lWH08IZTnMOf-y8XS_U";

function init(viewer) {
  highPolygonInit(viewer);
  lowerPolygonInit(viewer);
  setMouseMoveEvent(viewer);
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
  viewer = new Cesium.Viewer("cesiumContainer", {});
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
</style>
