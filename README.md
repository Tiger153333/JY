<html>
  <head>
    <style>
      body {
        font-family: 'Arial Black', Gothic, sans-serif; /* 哥特风格字体 */
        text-align: center;
        margin: 0;
        padding: 0;
        background: linear-gradient(to bottom, #1a1a1a, #4a0000); /* 末日暗红渐变背景 */
        color: #fff;
        overflow-x: hidden; /* 防止横向滚动 */
        position: relative;
      }
      h1 {
        font-size: 48px;
        color: #ff3333; /* 血红标题 */
        text-shadow: 2px 2px 4px #000;
        margin-top: 20px;
      }
      .range-input {
        font-size: 20px;
        margin: 20px;
        color: #ddd;
      }
      input[type="number"] {
        width: 80px;
        padding: 5px;
        background: #333;
        border: 2px solid #ff3333;
        color: #ff3333;
        font-weight: bold;
      }
      #generateButton {
        background-color: #ff1a1a; /* 血红按钮 */
        color: white;
        padding: 12px 24px;
        border: none;
        border-radius: 5px;
        cursor: pointer;
        font-size: 18px;
        text-transform: uppercase;
        box-shadow: 0 0 10px #ff3333;
        margin: 10px;
      }
      #generateButton:hover {
        background-color: #cc0000;
      }
      #output {
        margin-top: 30px;
      }
      .zombie-number {
        font-size: 60px; /* 放大数字 */
        color: #ff0000; /* 血淋淋红色 */
        font-weight: bold;
        text-shadow: 3px 3px 6px #800000, -3px -3px 6px #ff3333; /* 哥特阴影 */
        margin: 10px;
      }
      .zombie-head {
        font-size: 30px;
        color: #ff6666;
        margin-top: 5px;
      }
      .zombie-head span {
        margin: 0 5px;
      }
      /* 模拟丧尸脑袋 */
      .zombie-head span::after {
        content: "🧟"; /* Unicode 僵尸图标，可替换为图片 */
      }
      /* 箭头和房子样式 */
      .arrow {
        font-size: 30px;
        color: #ff9999;
        margin: 10px 0;
      }
      .arrow::after {
        content: "↓"; /* 向下箭头 */
      }
      .shelter {
        font-size: 20px;
        color: #ffcccc;
        margin-bottom: 20px;
      }
      /* 横向排列 */
      .horizontal .zombie-group {
        display: inline-block;
        margin: 0 20px;
        vertical-align: top;
      }
      /* 竖向排列 */
      .vertical .zombie-group {
        display: block;
        margin: 20px 0;
      }
      /* 丧尸动画 */
      .zombie-swarm {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        pointer-events: none; /* 防止遮挡其他元素 */
        z-index: 10;
      }
      .zombie {
        position: absolute;
        animation: runAcross linear forwards;
      }
      @keyframes runAcross {
        0% {
          transform: translateX(100vw); /* 从屏幕右侧开始 */
        }
        100% {
          transform: translateX(-100vw); /* 跑到屏幕左侧 */
        }
      }
      /* 坠落动画 */
      .fall-animation {
        animation: fallDown 1s ease-out forwards;
        opacity: 0; /* 初始隐藏 */
      }
      @keyframes fallDown {
        0% {
          transform: translateY(-100px);
          opacity: 0;
        }
        100% {
          transform: translateY(0);
          opacity: 1;
        }
      }
    </style>
  </head>
  <body>
    <h1>有几只丧尸会去你家做客？</h1>
    <div class="range-input">
      从 
      <input type="number" id="rangeStart" value="1" min="0" style="width: 80px;"> 
      到 
      <input type="number" id="rangeEnd" value="4" min="0" style="width: 80px;"> 
      生成 
      <input type="number" id="count" value="5" min="1" style="width: 80px;"> 
      组随机丧尸数
    </div>
    <label><input type="checkbox" id="noRepeat"> 无重复项</label><br>
    <label><input type="checkbox" id="displayHorizontal"> 横向排列（不勾选则竖向排列）</label>
    <br><br>
    <button id="generateButton">召唤</button>
    <h3 style="color: #ff6666;">结果</h3>
    <div id="output"></div>

    <!-- 丧尸咆哮音效（默认） -->
    <audio id="zombieRoar" preload="auto">
      <source src="https://www.fesliyanstudios.com/play-mp3/7039" type="audio/mpeg">
      你的浏览器不支持音频播放。
    </audio>

    <!-- 音效更换按钮 -->
    <div id="soundControl" style="margin-top: 20px;">
      <button id="changeSound">更换音效</button>
      <span id="currentSound">当前音效：咆哮</span>
    </div>

    <script>
      let soundIndex = 0;
      const soundOptions = [
        { src: "https://www.fesliyanstudios.com/play-mp3/7039", name: "咆哮" }, // 默认咆哮
        { src: "https://www.fesliyanstudios.com/play-mp3/7040", name: "尖叫" }, // 替换为尖叫音效
        { src: "https://www.fesliyanstudios.com/play-mp3/7041", name: "笑声" }  // 替换为笑声音效
      ];

      document.getElementById("generateButton").addEventListener("click", function() {
        // 播放当前选中的音效
        const zombieRoar = document.getElementById("zombieRoar");
        zombieRoar.src = soundOptions[soundIndex].src; // 更新音效源
        zombieRoar.currentTime = 0; // 重置音效
        zombieRoar.play();

        // 创建丧尸动画
        const swarmContainer = document.createElement("div");
        swarmContainer.className = "zombie-swarm";
        document.body.appendChild(swarmContainer);

        // 计算垂直位置，避免重叠
        const zombieCount = 100;
        const vhPerZombie = 80 / zombieCount; // 将 10vh 到 90vh 分成 100 份
        const positions = Array.from({ length: zombieCount }, (_, i) => 10 + i * vhPerZombie);

        // 随机打乱位置
        for (let i = positions.length - 1; i > 0; i--) {
          const j = Math.floor(Math.random() * (i + 1));
          [positions[i], positions[j]] = [positions[j], positions[i]];
        }

        // 生成100只丧尸
        for (let i = 0; i < zombieCount; i++) {
          const zombie = document.createElement("div");
          zombie.className = "zombie";
          zombie.innerHTML = "🧟"; // 可替换为图片 <img src="zombie.png">
          
          // 随机大小（最小10px，最大200px，最大是最小的20倍）
          const minSize = 10;
          const maxSize = minSize * 20; // 200px
          const size = Math.random() * (maxSize - minSize) + minSize;
          zombie.style.fontSize = `${size}px`;
          
          // 垂直位置（避免重叠）
          zombie.style.top = `${positions[i]}vh`;
          
          // 随机速度（2秒到5秒）
          const duration = Math.random() * 3 + 2; // 2s到5s
          zombie.style.animationDuration = `${duration}s`;
          
          // 随机延迟（0到1秒）
          zombie.style.animationDelay = `${Math.random() * 1}s`;
          
          swarmContainer.appendChild(zombie);
        }

        // 5秒后移除丧尸并显示结果（最慢丧尸跑完）
        setTimeout(() => {
          swarmContainer.remove(); // 移除动画
          
          // 获取用户输入
          let rangeStart = parseInt(document.getElementById("rangeStart").value);
          let rangeEnd = parseInt(document.getElementById("rangeEnd").value);
          let count = parseInt(document.getElementById("count").value);
          let noRepeat = document.getElementById("noRepeat").checked;
          let displayHorizontal = document.getElementById("displayHorizontal").checked;

          // 输入验证
          if (isNaN(rangeStart) || isNaN(rangeEnd) || isNaN(count)) {
            document.getElementById("output").innerHTML = "请输入有效的数字！";
            return;
          }
          if (rangeStart > rangeEnd) {
            document.getElementById("output").innerHTML = "起始范围不能大于结束范围！";
            return;
          }
          if (noRepeat && count > (rangeEnd - rangeStart + 1)) {
            document.getElementById("output").innerHTML = "无重复模式下，生成数量不能大于范围内的数字个数！";
            return;
          }
          if (count <= 0) {
            document.getElementById("output").innerHTML = "生成数量必须大于 0！";
            return;
          }

          // 生成随机数
          let numbers = [];
          if (noRepeat) {
            let possibleNumbers = [];
            for (let i = rangeStart; i <= rangeEnd; i++) {
              possibleNumbers.push(i);
            }
            for (let i = 0; i < count; i++) {
              let randomIndex = Math.floor(Math.random() * possibleNumbers.length);
              numbers.push(possibleNumbers[randomIndex]);
              possibleNumbers.splice(randomIndex, 1);
            }
          } else {
            for (let i = 0; i < count; i++) {
              let randomNum = Math.floor(Math.random() * (rangeEnd - rangeStart + 1)) + rangeStart;
              numbers.push(randomNum);
            }
          }

          // 显示结果（添加坠落动画）
          let outputHtml = `<div class="${displayHorizontal ? 'horizontal' : 'vertical'}">`;
          numbers.forEach((num, index) => {
            outputHtml += `<div class="zombie-group fall-animation" style="display: inline-block;">`;
            outputHtml += `<span class="zombie-number">${num}</span><br>`;
            outputHtml += `<div class="zombie-head">`;
            for (let i = 0; i < num; i++) {
              outputHtml += `<span></span>`;
            }
            outputHtml += `</div>`;
            outputHtml += `<div class="arrow"></div>`;
            outputHtml += `<div class="shelter">第${index + 1}避难所</div>`;
            outputHtml += `</div>`;
          });
          outputHtml += `</div>`;
          document.getElementById("output").innerHTML = outputHtml;
        }, 5000); // 5秒后显示结果（最慢丧尸跑完）
      });

      // 音效更换按钮事件
      document.getElementById("changeSound").addEventListener("click", function() {
        soundIndex = (soundIndex + 1) % soundOptions.length; // 循环切换音效
        const currentSound = document.getElementById("currentSound");
        currentSound.textContent = `当前音效：${soundOptions[soundIndex].name}`;
      });

      // 控制丧尸音效音量
      const zombieRoar = document.getElementById("zombieRoar");
      zombieRoar.volume = 0.5;
    </script>
  </body>
</html>
