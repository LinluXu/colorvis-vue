<!--
 * @Description:
 * @Version: 2.0
 * @Author: yichuanhao
 * @Date: 2023-04-23 11:49:04
 * @LastEditors: yichuanhao 1274816963@qq.com
 * @LastEditTime: 2023-05-16 22:09:56
-->
<template>
  <div class="threeDPage">
    <div class="webgl">
      <canvas id="webgl"></canvas>
    </div>
    <div class="info" id="info">
      <p class="title-top">
        <span>HEX：</span>
        <span id="name" class="c"></span>
      </p>
      <p>
        <span class="t">l：</span>
        <span id="l" class="c"></span>
        <span style="margin-left: 10px">x：</span>
        <span id="x" class="c"></span>
      </p>
      <p>
        <span class="t">a：</span>
        <span id="a" class="c"></span>
        <span style="margin-left: 10px">y：</span>
        <span id="y" class="c"></span>
      </p>
      <p>
        <span class="t">b：</span>
        <span id="b" class="c"></span>
        <span style="margin-left: 10px">z：</span>
        <span id="z" class="c"></span>
      </p>
      <p>
        <span>颜色名称：</span>
        <span id="colorName" class="c"></span>
      </p>
      <p>
        <span>色值来源：</span>
        <span id="from" class="c"></span>
      </p>
      <div class="other-box">
        <p>
          <span>近似色名：</span>
          <span id="otherInfo" class="c"></span>
        </p>
        <p style="margin-left: 40px">
          <span>距离：</span>
          <span id="otherInfo2" class="c"></span>
        </p>
      </div>
    </div>
    <commonDialog dialogTitle="使用帮助 " dialogWidth="600px" :isShowFooterBtn="false" :modal="true" ref="dialog">
      <p>
        ·计算方法：trimesh选取最外围顶点绘制几何体，优点是可以最大范围涵盖点，但易受离散值影响；推荐三级展示使用。alphashape绘制的几何体考虑了离散值，但对于某些点分布不均、数量较少的色名影响较大；推荐二级展示使用。<br />
        <br />
        ·搜索颜色：输入Lab值进行查找，画面中将展示输入值的点，并自动展示与输入值接近的色名，可以在“近似色名查看”框内选择或清除颜色。<br />
        <br />
        ·一级色名：以点来展示五大色调倾向 <br />
        <br />
        ·二级色名：展示了正色“青｜赤｜黄｜白｜黑”与间色“绿｜碧｜红｜紫｜流黄”的色域范围。<br />
        <br />
        ·三级色名：展示了可以确定色域的色名 <br />
        <br />
        ·近似色名查看：可在此搜索展示三级的所有色名（包括无法形成几何体的色名，以点展示）<br />
        <br />
        ·线框展示：开启后可以线框形式展示，便于比较。
      </p>
    </commonDialog>
    <el-drawer :visible.sync="drawer" direction="rtl" custom-class="threeD" :modal="false">
      <!-- 帮助 -->
      <div style="width: 220px; margin: 0 auto"><i class="el-icon-question question_icon" @click="$refs.dialog.showDialog = true"></i></div>
      <!-- 模式 -->
      <span style="display: inline-block; width: 220px; color: #000; font-size: 14px; margin: 10px auto 0px; text-align: left">计算方法：</span>
      <el-select v-model="functionType" placeholder="请选择一级色名类别" size="small" @change="functionTypeChange">
        <el-option label="Trimesh方法" :value="1">Trimesh方法</el-option>
        <el-option label="Alpha Shape方法" :value="2">Alpha Shape方法</el-option>
      </el-select>
      <span style="display: inline-block; width: 220px; color: #000; font-size: 14px; margin: 10px auto 0px; text-align: left">搜索颜色：</span>
      <el-input clearable v-model="colorValue" placeholder="请输入想查找的CIELab颜色值" size="small" @change="changeColor"> </el-input>
      <!-- 一级颜色 -->
      <span style="display: inline-block; width: 220px; color: #000; font-size: 14px; margin: 10px auto 0px; text-align: left">一级色名类别：</span>
      <el-select
        v-model="firstColorValue"
        placeholder="请选择一级色名类别"
        size="small"
        multiple
        collapse-tags
        filterable
        clearable
        @change="firstLevelChange"
      >
        <el-option v-for="(item, index) in firstLevelList" :key="index" :label="item" :value="item"> </el-option>
      </el-select>
      <!-- 二级颜色 -->
      <span style="display: inline-block; width: 220px; color: #000; font-size: 14px; margin: 10px auto 0px; text-align: left">二级色名类别：</span>
      <el-select
        v-model="secondColorValue"
        placeholder="请选择二级色名类别"
        size="small"
        multiple
        collapse-tags
        filterable
        clearable
        @change="secondLevelChange"
      >
        <el-option v-for="(item, index) in secondLevelList" :key="index" :label="item" :value="item"> </el-option>
      </el-select>
      <!-- 三级颜色 -->
      <span style="display: inline-block; width: 220px; color: #000; font-size: 14px; margin: 10px auto 0px; text-align: left">三级色名类别：</span>
      <el-select
        v-model="thirdColorValue"
        placeholder="请选择或输入三级色名"
        size="small"
        multiple
        filterable
        collapse-tags
        clearable
        @change="thirdLevelChange"
      >
        <el-option v-for="(item, index) in thirdLevelList" :key="index" :label="item" :value="item"> </el-option>
      </el-select>
      <!-- 其他颜色 -->
      <span style="display: inline-block; width: 220px; color: #000; font-size: 14px; margin: 10px auto 0px; text-align: left">近似色名查看：</span>
      <el-select v-model="otherColor" placeholder="相似色名" size="small" multiple filterable clearable @change="otherColorChange" value-key="color">
        <el-option v-for="(item, index) in allData" :key="index" :label="item.color" :value="item"> </el-option>
      </el-select>
      <!-- 是否以线框展示 -->
      <div class="switch">
        <span style="color: #000; margin: 10px; width: 220px; font-size: 14px; auto 0px; text-align: left">以线框展示:</span>
        <el-switch
          v-model="status"
          active-color="#13ce66"
          inactive-color="#D9D9D9"
          :active-text="status ? '是' : '否'"
          @change="statusChange"
        ></el-switch>
      </div>
    </el-drawer>
    <div class="fixed" @click="drawer = true">
      <i class="el-icon-s-operation"></i>
    </div>
  </div>
</template>

<script>
import { colord, extend } from 'colord';
import * as THREE from 'three';
import { ConvexGeometry } from 'three/addons/geometries/ConvexGeometry.js';
import { FontLoader } from 'three/addons/loaders/FontLoader.js';
import { TextGeometry } from 'three/addons/geometries/TextGeometry.js';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import commonDialog from './commonDialog.vue';
import { map } from '../utils/utils.js';
import labPlugin from 'colord/plugins/lab';
import Papa from 'papaparse';
import alphaShape from 'alpha-shape';
extend([labPlugin]);
let scene = null;
let canvas = null;
let camera = null;
let selectedObject = null;
export default {
  name: 'threeDPage', // 3D
  props: {
    backGroudColor: {
      type: String,
      default: '#808080',
    },
  },
  components: {
    commonDialog,
  },
  data() {
    return {
      functionType: 1,
      drawer: false,
      firstLevelList: [], // 第一层级列表
      secondLevelList: [], // 第二层级列表
      thirdLevelList: [], // 第三层级列表
      firstColorValue: [], // 第一层级选中的值
      secondColorValue: [], // 第二层级选中的值
      thirdColorValue: [], // 第三层级选中的值
      allData: [],
      group: null, // 用于管理mesh对象
      status: false,
      colorValue: '',
      textGroup: null,
      loader: null,
      firstLevel: [],
      secondLevel: [],
      thirdLevel: [],
      otherColor: [],
      approximationArr: [],
      thirdLevelPoint_alphashape: [],
      secondLevel_alphashape: [],
    };
  },
  methods: {
    // 模式切换
    functionTypeChange(val) {
      this.thirdLevelChange(this.thirdColorValue);
    },
    // 一层级切换
    firstLevelChange(val, flag = false) {
      if (!flag) {
        if (this.group.children.length > 0) this.group.clear();
        this.secondLevelChange(this.secondColorValue, true);
        this.thirdLevelChange(this.thirdColorValue, true);
        this.changeColor(this.colorValue, true);
        this.otherColorChange(this.otherColor, true);
      }
      val.forEach((item) => {
        let colorsData = this.firstLevel.filter((d) => d.parent === item);
        let points = [];
        colorsData.forEach((d, i) => {
          const c = colord({ l: d.l, a: d.a, b: d.b }).toHex();
          const y = map(d.l, 0, 100, 0, 1, true);
          const x = map(d.b, -128, 127, 0, 1, true);
          const z = map(d.a, -128, 127, 0, 1, true);
          let position = new THREE.Vector3(x, y, z);
          // 存入每个点坐标位置
          points.push(position);
          // 创建球体材质
          let convexSphere = this.creatSphere(position, c);
          convexSphere.name = d.color;
          this.group.add(convexSphere);
        });
      });
      scene.add(this.group);
    },
    // 二层级切换
    secondLevelChange(val, flag = false) {
      if (!flag) {
        if (this.group.children.length > 0) this.group.clear();
        this.thirdLevelChange(this.thirdColorValue, true);
        this.firstLevelChange(this.firstColorValue, true);
        this.changeColor(this.colorValue, true);
        this.otherColorChange(this.otherColor, true);
      }
      val.forEach((item) => {
        let colorsData =
          this.functionType == 1 ? this.secondLevel.filter((d) => d.parent === item) : this.secondLevel_alphashape.filter((d) => d.parent === item);
        let points = [];
        colorsData.forEach((d, i) => {
          const c = colord({ l: d.l, a: d.a, b: d.b }).toHex();
          const y = map(d.l, 0, 100, 0, 1, true);
          const x = map(d.b, -128, 127, 0, 1, true);
          const z = map(d.a, -128, 127, 0, 1, true);
          let position = new THREE.Vector3(x, y, z);
          // 存入每个点坐标位置
          points.push(position);
          // 创建球体材质
          let convexSphere = this.creatSphere(position, c);
          convexSphere.name = d.color;
          this.group.add(convexSphere);
        });
        let convexGeoMesh = this.creatGeometry(points);
        this.group.add(convexGeoMesh);
      });
      scene.add(this.group);
    },
    // 三层级切换
    thirdLevelChange(val, flag = false) {
      if (!flag) {
        if (this.group.children.length > 0) this.group.clear();
        this.secondLevelChange(this.secondColorValue, true);
        this.firstLevelChange(this.firstColorValue, true);
        this.changeColor(this.colorValue, true);
        this.otherColorChange(this.otherColor, true);
      }
      val.forEach((item) => {
        let colorsData =
          this.functionType == 1
            ? this.thirdLevel.filter((d) => d.parent === item)
            : this.thirdLevelPoint_alphashape.filter((d) => d.parent === item);
        let points = [];
        let otherPoints = [];
        colorsData.forEach((d, i) => {
          const c = colord({ l: d.l, a: d.a, b: d.b }).toHex();
          const y = map(d.l, 0, 100, 0, 1, true);
          const x = map(d.b, -128, 127, 0, 1, true);
          const z = map(d.a, -128, 127, 0, 1, true);
          let position = new THREE.Vector3(x, y, z);
          // 存入每个点坐标位置
          points.push(position);
          otherPoints.push([x, y, z]);
          // 创建球体材质
          let convexSphere = this.creatSphere(position, c);
          convexSphere.name = d.color;
          convexSphere.from = d.from;
          this.group.add(convexSphere);
        });
        let convexGeoMesh = this.functionType == 1 ? this.creatGeometry(points) : this.createAlphaShape(otherPoints);
        this.group.add(convexGeoMesh);
      });
      scene.add(this.group);
    },
    // 创建球体
    creatSphere(position, color) {
      const convexSphereGeo = new THREE.SphereGeometry(0.01);
      const convexSphereMat = new THREE.MeshBasicMaterial({
        color: color,
      });
      const convexSphere = new THREE.Mesh(convexSphereGeo, convexSphereMat);
      convexSphere.position.copy(position);
      return convexSphere;
    },

    // 创建几何体
    creatGeometry(points) {
      // 创建凹凸几何体
      var convexGeo = new ConvexGeometry(points);
      let p = [];
      // 存入顶点位置
      convexGeo.attributes.position.array.forEach((i) => {
        p.push(i);
      });
      const size = 3; // 每个子数组的长度
      const result = [];
      // 每3个拆成二维数组，取出每个顶点的x,y,z坐标
      for (let i = 0; i < p.length; i += size) {
        result.push(p.slice(i, i + size));
      }
      let colors = [];
      // 每个lab值转换成着色器需要的格式
      result.forEach((i) => {
        let l = map(i[1], 0, 1, 0, 100, true);
        let a = map(i[2], 0, 1, -128, 127, true);
        let b = map(i[0], 0, 1, -128, 127, true);
        let c = colord({ l: l, a: a, b: b }).toHex();
        colors = colors.concat(this.colorToRgb(c));
      });

      // 给凹凸几何体增加color属性
      convexGeo.setAttribute('color', new THREE.Float32BufferAttribute(colors, 3));
      if (!this.status) {
        // 着色器
        var material = new THREE.ShaderMaterial({
          vertexShader: `
          varying vec2 vUv;
          varying vec3 vColor;
          attribute vec3 color;
          void main() {
              vColor = color;
              vUv = uv;
              gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
          }
      `,
          fragmentShader: `
          varying vec2 vUv;
          varying vec3 vColor;
          void main() {
            gl_FragColor=vec4(vColor, 1.0); //最后设置顶点颜色，点与点之间会自动插值
          }
      `,
        });
        // 创建一个网格，将凹凸体和GLSL着色器结合起来
        return new THREE.Mesh(convexGeo, material);
      } else {
        // 给凹凸几何体增加color属性
        convexGeo.setAttribute('color', new THREE.Float32BufferAttribute(colors, 3));
        const testM = new THREE.MeshBasicMaterial({
          wireframe: true,
        });
        return new THREE.Mesh(convexGeo, testM);
      }
    },
    // 创建alphaShape
    createAlphaShape() {
      const points = [
        [0, 0, 0],
        [0, 0, 1],
        [0, 1, 0],
        [0, 1, 1],
        [1, 0, 0],
        [1, 0, 1],
        [1, 1, 0],
        [1, 1, 1],
      ];
      const triangles = alphaShape(0.1, points);
      const geometry = new THREE.BufferGeometry();
      // 将 AlphaShape 的结果转换为 Three.js 中的 Vector3 对象
      for (let i = 0; i < triangles.length; i++) {
        const t = triangles[i];
        geometry.vertices.push(
          new THREE.Vector3(points[t[0]][0], points[t[0]][1], points[t[0]][2]),
          new THREE.Vector3(points[t[1]][0], points[t[1]][1], points[t[1]][2]),
          new THREE.Vector3(points[t[2]][0], points[t[2]][1], points[t[2]][2]),
        );

        // 创建三角形面并向 Geometry 添加
        const faceIdx = i * 3;
        geometry.faces.push(new THREE.Face3(faceIdx, faceIdx + 1, faceIdx + 2));
      }

      // 根据 Geometry 对象创建 Mesh 对象，然后添加到场景中
      const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
      new THREE.Mesh(geometry, material);
    },
    colorToRgb(sColor) {
      sColor = sColor.toLowerCase();
      //十六进制颜色值的正则表达式
      var reg = /^#([0-9a-fA-f]{3}|[0-9a-fA-f]{6})$/;
      // 如果是16进制颜色
      if (sColor && reg.test(sColor)) {
        if (sColor.length === 4) {
          var sColorNew = '#';
          for (var i = 1; i < 4; i += 1) {
            sColorNew += sColor.slice(i, i + 1).concat(sColor.slice(i, i + 1));
          }
          sColor = sColorNew;
        }
        //处理六位的颜色值
        var sColorChange = [];
        for (var i = 1; i < 7; i += 2) {
          sColorChange.push(parseInt('0x' + sColor.slice(i, i + 2)) / 255);
        }
        return sColorChange;
      }
      return sColor;
    },
    statusChange() {
      if (this.group.children.length > 0) this.group.clear();
      this.firstLevelChange(this.firstColorValue, true);
      this.secondLevelChange(this.secondColorValue, true);
      this.thirdLevelChange(this.thirdColorValue, true);
      this.changeColor(this.colorValue, true);
      this.otherColorChange(this.otherColor, true);
    },
    distanceFn(point1, point2) {
      let dx = point2.b - point1.b;
      let dy = point2.l - point1.l;
      let dz = point2.a - point1.a;
      return {
        distance: Number(Math.sqrt(dx * dx + dy * dy + dz * dz).toFixed(3)),
        ...point2,
      };
    },

    otherColorChange(val, flag = false) {
      if (!flag) {
        if (this.group.children.length > 0) this.group.clear();
        this.thirdLevelChange(this.thirdColorValue, true);
        this.firstLevelChange(this.firstColorValue, true);
        this.secondLevelChange(this.secondColorValue, true);
        this.changeColor(this.colorValue, true);
      }
      val.forEach((d) => {
        const c = colord({ l: d.l, a: d.a, b: d.b }).toHex();
        const y = map(d.l, 0, 100, 0, 1, true);
        const x = map(d.b, -128, 127, 0, 1, true);
        const z = map(d.a, -128, 127, 0, 1, true);
        let position = new THREE.Vector3(x, y, z);
        // 创建球体材质
        let convexSphere = this.creatSphere(position, c);
        convexSphere.name = d.color;
        convexSphere.from = d.from;
        this.group.add(convexSphere);
      });
      scene.add(this.group);
    },
    changeOther() {
      this.otherColor.forEach((d) => {
        const c = colord({ l: d.l, a: d.a, b: d.b }).toHex();
        const y = map(d.l, 0, 100, 0, 1, true);
        const x = map(d.b, -128, 127, 0, 1, true);
        const z = map(d.a, -128, 127, 0, 1, true);
        let position = new THREE.Vector3(x, y, z);
        // 创建球体材质
        let convexSphere = this.creatSphere(position, c);
        convexSphere.name = d.color;
        convexSphere.from = d.from;
        this.group.add(convexSphere);
      });
      scene.add(this.group);
    },
    // 创建球体
    creatOtherSphere(position, color) {
      const convexSphereGeo = new THREE.ConeGeometry(0.01, 0.01, 32);
      const convexSphereMat = new THREE.MeshBasicMaterial({
        color: color,
      });
      const convexSphere = new THREE.Mesh(convexSphereGeo, convexSphereMat);
      convexSphere.position.copy(position);
      return convexSphere;
    },
    changeColor(val, flag = false) {
      let reg = /^(100|\d{1,2}),(0|-?(12[0-8]|1[01][0-9]|[1-9][0-9]?)),(0|-?(12[0-8]|1[01][0-9]|[1-9][0-9]?))$/;
      if (!flag) {
        if (this.group.children.length > 0) this.group.clear();
        // this.thirdLevelChange(this.thirdColorValue, true);
        this.firstLevelChange(this.firstColorValue, true);
        this.secondLevelChange(this.secondColorValue, true);
        // this.otherColorChange(this.otherColor, true);
      }
      if (flag && reg.test(val)) {
        let arr = val.split(',');
        const c = colord({ l: arr[0], a: arr[1], b: arr[2] }).toHex();
        const y = map(arr[0], 0, 100, 0, 1, true); // l
        const z = map(arr[1], -128, 127, 0, 1, true); // a
        const x = map(arr[2], -128, 127, 0, 1, true); // b
        let position = new THREE.Vector3(x, y, z);
        // 存入每个点坐标位置
        let convexSphere = this.creatOtherSphere(position, c);
        let disteanceArr = [];
        this.allData.forEach((item) => {
          let value = this.distanceFn(
            {
              b: arr[2],
              l: arr[0],
              a: arr[1],
            },
            item,
          );
          disteanceArr.push(value);
        });
        disteanceArr.sort((a, b) => a.distance - b.distance);
        this.approximationArr = disteanceArr.slice(0, 3); // 截取前三个
        convexSphere.otherInfo = this.approximationArr;
        convexSphere.name = '输入值';
        this.group.add(convexSphere);
        scene.add(this.group);
        return;
      }
      this.approximationArr = [];
      document.querySelector('#otherInfo').innerHTML = '';
      document.querySelector('#otherInfo2').innerHTML = '';
      if (reg.test(val)) {
        let arr = val.split(',');
        const c = colord({ l: arr[0], a: arr[1], b: arr[2] }).toHex();
        const y = map(arr[0], 0, 100, 0, 1, true); // l
        const z = map(arr[1], -128, 127, 0, 1, true); // a
        const x = map(arr[2], -128, 127, 0, 1, true); // b
        let position = new THREE.Vector3(x, y, z);
        // 存入每个点坐标位置
        let convexSphere = this.creatOtherSphere(position, c);
        let disteanceArr = [];
        this.allData.forEach((item) => {
          let value = this.distanceFn(
            {
              b: arr[2],
              l: arr[0],
              a: arr[1],
            },
            item,
          );
          disteanceArr.push(value);
        });
        this.otherColor = [];
        disteanceArr.sort((a, b) => a.distance - b.distance);
        this.approximationArr = disteanceArr.slice(0, 3); // 截取前三个
        convexSphere.otherInfo = this.approximationArr;
        convexSphere.name = '输入值';
        this.group.add(convexSphere);
        scene.add(this.group);
        let str = '';
        let str1 = '';
        this.approximationArr.forEach((item) => {
          str += item.color + '</br>';
          str1 += item.distance + '</br>';
        });
        this.approximationArr.forEach((item) => {
          let colorsData = this.thirdLevel.filter((d) => d.parent === item.color);
          if (colorsData.length > 0) {
            if (this.thirdColorValue.indexOf(item.color) == -1) {
              this.thirdColorValue.push(item.color);
            }
          } else {
            this.otherColor.push(item);
          }
        });
        this.changeOther(this.otherColor, true);
        this.thirdLevelChange(this.thirdColorValue, true);
        document.querySelector('#otherInfo').innerHTML = str;
        document.querySelector('#otherInfo2').innerHTML = str1;
        if (!flag) {
          this.$message({
            showClose: true,
            message: '添加成功',
            type: 'success',
          });
        }
      } else {
        if (flag) return;
        this.$message({
          showClose: true,
          message: '请使用英文输入法输入正确Lab颜色格式,格式为 0,0,0',
          type: 'error',
        });
      }
    },
    parseCsvData(url, key) {
      Papa.parse(url, {
        download: true,
        header: true,
        complete: (results) => {
          if (key == 'firstLevelList') {
            this.firstLevel = results.data;
          } else if (key == 'secondLevelList') {
            this.secondLevel = results.data;
          } else if (key == 'thirdLevelList') {
            this.thirdLevel = results.data;
          } else if (key == 'thirdLevelPoint_alphashape') {
            this.thirdLevelPoint_alphashape = results.data;
            return;
          } else if (key == 'secondLevel_alphashape') {
            this.secondLevel_alphashape = results.data;
            return;
          } else {
            this.allData = results.data;
            return;
          }
          this[key] = this.formatData(results.data);
        },
      });
    },
    // 处理数据给下拉框
    formatData(data) {
      let op = [];
      data.forEach((d) => {
        let name = d.parent;
        if (name) op.push(name);
      });
      op = Array.from(new Set(op));
      return op;
    },
    textLoader(text, position, translateType, fontColor = 0xbebebe, fontSize = 0.02) {
      let that = this;
      // text文本、positon（x,y,z）坐标
      this.loader.load(
        // resource URL
        '/SJgzks_Regular.json',
        // onLoad callback
        function (font) {
          var textGeometry = new TextGeometry(text, {
            font: font, // 字体
            size: fontSize, // 大小
            height: 0.0, // 厚度
          });
          switch (translateType) {
            case 'x':
              textGeometry.translate(position ? Number(position) : 1.1, position ? 0 : 0, -0.1);
              break;
            case 'y':
              textGeometry.translate(position ? -0.1 : -0.1, position ? Number(position) : 1.1, -0.1);
              break;
            case 'z':
              textGeometry.translate(position ? -0.1 : -0.1, 0, position ? Number(position) : 1.1);
              break;
          }
          // 创建一个文本网格
          var textMaterial = new THREE.MeshBasicMaterial({ color: fontColor });
          var textMesh = new THREE.Mesh(textGeometry, textMaterial);
          // 将文本网格添加到group中
          that.textGroup.add(textMesh);
        },
      );
    },
    initThreeD() {
      const pointer = new THREE.Vector2();
      const raycaster = new THREE.Raycaster();
      canvas = document.getElementById('webgl');
      scene = new THREE.Scene();
      scene.background = new THREE.Color('#808080');
      const geometry = new THREE.BoxGeometry(1, 1, 1);
      const geo = new THREE.EdgesGeometry(geometry);
      const mat = new THREE.LineBasicMaterial({
        color: 0xffffff,
        linewidth: 1,
      });
      const mesh = new THREE.LineSegments(geo, mat);
      mesh.position.set(0.5, 0.5, 0.5);
      scene.add(mesh);
      // 创建一个文本几何体
      this.loader = new FontLoader();
      this.textGroup = new THREE.Group();
      let coordinate = ['L*', 'a*', 'b*'];
      coordinate.forEach((item) => {
        this.textLoader(item, '', item == 'L*' ? 'y' : item == 'a*' ? 'z' : 'x', 0xc4c4c4, 0.03);
      });
      // 增加x、y、z刻度
      let l = ['-128', '10', '20', '30', '40', '50', '60', '70', '80', '90', '100'];
      let a = ['-102', '-76', '-50', '-24', '-2', '28', '44', '70', '96', '124'];
      let b = ['-102', '-76', '-50', '-24', '-2', '28', '44', '70', '96', '124'];
      l.forEach((item, index) => {
        this.textLoader(item, (index / 10).toFixed(1), 'y');
      });
      a.forEach((item, index) => {
        this.textLoader(item, ((index + 1) / 10).toFixed(1), 'z');
      });
      b.forEach((item, index) => {
        this.textLoader(item, ((index + 1) / 10).toFixed(1), 'x');
      });
      scene.add(this.textGroup); // 将刻度group放到场景中
      let dom = document.querySelector('.el-tabs__content');
      const size = {
        width: dom.offsetWidth,
        height: dom.offsetHeight,
      };
      // 创建相机
      camera = new THREE.PerspectiveCamera(45, size.width / size.height);
      camera.position.z = 1.5;
      camera.position.x = 1.5;
      camera.position.y = 1.5;
      scene.add(camera);
      const renderer = new THREE.WebGLRenderer({
        canvas: canvas,
        antialias: true,
      });
      renderer.setPixelRatio(window.devicePixelRatio);
      renderer.setSize(size.width, size.height);
      // 坐标轴
      let axisHelper = new THREE.AxesHelper(1.5);
      scene.add(axisHelper);
      animate();

      function animate() {
        render();
        requestAnimationFrame(animate);
      }

      window.addEventListener('resize', onWindowResize);
      function render() {
        renderer.render(scene, camera);
      }
      // 创建控制器
      const controls = new OrbitControls(camera, renderer.domElement);
      controls.addEventListener('change', render); // use if there is no animation loop
      controls.minDistance = 1;
      controls.maxDistance = 10;
      controls.target.set(0.5, 0.5, 0.5);
      controls.update();

      function onWindowResize() {
        camera.aspect = dom.offsetWidth / dom.offsetHeight;
        camera.updateProjectionMatrix();
        renderer.setSize(dom.offsetWidth, dom.offsetHeight);
      }
      canvas.addEventListener('pointermove', onPointerMove);
      let that = this;
      function onPointerMove(event) {
        setInfo('', 0, 0, 0, '', '', '');
        if (!that.group) return;
        if (selectedObject) {
          selectedObject.scale.set(0.01, 0.01, 0.01);
          selectedObject = null;
        }

        pointer.x = (event.clientX / window.innerWidth) * 2 - 1;
        pointer.y = -((event.clientY - 50) / (window.innerHeight - 50)) * 2 + 1;

        raycaster.setFromCamera(pointer, camera);

        const intersects = raycaster.intersectObject(that.group, true);

        if (intersects.length > 0) {
          const res = intersects.filter(function (res) {
            return res && res.object;
          })[0];
          if (res && res.point && res.object) {
            let l = map(res.point.y, 0, 1, 0, 100, true);
            let b = map(res.point.x, 0, 1, -128, 127, true);
            let a = map(res.point.z, 0, 1, -128, 127, true);
            setInfo(colord({ l: l, a: a, b: b }).toHex(), l, a, b, res.object.name, res.object.from ? res.object.from : '', res.point);
          }
        }
      }
      function setInfo(name, l, a, b, colorName, from, point) {
        document.querySelector('#name').innerHTML = name;
        document.querySelector('#name').style.color = name;
        document.querySelector('#l').innerHTML = Number(l).toFixed(2);
        document.querySelector('#a').innerHTML = Number(a).toFixed(2);
        document.querySelector('#b').innerHTML = Number(b).toFixed(2);
        document.querySelector('#colorName').innerHTML = colorName;
        document.querySelector('#from').innerHTML = from;
        if (point) {
          document.querySelector('#x').innerHTML = point.x.toFixed(2);
          document.querySelector('#y').innerHTML = point.y.toFixed(2);
          document.querySelector('#z').innerHTML = point.z.toFixed(2);
        } else {
          document.querySelector('#x').innerHTML = 0.0;
          document.querySelector('#y').innerHTML = 0.0;
          document.querySelector('#z').innerHTML = 0.0;
        }
      }
    },
  },
  created() {
    this.group = new THREE.Group(); // 创建group管理所有几何体和点
    this.parseCsvData('/assets/color/firstLevel.csv', 'firstLevelList');
    this.parseCsvData('/assets/color/secondLevel.csv', 'secondLevelList');
    this.parseCsvData('/assets/color/thirdLevel.csv', 'thirdLevelList');
    this.parseCsvData('/assets/color/allData.csv', 'allData');
    this.parseCsvData('/assets/color/thirdLevelPoint_alphashape.csv', 'thirdLevelPoint_alphashape');
    this.parseCsvData('/assets/color/secondLevel_alphashape.csv', 'secondLevel_alphashape');
  },
  mounted() {
    this.initThreeD();
  },
  watch: {
    backGroudColor: function (val) {
      scene.background = new THREE.Color(val ? val : '#808080');
    },
  },
};
</script>

<style lang="scss">
.threeDPage {
  color: #fff;

  .webgl {
    width: 100%;
    height: 100%;
  }
  .title-top {
    margin-top: 15px;
  }
  p {
    margin-bottom: 15px;
    text-align: left;
  }
  .info {
    position: absolute;
    top: 0;
    left: 20px;
    color: aliceblue;
    display: flex;
    flex-direction: column;
  }

  .fixed {
    position: absolute;
    right: 12px;
    top: 10px;
    i {
      font-size: 24px;
      text-shadow: 0px 0px 2px #000;
      cursor: pointer;
    }
  }
  .other-box {
    display: flex;
    align-content: flex-start;
    p {
      display: flex;
      flex-direction: column;
      justify-content: center;
      span {
        text-align: center;
      }
    }
  }
  p {
    .c {
      color: #cccccc;
    }
    .t {
      display: inline-block;
      width: 20px;
    }
    #l,
    #a,
    #b {
      display: inline-block;
      width: 50px;
    }
  }
}
.threeD {
  width: 240px !important;
  .el-input {
    width: 220px !important;
    // margin-top: 15px;
  }
  .switch {
    text-align: left;
    margin-top: 15px;
  }
  .el-select__tags {
    top: 66%;
  }
  .el-drawer__header {
    padding: 10px 6px 0;
  }
}
</style>
