<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>
    @import url("https://fonts.googleapis.com/css2?family=BIZ+UDGothic&display=swap");
:root {
  --x: 90vw;
  --y: 100vh;
  --duration: 3s;
}
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
html,
body {
  background-color: white;
  overscroll-behavior-x: none;
  overscroll-behavior-y: none;
  overflow: hidden;
}
body {
  width: 100vw;
  height: 100vh;
  font-family: serif;
  color: black;
  text-align: center;
  line-height: 1;
}
main {
  position: relative;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  /*
  background: url(https://images.unsplash.com/photo-1561948955-570b270e7c36?ixlib=rb-4.0.3&q=85&fm=jpg&crop=entropy&cs=srgb&w=1920)
    no-repeat center 35% / cover;
*/
}
main img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: center 33%;
}
main img:nth-of-type(2) {
  object-position: center center;
  animation: slide var(--duration) steps(1) infinite;
}
#myCanvas {
  position: absolute;
  margin-left: calc(-100vw + var(--x));
  margin-top: calc(-100vh + var(--y));
  animation: pos var(--duration) steps(1) infinite;
}
@keyframes slide {
  0% {
    opacity: 0;
  }
  50% {
    opacity: 1;
  }
  100% {
    opacity: 0;
  }
}
@keyframes pos {
  0% {
    --x: 90vw;
    --y: 100vh;
    transform: scale(1);
  }
  50% {
    --x: 100vw;
    --y: 110vh;
    transform: scale(0.8);
  }
  100% {
    --x: 90vw;
    --y: 100vh;
    transform: scale(1);
  }
}

  </style>
</head>
<body>

<main>
  <img src='https://images.unsplash.com/photo-1561948955-570b270e7c36?ixlib=rb-4.0.3&q=85&fm=jpg&crop=entropy&cs=srgb&w=1920' />
  <img src='https://images.unsplash.com/photo-1621451683587-8be65b8b975c?ixlib=rb-4.0.3&q=85&fm=jpg&crop=entropy&cs=srgb&w=1920' />

  <canvas id="myCanvas"></canvas>
</main>
<script>
  /*!
Speed lines 集中線

*/
/*!
forked from nekoneko-wanwan's "Canvasで漫画の集中線を描く" 
https://codepen.io/nekoneko-wanwan/pen/xwRjbq
*/
"use strict";
console.clear();

var cs = document.getElementById("myCanvas");
var ctx = cs.getContext("2d");
let w, h;
let timerID = null;

/*
//一旦、保留中
var root = document.querySelector(":root");
function spotlight(e) {
  let x = e.pageX * 2;
  let y = e.pageY * 2;
  if (x < 0) x = 0;
  if (x > w) x = w;
  if (y < 0) y = 0;
  if (y > h) y = h;
  root.style.setProperty("--x", x + "px");
  root.style.setProperty("--y", y + "px");
}
window.addEventListener("pointermove", spotlight);
window.addEventListener("pointerdown", spotlight);
*/

/*!*
 * 集中線メーカー
 * @param {obj} canvas object
 * @param {number} centralX: 集中線を配置するx座標
 * @param {number} centralY: 集中線を配置するy座標
 * @param {number} lineWidth: 線の太さ（ランダムの上限）
 * @param {number} lineNum: 線の数
 * @param {number} circleRadiusMax: 集中線の円形の半径上限
 * @param {number} circleRadiusMin: 集中線の円形の半径下限
 * @param {color} lineColor: 集中線の色

Copyright (c) 2023 by nekoneko-wanwan (https://codepen.io/nekoneko-wanwan/pen/xwRjbq)
*/
var focusLine = function (
  cs,
  centralX,
  centralY,
  lineWidth,
  lineNum,
  circleRadiusMax,
  circleRadiusMin,
  lineColor
) {
  var lines = [];

  // canvasの中心から角までの斜辺距離を円の半径とする
  var csRadius =
    Math.sqrt(Math.pow(cs.width / 2, 2) + Math.pow(cs.height / 2, 2)) | 0;

  /**
   * ランダムな整数を返す
   * @param max 最大値
   * @param min 最小値
   * @return min ~ max
   */
  var getRandomInt = function (max, min) {
    return Math.floor(Math.random() * (max - min)) + min;
  };

  /**
   * 円周上の座標を返す
   * @param d 角度
   * @param r 半径
   * @param cx, cy 中心座標
   */
  var getCircumPos = {
    // x座標
    x: function (d, r, cx) {
      return Math.cos((Math.PI / 180) * d) * r + cx;
    },
    // y座標
    y: function (d, r, cy) {
      return Math.sin((Math.PI / 180) * d) * r + cy;
    }
  };

  /**
   * @constructor
   */
  var Liner = function () {
    this.initialize();
  };
  Liner.prototype = {
    /* initialize()からsetPos()へ値を移すと、アニメーションの動きが変わる */
    initialize: function () {
      this.deg = getRandomInt(360, 0);
    },
    setPos: function () {
      this.moveDeg = this.deg + getRandomInt(lineWidth, 1) / 10;
      this.endRadius = getRandomInt(circleRadiusMax, circleRadiusMin);

      // 開始座標
      this.startPos = {
        x: getCircumPos.x(this.deg, csRadius, centralX),
        y: getCircumPos.y(this.deg, csRadius, centralY)
      };

      // 移動座標
      this.movePos = {
        x: getCircumPos.x(this.moveDeg, csRadius, centralX),
        y: getCircumPos.y(this.moveDeg, csRadius, centralY)
      };

      // 終了座標
      this.endPos = {
        x: getCircumPos.x(this.moveDeg, this.endRadius, centralX),
        y: getCircumPos.y(this.moveDeg, this.endRadius, centralY)
      };
    },
    update: function () {
      this.setPos();
    },
    draw: function () {
      ctx.beginPath();
      ctx.lineWidth = 1;
      ctx.fillStyle = lineColor;
      ctx.moveTo(this.startPos.x, this.startPos.y);
      ctx.lineTo(this.movePos.x, this.movePos.y);
      ctx.lineTo(this.endPos.x, this.endPos.y);
      ctx.fill();
      ctx.closePath();
    },
    render: function () {
      this.update();
      this.draw();
    }
  };

  /**
   * 線インスタンスの作成
   * @return lines[instance, instance...];
   */
  function createLines(num) {
    var i = 0;
    for (; i < num; i++) {
      lines[lines.length] = new Liner();
    }
  }

  /**
   * 描画
   */
  function render() {
    var i = 0;
    var l = lines.length;
    ctx.clearRect(0, 0, cs.width, cs.height);
    for (; i < l; i++) {
      lines[i].render();
    }
    timerID = setTimeout(function () {
      render();
    }, 100);
  }

  createLines(lineNum);
  render();
};

/**
 * focusLine()に渡す引数の設定
 */
function init() {
  w = window.innerWidth * 2;
  h = window.innerHeight * 2;

  cs.width = w;
  cs.height = h;
  var conf = {
    cx: w / 2,
    cy: h / 2,
    lineWidth: 10,
    lineNum: 220,
    crMax: Math.min(w, h) / 2 / 2, // 集中線の円形の半径上限
    crMin: Math.min(w, h) / 3 / 2, // 集中線の円形の半径下限
    color: "black"
  };

  focusLine(
    cs,
    conf.cx,
    conf.cy,
    conf.lineWidth,
    conf.lineNum,
    conf.crMax,
    conf.crMin,
    conf.color
  );
}
init();

//
window.addEventListener("resize", onWindowResize);
function onWindowResize() {
  ctx.clearRect(0, 0, cs.width, cs.height);

  clearTimeout(timerID);
  timerID = null;
  init();
}

</script>
</body>
</html>

# 𝙃𝙚𝙡𝙡𝙤, 𝙄'𝙢 Ayoub Hanaf

[![](https://img.shields.io/badge/-@ahanaf-%231DA1F2?style=flat-square&logo=twitter&logoColor=ffffff)](https://twitter.com/)
[![](https://img.shields.io/badge/-@ahanaf-%23181717?style=flat-square&logo=github)](https://github.com/dev-hanaf)
[![](https://img.shields.io/badge/-@ahanaf-%23000000?style=flat-square&logo=codepen)](https://codepen.io/)
[![](https://img.shields.io/badge/-@ahanaf-%23000000?style=flat-square&logo=codesandbox)](https://codesandbox.io/u/)
[![](https://img.shields.io/website?color=0ab9e6&style=flat-square&up_message=ahanaf.me&url=https%3A%2F%2Fxlbd.me)](https://)


[![ahanaf's 42 stats](https://badge.mediaplus.ma/binary/ahanaf)](https://github.com/oakoudad/badge42)

## 𝗠𝘆 𝗧𝗲𝗰𝗸 𝗦𝘁𝗮𝗰𝗸

![HTML5](https://img.shields.io/badge/-HTML5-%23E44D27?style=flat-square&logo=html5&logoColor=ffffff)
![CSS3](https://img.shields.io/badge/-CSS3-%231572B6?style=flat-square&logo=css3)
![JavaScript](https://img.shields.io/badge/-JavaScript-%23F7DF1C?style=flat-square&logo=javascript&logoColor=000000&labelColor=%23F7DF1C&color=%23FFCE5A)
![React.js](https://img.shields.io/badge/-React.js-%23282C34?style=flat-square&logo=react)

![Sass](https://img.shields.io/badge/-Sass-%23CC6699?style=flat-square&logo=sass&logoColor=ffffff)
![TailwindCSS](https://img.shields.io/badge/-TailwindCSS-%231a202c?style=flat-square&logo=tailwind-css)

![Git](https://img.shields.io/badge/-Git-%23F05032?style=flat-square&logo=git&logoColor=%23ffffff)
![GitLab](https://img.shields.io/badge/-GitLab-FCA121?style=flat-square&logo=gitlab)
![VS Code](https://img.shields.io/badge/-VSCode-%23007ACC?style=flat-square&logo=visual-studio-code)

## ⚡ GitHub Stats

<img align="left" src="https://github-readme-stats.vercel.app/api?username=dev-hanaf&show_icons=true&count_private=true&theme=dracula" />
<img src="https://github-readme-stats.vercel.app/api/top-langs/?username=dev-hanaf&layout=compact&count_private=true&theme=dracula" />
