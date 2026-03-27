[heart.html](https://github.com/user-attachments/files/26309085/heart.html)
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta
    name="viewport"
    content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no"
  />
  <title>для тебя</title>

  <style>
    html, body {
      margin: 0;
      padding: 0;
      width: 100%;
      height: 100%;
      overflow: hidden;
      touch-action: none;
      background: radial-gradient(circle at center, #1a0010 0%, #09000a 45%, #000 100%);
      -webkit-tap-highlight-color: transparent;
    }

    body {
      position: fixed;
      inset: 0;
    }

    canvas {
      display: block;
      width: 100vw;
      height: 100vh;
      height: 100dvh;
    }
  </style>
</head>
<body>
  <canvas id="heart"></canvas>

  <script>
    const canvas = document.getElementById("heart");
    const ctx = canvas.getContext("2d");

    const isMobile = /android|webos|iphone|ipad|ipod|blackberry|iemobile|opera mini/i
      .test((navigator.userAgent || navigator.vendor || window.opera).toLowerCase());

    let width = 0;
    let height = 0;
    let dpr = 1;

    let points = [];
    let drawProgress = 0;
    let glowProgress = 0;
    let pulseTime = 0;
    let particles = [];
    let snowflakes = [];

    function getViewportSize() {
      return {
        width: window.innerWidth,
        height: window.innerHeight
      };
    }

    function resizeCanvas() {
      const vp = getViewportSize();
      width = vp.width;
      height = vp.height;

      dpr = Math.min(window.devicePixelRatio || 1, isMobile ? 2 : 2);

      canvas.width = Math.floor(width * dpr);
      canvas.height = Math.floor(height * dpr);

      ctx.setTransform(dpr, 0, 0, dpr, 0, 0);

      buildHeart();
      createParticles();
      createSnow();

      drawProgress = 0;
      glowProgress = 0;
    }

    function heartPosition(t) {
      return {
        x: 16 * Math.pow(Math.sin(t), 3),
        y: -(13 * Math.cos(t) - 5 * Math.cos(2 * t) - 2 * Math.cos(3 * t) - Math.cos(4 * t))
      };
    }

    function buildHeart() {
      points = [];

      const totalPoints = isMobile ? 420 : 700;
      const scale = Math.min(width, height) * (isMobile ? 0.022 : 0.018);

      for (let i = 0; i <= totalPoints; i++) {
        const t = (Math.PI * 2 * i) / totalPoints;
        const p = heartPosition(t);

        points.push({
          x: width / 2 + p.x * scale,
          y: height / 2 + p.y * scale
        });
      }
    }

    function createParticles() {
      particles = [];
      const count = isMobile ? 35 : 80;

      for (let i = 0; i < count; i++) {
        particles.push({
          angle: Math.random() * Math.PI * 2,
          offset: Math.random(),
          size: Math.random() * (isMobile ? 1.5 : 2) + 0.8,
          speed: Math.random() * 0.015 + 0.005,
          alpha: Math.random() * 0.35 + 0.12
        });
      }
    }

    function createSnow() {
      snowflakes = [];
      const count = isMobile
        ? Math.max(22, Math.floor(width / 22))
        : Math.max(40, Math.floor(width / 30));

      for (let i = 0; i < count; i++) {
        snowflakes.push({
          x: Math.random() * width,
          y: Math.random() * height,
          r: Math.random() * (isMobile ? 1.4 : 2) + 0.4,
          speedY: Math.random() * 0.5 + 0.15,
          speedX: Math.random() * 0.3 - 0.15,
          alpha: Math.random() * 0.05 + 0.01,
          sway: Math.random() * Math.PI * 2
        });
      }
    }

    function updateAndDrawSnow() {
      ctx.save();

      for (const s of snowflakes) {
        s.y += s.speedY;
        s.sway += 0.01;
        s.x += s.speedX + Math.sin(s.sway) * 0.12;

        if (s.y > height + 8) {
          s.y = -8;
          s.x = Math.random() * width;
        }

        if (s.x < -8) s.x = width + 8;
        if (s.x > width + 8) s.x = -8;

        ctx.beginPath();
        ctx.arc(s.x, s.y, s.r, 0, Math.PI * 2);
        ctx.fillStyle = `rgba(255,255,255,${s.alpha})`;
        ctx.shadowColor = `rgba(255,255,255,${s.alpha * 0.5})`;
        ctx.shadowBlur = 3;
        ctx.fill();
      }

      ctx.restore();
    }

    function drawBackground() {
      ctx.clearRect(0, 0, width, height);

      const bg = ctx.createRadialGradient(
        width / 2, height / 2, 50,
        width / 2, height / 2, Math.max(width, height) * 0.7
      );
      bg.addColorStop(0, "rgba(50, 0, 30, 0.25)");
      bg.addColorStop(0.4, "rgba(15, 0, 20, 0.18)");
      bg.addColorStop(1, "rgba(0, 0, 0, 0.35)");

      ctx.fillStyle = bg;
      ctx.fillRect(0, 0, width, height);
    }

    function drawText() {
      ctx.save();
      ctx.textAlign = "center";
      ctx.textBaseline = "middle";

      const fontSize = isMobile ? Math.max(28, width * 0.09) : 56;
      ctx.font = `bold ${fontSize}px 'Brush Script MT', 'Segoe Script', cursive`;
      ctx.fillStyle = "rgba(255, 255, 255, 0.035)";
      ctx.shadowColor = "rgba(255, 120, 180, 0.16)";
      ctx.shadowBlur = isMobile ? 8 : 12;

      ctx.fillText("для тебя", width / 2, height / 2 + (isMobile ? 8 : 10));
      ctx.restore();
    }

    function drawHeartPath(count, scalePulse = 1) {
      if (count < 2) return;

      ctx.save();
      ctx.translate(width / 2, height / 2);
      ctx.scale(scalePulse, scalePulse);
      ctx.translate(-width / 2, -height / 2);

      ctx.beginPath();
      ctx.moveTo(points[0].x, points[0].y);

      for (let i = 1; i < count; i++) {
        ctx.lineTo(points[i].x, points[i].y);
      }

      ctx.strokeStyle = "rgba(255, 120, 160, 1)";
      ctx.lineWidth = isMobile ? 2.2 : 3;
      ctx.shadowColor = "rgba(255, 80, 140, 0.95)";
      ctx.shadowBlur = isMobile ? 16 : 25;
      ctx.stroke();

      ctx.strokeStyle = "rgba(255, 200, 220, 0.95)";
      ctx.lineWidth = isMobile ? 1 : 1.2;
      ctx.shadowBlur = isMobile ? 5 : 8;
      ctx.stroke();

      ctx.restore();
    }

    function drawHeartGlow(fillAmount, scalePulse = 1) {
      if (fillAmount <= 0) return;

      ctx.save();
      ctx.translate(width / 2, height / 2);
      ctx.scale(scalePulse, scalePulse);
      ctx.translate(-width / 2, -height / 2);

      ctx.beginPath();
      ctx.moveTo(points[0].x, points[0].y);
      for (let i = 1; i < points.length; i++) {
        ctx.lineTo(points[i].x, points[i].y);
      }
      ctx.closePath();

      const g = ctx.createRadialGradient(
        width / 2, height / 2 - 20, 10,
        width / 2, height / 2, Math.min(width, height) * 0.22
      );
      g.addColorStop(0, `rgba(255, 180, 210, ${0.14 * fillAmount})`);
      g.addColorStop(0.5, `rgba(255, 90, 150, ${0.09 * fillAmount})`);
      g.addColorStop(1, `rgba(255, 0, 90, 0)`);

      ctx.fillStyle = g;
      ctx.shadowColor = `rgba(255, 70, 140, ${0.4 * fillAmount})`;
      ctx.shadowBlur = isMobile ? 22 : 35;
      ctx.fill();

      ctx.restore();
    }

    function drawSparkParticles(scalePulse = 1) {
      if (drawProgress < points.length * 0.6) return;

      ctx.save();
      ctx.translate(width / 2, height / 2);
      ctx.scale(scalePulse, scalePulse);
      ctx.translate(-width / 2, -height / 2);

      for (const p of particles) {
        p.angle += p.speed;

        const index = Math.floor(
          ((Math.sin(p.angle + p.offset * 5) + 1) / 2) * (points.length - 1)
        );
        const base = points[index];
        const spread = isMobile
          ? 5 + 7 * Math.sin(p.angle * 2 + p.offset)
          : 8 + 12 * Math.sin(p.angle * 2 + p.offset);

        const x = base.x + Math.cos(p.angle * 3) * spread;
        const y = base.y + Math.sin(p.angle * 2) * spread;

        ctx.beginPath();
        ctx.fillStyle = `rgba(255, 220, 235, ${p.alpha})`;
        ctx.shadowColor = "rgba(255, 150, 200, 0.85)";
        ctx.shadowBlur = isMobile ? 7 : 12;
        ctx.arc(x, y, p.size, 0, Math.PI * 2);
        ctx.fill();
      }

      ctx.restore();
    }

    function drawTrail() {
      ctx.save();
      ctx.fillStyle = "rgba(0, 0, 0, 0.08)";
      ctx.fillRect(0, 0, width, height);
      ctx.restore();
    }

    function animate() {
      drawBackground();
      updateAndDrawSnow();
      drawTrail();

      let pulseScale = 1;

      if (drawProgress < points.length) {
        drawProgress += isMobile ? 3 : 4;
      } else {
        pulseTime += isMobile ? 0.04 : 0.05;
        pulseScale = 1 + Math.sin(pulseTime) * 0.025;

        if (glowProgress < 1) {
          glowProgress += isMobile ? 0.008 : 0.01;
        }
      }

      drawHeartGlow(glowProgress, pulseScale);
      drawHeartPath(Math.min(drawProgress, points.length), pulseScale);
      drawSparkParticles(pulseScale);
      drawText();

      requestAnimationFrame(animate);
    }

    let resizeTimer = null;
    window.addEventListener("resize", () => {
      clearTimeout(resizeTimer);
      resizeTimer = setTimeout(resizeCanvas, 120);
    });

    window.addEventListener("orientationchange", () => {
      setTimeout(resizeCanvas, 200);
    });

    resizeCanvas();
    animate();
  </script>
</body>
</html>
