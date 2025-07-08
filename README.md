<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>色判定アプリ（パスワード付き）</title>
  <style>
    body { font-family: sans-serif; text-align: center; background: #f0f0f0; }
    #app { display: none; margin-top: 20px; }
    #video { width: 90%; max-width: 400px; border-radius: 8px; }
    #colorBox { width: 50px; height: 50px; margin: 10px auto; border: 1px solid #000; }
  </style>
</head>
<body>
  <!-- ログイン画面 -->
  <div id="login">
    <h2>ログイン</h2>
    <input type="password" id="password" placeholder="パスワードを入力">
    <button onclick="checkPassword()">ログイン</button>
    <p id="error" style="color:red;"></p>
  </div>

  <!-- アプリ本体 -->
  <div id="app">
    <h2>色判定アプリ</h2>
    <video id="video" autoplay playsinline></video>
    <canvas id="canvas" style="display:none;"></canvas>
    <div id="colorBox"></div>
    <p id="rgbValue"></p>
    <p id="judgment"></p>
  </div>

  <script>
    // パスワード認証
    const correctPassword = "1234";
    function checkPassword() {
      const input = document.getElementById("password").value;
      if (input === correctPassword) {
        document.getElementById("login").style.display = "none";
        document.getElementById("app").style.display = "block";
        startCamera();
      } else {
        document.getElementById("error").textContent = "パスワードが違います。";
      }
    }

    // カメラ起動
    async function startCamera() {
      const video = document.getElementById('video');
      const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'environment' }, audio: false });
      video.srcObject = stream;

      const canvas = document.getElementById('canvas');
      const context = canvas.getContext('2d');

      video.addEventListener('loadedmetadata', () => {
        canvas.width = video.videoWidth;
        canvas.height = video.videoHeight;

        setInterval(() => {
          context.drawImage(video, 0, 0, canvas.width, canvas.height);

          // 中央の20x20ピクセルの平均色を取得
          const size = 20;
          const x = canvas.width / 2 - size / 2;
          const y = canvas.height / 2 - size / 2;
          const imageData = context.getImageData(x, y, size, size);
          const data = imageData.data;

          let r = 0, g = 0, b = 0;
          for (let i = 0; i < data.length; i += 4) {
            r += data[i];
            g += data[i + 1];
            b += data[i + 2];
          }
          const count = data.length / 4;
          r = Math.round(r / count);
          g = Math.round(g / count);
          b = Math.round(b / count);

          // 表示更新
          document.getElementById("rgbValue").textContent = `RGB: (${r}, ${g}, ${b})`;
          document.getElementById("colorBox").style.backgroundColor = `rgb(${r},${g},${b})`;

          // 濃淡判定（暫定値）
          const brightness = 0.299 * r + 0.587 * g + 0.114 * b;
          let judgment = "";
          if (brightness < 90) {
            judgment = "濃い";
          } else if (brightness > 180) {
            judgment = "薄い";
          } else {
            judgment = "正常";
          }
          document.getElementById("judgment").textContent = `判定: ${judgment}`;
        }, 1000);
      });
    }
  </script>
</body>
</html>
