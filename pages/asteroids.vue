<template>
  <div class="flex flex-col items-center justify-center gap-16">
    <span
      class="text-center font-poppins text-[2.125rem] font-bold leading-normal text-white"
    >
      ASTEROIDS
    </span>
    <div class="flex justify-evenly gap-16">
      <span
        ref="scoreDisplay"
        class="text-center font-poppins text-[2.125rem] font-bold leading-normal text-white"
      >
        Puntaje: {{ animatedScore }}
      </span>
      <span
        class="text-center font-poppins text-[2.125rem] font-bold leading-normal text-white"
      >
        Vidas: {{ lifes }}
      </span>
      <span v-if="gameOver" class="font-popins text-[2rem] text-red-500">
        Game Over please restart
      </span>
    </div>
    <P5Wrapper :sketch="sketch" />
  </div>
</template>

<script setup lang="ts">
import { gsap } from "gsap";
import { TextPlugin } from "gsap/TextPlugin";
gsap.registerPlugin(TextPlugin);

import p5 from "p5";
import { Howl, Howler } from "howler";
let score = ref(0);
let lifes = ref(3);
let gameOver = ref(false);

const animatedScore = ref(score.value);
const scoreDisplay = ref(null);

watch(
  () => score.value,
  (newScore) => {
    gsap.to(animatedScore, {
      duration: 0.5,
      value: newScore,
      ease: "power1.out",
      onUpdate: () => {
        animatedScore.value = Math.round(animatedScore.value);
      },
    });
  },
);

let laserSound: Howl;
const playShooting = () => {
  if (!laserSound) {
    laserSound = new Howl({
      src: ["/audio/laser.wav"],
      autoplay: false,
      preload: true,
    });
  }
  Howler.ctx
    .resume()
    .then(() => {
      laserSound.play();
      laserSound.volume(0.1);
    })
    .catch((err) => {
      console.error("Error resuming AudioContext:", err);
    });
};

let collisionSound: Howl;
const playCollision = () => {
  if (!collisionSound) {
    collisionSound = new Howl({
      src: ["/audio/bang_sm.wav"],
      autoplay: false,
      preload: true,
    });
  }
  Howler.ctx
    .resume()
    .then(() => {
      collisionSound.volume(0.1);
      collisionSound.play();
    })
    .catch((err) => {
      console.error("Error resuming AudioContext:", err);
    });
};

const sketch = (p5: p5) => {
  class Ship {
    pos: p5.Vector;
    r: number;
    heading: number;
    rotation: number;
    vel: p5.Vector;
    acc: p5.Vector;
    isBoosting: boolean;

    constructor() {
      this.pos = p5.createVector(p5.width / 2, p5.height / 2);
      this.r = 20;
      this.heading = 0;
      this.rotation = 0;
      this.vel = p5.createVector(0, 0);
      this.acc = p5.createVector(0, 0);
      this.isBoosting = false;
    }

    show() {
      p5.push();
      p5.translate(this.pos.x, this.pos.y);
      this.shape(0, -this.r, -this.r * 0.65, +this.r, +this.r * 0.65, +this.r);
      p5.pop();
    }

    update() {
      if (this.isBoosting) {
        this.boost();
      }
      this.turn();
      this.pos.add(this.vel);
      this.vel.mult(0.97);
      this.edges();
    }

    boost() {
      const forceX = Math.cos(this.heading) * 0.5;
      const forceY = Math.sin(this.heading) * 0.5;
      this.vel.x += forceX;
      this.vel.y += forceY;
    }

    boosting(value: boolean) {
      this.isBoosting = value;
    }

    turn() {
      this.heading += this.rotation;
    }

    setRotation(angle: number) {
      this.rotation = angle;
    }

    hits(asteroid: Asteroid) {
      const d = p5.dist(this.pos.x, this.pos.y, asteroid.pos.x, asteroid.pos.y);
      if (d < this.r + asteroid.r) {
        playCollision();
        return true;
      } else {
        return false;
      }
    }

    edges() {
      if (this.pos.x > p5.width + this.r) {
        this.pos.x = -this.r;
      } else if (this.pos.x < -this.r) {
        this.pos.x = p5.width + this.r;
      }

      if (this.pos.y > p5.height + this.r) {
        this.pos.y = -this.r;
      } else if (this.pos.y < -this.r) {
        this.pos.y = p5.height + this.r;
      }
    }

    shape(
      x1: number,
      y1: number,
      x2: number,
      y2: number,
      x3: number,
      y3: number,
    ) {
      p5.rotate(this.heading + p5.PI / 2);

      // Debug: Draw the circle based on the ship's radius
      p5.stroke(0, 255, 0); // Use a green stroke for visibility
      p5.noFill(); // Ensure the circle is not filled
      p5.ellipse(0, 0, this.r * 2); // Draw the circle with diameter `2 * this.r`

      let arcCenter = p5.createVector((x2 + x3) / 2, (y2 + y3) / 2);
      p5.stroke(255);
      p5.fill(255);
      p5.noFill();
      p5.line(x1, y1, x2, y2);
      p5.line(x1, y1, x3, y3);
      p5.arc(arcCenter.x, arcCenter.y, x3 - x2, this.r * 0.65, p5.PI, 0);

      if (this.isBoosting) {
        p5.fill(255, 255, 255);
        p5.noStroke();

        let flameLength =
          this.r * 0.75 + p5.random(-this.r * 0.1, this.r * 0.1);
        let flameSize = this.r * 0.5 + p5.random(-this.r * 0.1, this.r * 0.1);

        p5.triangle(
          arcCenter.x,
          arcCenter.y, // Base of the flame (arc center)
          arcCenter.x - flameSize / 2,
          arcCenter.y + flameLength, // Left corner
          arcCenter.x + flameSize / 2,
          arcCenter.y + flameLength, // Right corner
        );
      }
    }
  }

  class Laser {
    pos: p5.Vector;
    vel: p5.Vector;

    constructor(spos: p5.Vector, angle: number) {
      this.pos = p5.createVector(spos.x, spos.y);
      this.vel = p5.createVector(Math.cos(angle), Math.sin(angle));
      this.vel.mult(10);
    }

    update() {
      this.pos.add(this.vel);
    }

    render() {
      p5.push();
      p5.stroke(255);
      p5.strokeWeight(4);
      p5.point(this.pos.x, this.pos.y);
      p5.pop();
    }

    hits(asteroid: Asteroid | Alien | Ship) {
      let d = p5.dist(this.pos.x, this.pos.y, asteroid.pos.x, asteroid.pos.y);
      if (d < asteroid.r) {
        playCollision();
        return true;
      } else {
        return false;
      }
    }

    offscreen() {
      if (this.pos.x > p5.width || this.pos.x < 0) {
        return true;
      }
      if (this.pos.y > p5.height || this.pos.y < 0) {
        return true;
      }
      return false;
    }
  }

  class Asteroid {
    pos: any;
    v: { x: number; y: number };
    r: any;
    totalVertices: number;
    offset: number[];
    constructor(pos?: any, r?: any) {
      if (pos) {
        this.pos = pos.copy();
      } else {
        this.pos = p5.createVector(p5.random(p5.width), p5.random(p5.height));
      }
      const angle = Math.random() * Math.PI * 2; // Random angle in radians
      this.v = {
        x: Math.cos(angle),
        y: Math.sin(angle),
      };
      this.r = r || p5.random(30, 50);
      this.totalVertices = p5.floor(p5.random(5, 15));
      this.offset = [];
      for (let i = 0; i < this.totalVertices; i++) {
        this.offset[i] = p5.random(-this.r * 0.5, this.r * 0.5);
      }
    }
    update() {
      this.pos.x += this.v.x;
      this.pos.y += this.v.y;
    }
    show() {
      p5.push();
      p5.stroke(255);
      p5.noFill();
      p5.translate(this.pos.x, this.pos.y);
      p5.beginShape();
      for (let i = 0; i < this.totalVertices; i++) {
        let angle = p5.map(i, 0, this.totalVertices, 0, p5.TWO_PI);
        let r = this.r + this.offset[i];
        let x = r * p5.cos(angle);
        let y = r * p5.sin(angle);
        p5.vertex(x, y);
      }
      p5.endShape(p5.CLOSE);

      p5.stroke(255, 0, 0); // Use a red stroke for visibility
      p5.noFill(); // Ensure the circle is not filled
      p5.ellipse(0, 0, this.r * 2); // Draw the circle with diameter `2 * this.r`

      p5.pop();
    }
    breakup() {
      let newA = [];
      newA[0] = new Asteroid(this.pos, this.r / 2);
      newA[1] = new Asteroid(this.pos, this.r / 2);
      return newA;
    }
    edges() {
      if (this.pos.x > p5.width + this.r) {
        this.pos.x = -this.r;
      } else if (this.pos.x < -this.r) {
        this.pos.x = p5.width + this.r;
      }

      if (this.pos.y > p5.height + this.r) {
        this.pos.y = -this.r;
      } else if (this.pos.y < -this.r) {
        this.pos.y = p5.height + this.r;
      }
    }
  }

  class Alien {
    r: any;
    defineStart: number;
    timeToSpawn: number;
    isFirstTimeInScreen: boolean;
    isActive: boolean;
    timeToChangeDirection: number;
    pos: any;
    vel: any;
    heading: number;
    velMultiplier: number;

    constructor() {
      this.r = this.determineSize();
      this.defineStart = Math.floor(Math.random() * 4);
      this.timeToSpawn = Math.floor(Math.random() * 300) + 500;
      this.isFirstTimeInScreen = true;
      this.isActive = false;
      this.timeToChangeDirection = 100;
      this.pos = this.determineStart();
      this.vel = p5.createVector(0, 0);
      this.heading = 0;
      this.velMultiplier = 4;
    }

    restartValues() {
      this.r = this.determineSize();
      this.defineStart = Math.floor(Math.random() * 4);
      this.timeToSpawn = Math.floor(Math.random() * 300) + 500;
      this.isFirstTimeInScreen = true;
      this.isActive = false;
      this.timeToChangeDirection = 100;
      this.pos = this.determineStart();
      this.vel = p5.createVector(0, 0);
      this.heading = 0;
      this.velMultiplier = 4;
    }

    show() {
      if (this.isActive) {
        p5.push();
        p5.translate(this.pos.x, this.pos.y);
        this.shape(
          -this.r,
          0, //x1
          -this.r * 0.33,
          this.r * 0.33, //x2
          this.r * 0.33,
          this.r * 0.33, //x3
          this.r,
          0, //x4
          this.r * 0.33,
          -this.r * 0.33, //x5
          -this.r * 0.33,
          -this.r * 0.33, //x6
          -this.r * 0.166,
          -this.r * 0.66, //x7
          this.r * 0.166,
          -this.r * 0.66,
        );
        p5.pop();
      }
    }

    update() {
      if (!this.isActive) {
        this.timeToSpawn -= 1;
        if (this.timeToSpawn <= 0) {
          this.isActive = true;
        }
      } else {
        this.moveAlien(this.isFirstTimeInScreen);
      }

      this.pos.add(this.vel);
      this.edges();
    }

    moveAlien(flag: boolean) {
      if (flag) {
        if (this.defineStart == 0 || this.defineStart == 1) {
          this.vel.x = 1;
          this.heading = 0;
        } else if (this.defineStart == 2 || this.defineStart == 3) {
          this.vel.x = -1;
          this.heading = p5.PI;
        }
        this.pos.add(this.vel);

        this.timeToChangeDirection -= 1;

        if (this.timeToChangeDirection <= 0) {
          this.isFirstTimeInScreen = false;
        }
      } else {
        this.setPath();
      }
    }

    setPath() {
      let direction = 1;
      if (this.timeToChangeDirection <= 0) {
        if (this.defineStart == 0 || this.defineStart == 1) {
          direction = Math.random() * ((5 * p5.PI) / 7) - (5 * p5.PI) / 14;
        } else if (this.defineStart == 2 || this.defineStart == 3) {
          direction =
            Math.random() * ((5 * p5.PI) / 7) - (5 * p5.PI) / 14 + p5.PI;
        }
        this.heading = direction;

        const forceX = Math.cos(this.heading) * 0.5;
        const forceY = Math.sin(this.heading) * 0.5;
        this.vel.x += forceX * this.velMultiplier;
        this.vel.y += forceY * this.velMultiplier;
        this.timeToChangeDirection = 100;
      }
      this.timeToChangeDirection -= 1;
    }

    hits(asteroid: Asteroid) {
      if (this.isActive == true) {
        const d = p5.dist(
          this.pos.x,
          this.pos.y,
          asteroid.pos.x,
          asteroid.pos.y,
        );
        if (d < this.r + asteroid.r) {
          playCollision();
          return true;
        } else {
          return false;
        }
      }
    }

    determineSize() {
      const x = Math.floor(Math.random() * 5);
      if (x == 0) {
        return 20;
      } else {
        return 40;
      }
    }

    determineStart() {
      let y;
      if (this.defineStart == 0) {
        y = p5.createVector(-this.r, this.r);
        return y;
      } else if (this.defineStart == 1) {
        y = p5.createVector(-this.r, p5.height - this.r);
        return y;
      } else if (this.defineStart == 2) {
        y = p5.createVector(p5.width + this.r, this.r);
        return y;
      } else if (this.defineStart == 3) {
        y = p5.createVector(p5.width + this.r, p5.height - this.r);
        return y;
      }
    }

    edges() {
      if (this.pos.x > p5.width + this.r) {
        this.restartValues();
      } else if (this.pos.x < -this.r) {
        this.restartValues();
      }
      if (this.pos.y > p5.height + this.r) {
        this.pos.y = -this.r;
      } else if (this.pos.y < -this.r) {
        this.pos.y = p5.height + this.r;
      }
    }

    shape(
      x1: number,
      y1: number,
      x2: number,
      y2: number,
      x3: number,
      y3: number,
      x4: number,
      y4: number,
      x5: number,
      y5: number,
      x6: number,
      y6: number,
      x7: number,
      y7: number,
      x8: number,
      y8: number,
    ) {
      p5.stroke(255);
      p5.fill(255);
      p5.line(x1, y1, x2, y2);
      p5.line(x2, y2, x3, y3);
      p5.line(x3, y3, x4, y4);
      p5.line(x1, y1, x4, y4);
      p5.line(x4, y4, x5, y5);
      p5.line(x5, y5, x6, y6);
      p5.line(x6, y6, x1, y1);
      p5.line(x6, y6, x7, y7);
      p5.line(x7, y7, x8, y8);
      p5.line(x8, y8, x5, y5);
    }
  }

  let ship: Ship;
  let alien: Alien;
  let lasers: Laser[] = [];
  let alienLasers: Laser[] = [];
  let gameFont: p5.Font;
  let gameState = "MENU";
  let asteroids: Asteroid[] = [];
  let alienLaserCooldown: number = 0;

  p5.preload = () => {
    gameFont = p5.loadFont("Fonts/PressStart2P-Regular.ttf");
  };

  p5.setup = () => {
    p5.createCanvas(960, 720);
    p5.frameRate(60);

    ship = new Ship();
    alien = new Alien();
    alienLaserCooldown = 60;
    for (let i = 0; i < 5; i++) {
      asteroids.push(new Asteroid());
    }
  };

  p5.draw = () => {
    p5.background(0);

    if (gameState === "MENU") {
      drawMenu();
    } else if (gameState === "PLAYING") {
      playGame();
    } else if (gameState === "GAMEOVER") {
      drawGameOver();
    }
  };

  let drawMenu = () => {
    for (let i = asteroids.length - 1; i >= 0; i--) {
      asteroids[i].update();
      asteroids[i].show();
      asteroids[i].edges();
    }
    p5.fill(255);
    p5.textFont(gameFont);
    p5.textAlign(p5.CENTER);
    p5.textSize(32);
    p5.text("ASTEROIDS", p5.width / 2, p5.height / 2 - 40);
    p5.textSize(16);
    p5.text("Press      to start", p5.width / 2, p5.height / 2 + 20);
    if (p5.frameCount % 30 < 15) {
      p5.text("ENTER ", p5.width / 2 - 15, p5.height / 2 + 20);
    }
  };

  let playGame = () => {
    ship.show();
    ship.update();
    alien.show();
    alien.update();

    for (let i = lasers.length - 1; i >= 0; i--) {
      lasers[i].render();
      lasers[i].update();
      if (lasers[i] && lasers[i].offscreen()) {
        lasers.splice(i, 1);
      } else if (alien.isActive && lasers[i].hits(alien)) {
        // hitSound.play();
        alien.restartValues();
        lasers.splice(i, 1);
      } else {
        for (let j = asteroids.length - 1; j >= 0; j--) {
          if (lasers[i].hits(asteroids[j])) {
            score.value += 10;
            if (asteroids[j].r > 20) {
              let newAsteroids = asteroids[j].breakup();
              asteroids = asteroids.concat(newAsteroids);
            }
            asteroids.splice(j, 1);
            lasers.splice(i, 1);
            break;
          }
        }
      }
    }

    for (let i = asteroids.length - 1; i >= 0; i--) {
      if (ship.hits(asteroids[i])) {
        lifes.value -= 1;
        if (lifes.value === 0) {
          gameOver.value = true;
          gameState = "GAMEOVER";
        } else {
          ship = new Ship();
        }
      }

      if (alien.hits(asteroids[i])) {
        alien.restartValues();
      }

      asteroids[i].update();
      asteroids[i].show();
      asteroids[i].edges();
    }

    if (alien.isActive) {
      if (alienLaserCooldown <= 0) {
        let direction = p5.createVector(
          ship.pos.x - alien.pos.x,
          ship.pos.y - alien.pos.y,
        );
        let angle = Math.atan(direction.y / direction.x);
        if (ship.pos.x <= alien.pos.x) {
          if (ship.pos.y > alien.pos.y) {
            angle += p5.PI;
          } else {
            angle -= p5.PI;
          }
        }
        alienLasers.push(new Laser(alien.pos, angle));
        playShooting();
        alienLaserCooldown = 60;
      } else {
        alienLaserCooldown -= 1;
      }
    }

    for (let i = alienLasers.length - 1; i >= 0; i--) {
      alienLasers[i].render();
      alienLasers[i].update();
      if (alienLasers[i].hits(ship)) {
        lifes.value -= 1;
        if (lifes.value === 0) {
          gameOver.value = true;
          gameState = "GAMEOVER";
        } else {
          ship = new Ship();
        }
        alienLasers.splice(i, 1);
        continue;
      }
      if (alienLasers[i] && alienLasers[i].offscreen()) {
        alienLasers.splice(i, 1);
      } else {
        for (let j = asteroids.length - 1; j >= 0; j--) {
          if (alienLasers[i].hits(asteroids[j])) {
            if (asteroids[j].r > 20) {
              let newAsteroids = asteroids[j].breakup();
              asteroids = asteroids.concat(newAsteroids);
            }
            asteroids.splice(j, 1);
            alienLasers.splice(i, 1);
            break;
          }
        }
      }
    }
    drawScore();
  };

  let drawGameOver = () => {
    p5.fill(255);
    p5.textFont(gameFont);
    p5.textAlign(p5.CENTER);
    p5.textSize(32);
    p5.text("GAME OVER", p5.width / 2, p5.height / 2 - 40);
    p5.textSize(16);
    p5.text(`Final Score: ${score.value}`, p5.width / 2, p5.height / 2);
    p5.text("Press ENTER to Return to Menu", p5.width / 2, p5.height / 2 + 40);
  };

  let drawScore = () => {
    p5.fill(255);
    p5.textFont(gameFont);
    p5.textAlign(p5.LEFT);
    p5.textSize(16);
    p5.text(`Score: ${score.value}`, 10, 20);
  };

  let startGame = () => {
    gameState = "PLAYING";
    ship = new Ship();
    asteroids = [];
    for (let i = 0; i < 5; i++) {
      asteroids.push(new Asteroid());
    }
    alien = new Alien();
    lasers = [];
    alienLasers = [];
    score.value = 0;
    lifes.value = 3;
  };

  let resetGame = () => {
    gameState = "MENU";
  };

  p5.keyReleased = () => {
    ship.setRotation(0);
    ship.boosting(false);
  };

  p5.keyPressed = () => {
    if (gameState === "MENU" && p5.keyCode === p5.ENTER) {
      startGame();
    } else if (gameState === "GAMEOVER" && p5.keyCode === p5.ENTER) {
      resetGame();
    }

    if (p5.key === " ") {
      playShooting();
      lasers.push(new Laser(ship.pos, ship.heading));
    }

    if (p5.keyCode === p5.RIGHT_ARROW && !p5.keyIsDown(p5.LEFT_ARROW)) {
      ship.setRotation(p5.PI / 36);
    }
    if (p5.keyCode === p5.LEFT_ARROW && !p5.keyIsDown(p5.RIGHT_ARROW)) {
      ship.setRotation(-p5.PI / 36);
    }
    if (p5.keyCode === p5.UP_ARROW) {
      ship.boosting(true);
    }

    if (p5.keyCode === p5.ESCAPE) {
      p5.noLoop();
    }
    if (!p5.isLooping() && p5.keyCode === p5.ESCAPE) {
      p5.loop();
    }
  };
};
</script>

<style></style>
