<template>
  <div class="flex flex-col items-center justify-center gap-16">
    <span
      class="font-poppins text-center text-[2.125rem] font-bold leading-normal text-white"
    >
      ASTEROIDS
    </span>
    <P5Wrapper :sketch="sketch" />
  </div>
</template>

<script lang="ts" setup>
import p5 from "p5";
import { Howl, Howler } from "howler";

let sound = null;
const playAudio = () => {
  if (!sound) {
    sound = new Howl({
      src: ["/audio/laser.wav"],
      autoplay: false,
      preload: true,
    });
  }
  Howler.ctx.resume().then(() => {
    sound.play();
  }).catch((err) => {
    console.error("Error resuming AudioContext:", err);
  });
};

const sketch = (p5: p5) => {
  class Ship {
    constructor() {
      this.pos = p5.createVector(p5.windowWidth / 2, p5.windowHeight / 2);
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
      this.shape(
        0,
        -this.r,
        -this.r * 0.65,
        +this.r,
        +this.r * 0.65,
        +this.r,
        this.r * 1.9,
        this.r * 0.75,
      );
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

    boosting(value) {
      this.isBoosting = value;
    }

    turn() {
      this.heading += this.rotation;
    }

    setRotation(angle) {
      this.rotation = angle;
    }

    hits(asteroid) {
      const d = p5.dist(this.pos.x, this.pos.y, asteroid.pos.x, asteroid.pos.y);
      if (d < this.r + asteroid.r) {
        // collisionSound.play();
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

    shape(x1, y1, x2, y2, x3, y3) {
      p5.rotate(this.heading + p5.PI / 2);
      let arcCenter = p5.createVector((x2 + x3) / 2, (y2 + y3) / 2);
      p5.stroke(255);
      p5.fill(255);
      p5.noFill();
      p5.line(x1, y1, x2, y2);
      p5.line(x1, y1, x3, y3);
      p5.arc(arcCenter.x, arcCenter.y, x3 - x2, this.r * 0.65, p5.PI, 0);
    }
  }

  class Laser {
    constructor(spos, angle) {
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

    hits(asteroid) {
      let d = p5.dist(this.pos.x, this.pos.y, asteroid.pos.x, asteroid.pos.y);
      if (d < asteroid.r) {
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
    constructor(pos, r) {
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

  let ship;
  let lasers = [];
  let asteroids = [];

  p5.preload = () => {};

  p5.setup = () => {
    p5.createCanvas(1020, 960);
    ship = new Ship();
    for (let i = 0; i < 1; i++) {
      asteroids.push(new Asteroid());
    }
  };
  p5.draw = () => {
    p5.background(0);
    ship.show();
    ship.update();

    for (let i = lasers.length - 1; i >= 0; i--) {
      lasers[i].render();
      lasers[i].update();
      if (lasers[i] && lasers[i].offscreen()) {
        lasers.splice(i, 1);
      } else {
        for (let j = asteroids.length - 1; j >= 0; j--) {
          if (lasers[i].hits(asteroids[j])) {
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
        console.log("opps");
        p5.noLoop();
      }
      asteroids[i].update();
      asteroids[i].show();
      asteroids[i].edges();
    }
  };

  p5.keyReleased = () => {
    ship.setRotation(0);
    ship.boosting(false);
  };

  p5.keyPressed = () => {
    if (p5.key === " ") {
      playAudio();
      lasers.push(new Laser(ship.pos, ship.heading));
    }
    if (p5.keyCode === p5.RIGHT_ARROW) {
      ship.setRotation(0.1);
    } else if (p5.keyCode === p5.LEFT_ARROW) {
      ship.setRotation(-0.1);
    } else if (p5.keyCode === p5.UP_ARROW) {
      ship.boosting(true);
    }
  };
};
</script>

<style></style>
