
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>六一儿童节快乐！</title>
  <style>
    body {
      margin: 0;
      background: linear-gradient(to top, #0d1b2a, #1b263b);
      color: #fff;
      font-family: '微软雅黑', sans-serif;
      overflow: hidden;
      position: relative;
      height: 100vh;
    }
    header {
      text-align: center;
      padding: 30px 20px 10px;
      font-size: 2.5rem;
      font-weight: bold;
      text-shadow: 0 0 8px #f39c12;
    }
    .subtitle {
      text-align: center;
      font-size: 1.2rem;
      margin-bottom: 20px;
      color: #f1c40f;
    }
    .image-container {
      display: flex;
      justify-content: center;
      margin-bottom: 40px;
      cursor: pointer;
    }
    .image-container img {
      max-width: 90%;
      max-height: 400px;
      border-radius: 15px;
      box-shadow: 0 0 20px #f39c12;
      user-select: none;
      transition: transform 0.3s ease;
    }
    .image-container img:hover {
      transform: scale(1.05);
    }
    canvas {
      position: fixed;
      top: 0;
      left: 0;
      pointer-events: none;
      width: 100%;
      height: 100%;
      z-index: 9999;
    }
  </style>
</head>
<body>
  <header>六一儿童节快乐！</header>
  <div class="subtitle">愿每个孩子都拥有一个充满欢乐与梦想的童年</div>
  <div class="image-container" id="imageContainer" title="点击切换图片">
    <img id="mainImage" src="https://images.unsplash.com/photo-1506744038136-46273834b3fb?auto=format&fit=crop&w=800&q=80" alt="儿童在玩耍" />
  </div>

  <canvas id="fireworks"></canvas>

  <script>
    const images = [
      "https://images.unsplash.com/photo-1506744038136-46273834b3fb?auto=format&fit=crop&w=800&q=80",
      "https://images.unsplash.com/photo-1517423440428-a5a00ad493e8?auto=format&fit=crop&w=800&q=80",
      "https://images.unsplash.com/photo-1549924231-f129b911e442?auto=format&fit=crop&w=800&q=80",
      "https://images.unsplash.com/photo-1496307042754-b4aa456c4a2d?auto=format&fit=crop&w=800&q=80",
      "https://images.unsplash.com/photo-1488469020697-9531d961b348?auto=format&fit=crop&w=800&q=80"
    ];

    let currentImageIndex = 0;
    const mainImage = document.getElementById('mainImage');
    const imageContainer = document.getElementById('imageContainer');

    imageContainer.addEventListener('click', () => {
      currentImageIndex = (currentImageIndex + 1) % images.length;
      mainImage.src = images[currentImageIndex];
    });

    // 以下是烟花部分，不变
    const canvas = document.getElementById('fireworks');
    const ctx = canvas.getContext('2d');

    let cw, ch;
    let fireworks = [];
    let particles = [];

    function resize() {
      cw = window.innerWidth;
      ch = window.innerHeight;
      canvas.width = cw;
      canvas.height = ch;
    }
    window.addEventListener('resize', resize);
    resize();

    class Firework {
      constructor(sx, sy, tx, ty) {
        this.x = sx;
        this.y = sy;
        this.sx = sx;
        this.sy = sy;
        this.tx = tx;
        this.ty = ty;
        this.distanceToTarget = distance(sx, sy, tx, ty);
        this.distanceTraveled = 0;
        this.coordinates = [];
        this.coordinateCount = 3;
        while(this.coordinateCount--) {
          this.coordinates.push([this.x, this.y]);
        }
        this.angle = Math.atan2(ty - sy, tx - sx);
        this.speed = 5;
        this.acceleration = 1.05;
        this.brightness = random(50, 70);
      }
      update(index) {
        this.coordinates.pop();
        this.coordinates.unshift([this.x, this.y]);

        this.speed *= this.acceleration;
        let vx = Math.cos(this.angle) * this.speed;
        let vy = Math.sin(this.angle) * this.speed;
        this.distanceTraveled = distance(this.sx, this.sy, this.x + vx, this.y + vy);

        if(this.distanceTraveled >= this.distanceToTarget) {
          createParticles(this.tx, this.ty);
          fireworks.splice(index, 1);
        } else {
          this.x += vx;
          this.y += vy;
        }
      }
      draw() {
        ctx.beginPath();
        ctx.moveTo(this.coordinates[this.coordinates.length - 1][0], this.coordinates[this.coordinates.length - 1][1]);
        ctx.lineTo(this.x, this.y);
        ctx.strokeStyle = `hsl(${this.brightness}, 100%, 50%)`;
        ctx.stroke();
      }
    }

    class Particle {
      constructor(x, y) {
        this.x = x;
        this.y = y;
        this.coordinates = [];
        this.coordinateCount = 5;
        while(this.coordinateCount--) {
          this.coordinates.push([this.x, this.y]);
        }
        this.angle = random(0, Math.PI * 2);
        this.speed = random(1, 10);
        this.friction = 0.95;
        this.gravity = 0.7;
        this.hue = random(0, 360);
        this.brightness = random(50, 80);
        this.alpha = 1;
        this.decay = random(0.015, 0.03);
      }
      update(index) {
        this.coordinates.pop();
        this.coordinates.unshift([this.x, this.y]);
        this.speed *= this.friction;
        this.x += Math.cos(this.angle) * this.speed;
        this.y += Math.sin(this.angle) * this.speed + this.gravity;
        this.alpha -= this.decay;

        if(this.alpha <= 0) {
          particles.splice(index, 1);
        }
      }
      draw() {
        ctx.beginPath();
        ctx.moveTo(this.coordinates[this.coordinates.length - 1][0], this.coordinates[this.coordinates.length - 1][1]);
        ctx.lineTo(this.x, this.y);
        ctx.strokeStyle = `hsla(${this.hue}, 100%, ${this.brightness}%, ${this.alpha})`;
        ctx.stroke();
      }
    }

    function createParticles(x, y) {
      let particleCount = 30;
      while(particleCount--) {
        particles.push(new Particle(x, y));
      }
    }

    function distance(aX, aY, bX, bY) {
      let dx = bX - aX;
      let dy = bY - aY;
      return Math.sqrt(dx * dx + dy * dy);
    }

    function random(min, max) {
      return Math.random() * (max - min) + min;
    }

    function loop() {
      requestAnimationFrame(loop);
      ctx.globalCompositeOperation = 'destination-out';
      ctx.fillStyle = 'rgba(0, 0, 0, 0.3)';
      ctx.fillRect(0, 0, cw, ch);
      ctx.globalCompositeOperation = 'lighter';

      for(let i = fireworks.length - 1; i >= 0; i--) {
        fireworks[i].draw();
        fireworks[i].update(i);
      }

      for(let i = particles.length - 1; i >= 0; i--) {
        particles[i].draw();
        particles[i].update(i);
      }
    }

    setInterval(() => {
      let sx = cw / 2;
      let sy = ch;
      let tx = random(50, cw - 50);
      let ty = random(50, ch / 2);
      fireworks.push(new Firework(sx, sy, tx, ty));
    }, 800);

    window.onload = loop;
  </script>
</body>
</html>
