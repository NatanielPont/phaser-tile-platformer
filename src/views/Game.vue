<template>
    <div id="game"></div>
</template>

<script>
import Phaser from 'phaser'
import wall from '../assets/img/wall.png'
import ground from '../assets/img/ground.png'
import player from '../assets/img/player.png'
import explosion from '../assets/img/effects/explode.png'
import coin from '../assets/img/coin.png'
import splash from '../assets/img/effects/splash.png'
import explosionMobileEnemy from '../assets/img/effects/splash_enemy.png'
import enemy from '../assets/img/enemy.png'
// import mobileEnemy from '../assets/img/enemy.png'
import soundDie from '../assets/audio/dead.mp3'
import soundTakeCoin from '../assets/audio/coin.mp3'

let score = 0
let lives = 3
let livesText = 3
let scoreText
let soundDead
let soundCoin

// TODO -> Detectar quan l'usuari surt del mon -> executar un die --done
function createRandomCoins (value, coins) {
  switch (value) {
    case 1:
      coins.create(140, 200 / 2, 'coin')

      break
    case 2:
      coins.create(170, 200 / 2, 'coin')

      break
    case 3:
      coins.create(200, 200 / 2, 'coin')

      break
    case 4:
      coins.create(290, 200 / 2, 'coin')

      break
    case 5:
      coins.create(320, 200 / 2, 'coin')

      break
    case 6:
      coins.create(350, 200 / 2, 'coin')

      break
  }
}
// TODO: Display amb el número de vidas -> Mostrar el número de vidas i si l'has hem gastat aturar el joc --done

function lesslive (player, enemy) {
  this.explosionMobileEnemy = this.physics.add.sprite(player.body.x + player.body.height / 2, player.body.y + player.body.width / 2, 'explosionMobileEnemy')
  // TODO: Simular la explosió --done

  this.explosionMobileEnemy.anims.play('splash_enemy', true)

  lives = lives - 1
  let livesAux = lives
  let scoreAux = score
  player.disableBody(true, true)

  // TODO -> EXECUTAR SO --done
  soundDead.play()

  // TODO REINICIAR NIVELL --done
  // player.scene.cameras.main.shake(500)
  // score = 0
  // if (lives > 0) {
  if (lives > 0) {
    this.scene.restart()

    lives = livesAux
    score = scoreAux
    // scoreText.setText('Score: ' + score)
  } else {
    score = scoreAux

    this.gameOverText.setVisible(true)
    this.mobileEnemy.disableBody(true, true)
    // scoreText.setText('Score: ' + score)
  }

  //   scoreText.setText('Score: ' + lives)
}

function die (player, enemy) {
  // TODO -> MILLORAR AMB UNA ANIMACIÓ --done
  this.explosion = this.physics.add.sprite(player.body.x + player.body.height / 2, player.body.y + player.body.width / 2, 'explosion')
  // TODO: Simular la explosió --done

  this.explosion.anims.play('explode', false)

  player.disableBody(true, true)

  // TODO -> EXECUTAR SO --done
  soundDead.play()

  // TODO REINICIAR NIVELL --done
  player.scene.cameras.main.shake(500)
  score = 0

  this.scene.restart()
}

// SHAKE EFFECT
// http://labs.phaser.io/edit.html?src=src/camera/shake.js

function takeCoin (player, coin) {
  // see random value doesn't have the same value of oldValue random
  let oldValue = this.value
  this.value = Phaser.Math.Between(1, 6)

  while (oldValue === this.value) {
    this.value = Phaser.Math.Between(1, 6)
  }

  createRandomCoins(this.value, this.coins)
  // TODO -> MILLORAR AMB UNA ANIMACIÓ--done
  this.splash = this.physics.add.sprite(coin.body.x + coin.body.height / 2, coin.body.y + coin.body.width / 2, 'splash')

  this.splash.anims.play('splash_coin', true)
  coin.disableBody(true, true)
  console.log('XIVATO')
  // TODO -> ACTUALITZAR COMPTADOR/SCORE DE PUNTS --done

  score = score + 10
  // this.add.text(10, 10, 'score: ' + score, { fontSize: '12px', fill: '#000' })
  // TODO -> EXECUTAR SO QUE PERTOCA (coin.mp3) --done

  soundCoin.play()
}

export default {
  name: 'Game',
  mounted () {
    // PHASER 2.6 -> phaser-ce
    // PHASER 3.0 -> phaser
    let config = {
      type: Phaser.AUTO,
      width: 500,
      height: 200,
      physics: {
        default: 'arcade',
        arcade: {
          gravity: { y: 200 }
        }
      },
      // NO HI HA STATES A 3.0 -> SCENES
      scene: {
        preload () {
          console.log('PRELOAD')
          // CARREGAR ASSETS -> IMAGES, PLAYERS (SPRITESHEETS), AUDIOS, VIDEOS
          this.load.image('wall', wall)
          this.load.image('ground', ground)
          this.load.spritesheet('player', player, { frameWidth: 28, frameHeight: 22 })

          this.load.image('coin', coin)
          this.load.image('enemy', enemy)
          this.load.image('mobileEnemy', enemy)

          // loading AUDIO
          // this.load.setBaseURL('http://labs.phaser.io')
          this.load.audio('dieSound', soundDie)
          this.load.audio('coinSound', soundTakeCoin)
          // explosion
          this.load.spritesheet('boom', explosion, { frameWidth: 128, frameHeight: 128 })
          // coin effect
          this.load.spritesheet('coin_effect', splash, { frameWidth: 90, frameHeight: 90 })
          this.load.spritesheet('mobile_enemy_effect', explosionMobileEnemy, { frameWidth: 91, frameHeight: 90 })
        },
        create () {
          // jump
          this.JUMP_SPEED = -130
          this.jumping = false

          // INITILIZE del nivell -> Afegirem tiles (level: pareds, terras), afegirem player/s, collectibles (coins) , enemies
          console.log('CREATED')

          this.cameras.main.backgroundColor.setTo(52, 152, 219)

          // GROUP PER DEFECTE ES DINAMIC
          // this.level = this.physics.add.group()
          this.level = this.physics.add.staticGroup()
          this.levelWorld = this.physics.add.staticGroup()
          // this.worldWidth.create(config['width'], config['height'],)

          // this.add.image(500 / 2 - 160, 200 / 2, 'wall')
          // this.add.image(500 / 2 + 160, 200 / 2, 'wall')
          // this.ground = this.physics.add.image(500 / 2, 200 / 2 + 30, 'ground')

          this.level.create(500 / 2 - 160, 200 / 2, 'wall')
          this.wall1 = this.level.get(500 / 2 - 160, 200 / 2, 'wall').x + this.level.get(500 / 2 - 160, 200 / 2, 'wall').width / 2
          this.level.create(500 / 2 + 160, 200 / 2, 'wall')
          this.wall2 = this.level.get(500 / 2 + 160, 200 / 2, 'wall').x - this.level.get(500 / 2 - 160, 200 / 2, 'wall').width / 2
          this.level.create(500 / 2, 200 / 2 + 30, 'ground')
          // PLAYER -> FÍSIQUES
          this.player = this.physics.add.sprite(500 / 2, 200 / 2 - 50, 'player')
          this.mobileEnemy = this.physics.add.sprite(500 / 2, 200 / 2, 'enemy')
          this.physics.add.collider(this.mobileEnemy, this.level)
          this.enemyLeft = true
          this.enemyRight = false
          livesText = this.add.text(300, 10, 'Lives: 3', { fontSize: '12px', fill: '#000' })

          // this.physics.add.OVERLAP(this.player, this.mobileEnemy,lessLive)
          // colide mobile enemy with player
          // this.physics.arcade.overlap(this.player, this.mobileEnemy, lessLive)

          // this.physics.add.overlap(this.player, this.mobileEnemy, lessLive, null, this)

          this.player.setBounce(0.2)

          // this.player.setCollideWorldBounds(true)

          this.physics.add.collider(this.player, this.level)

          // DEFINE SOUNDS
          // this.sound.add('die')

          // this.dustSound = this.add.audio('dust', 0.1)
          // this.sound.add.audio('die', 0.1)
          soundDead = this.sound.add('dieSound')
          soundCoin = this.sound.add('coinSound')
          // let soundSample = this.add.audio('die')
          // if (typeof soundSample !== 'undifinded') { soundSample.play() }
          // soundSample.play()
          // soundDead = this.sound.add('die')
          // soundDead = this.sound.play('die')
          // jumping

          // cursors
          this.cursors = this.input.keyboard.createCursorKeys()

          // ANIMATE PLAYER
          this.anims.create({
            key: 'idle',
            frames: this.anims.generateFrameNumbers('player', { start: 3, end: 5 }),
            frameRate: 5,
            repeat: -1
          })
          // ANIMATE Explosion
          this.anims.create({
            key: 'explode',
            frames: this.anims.generateFrameNumbers('boom', { start: 0, end: 16 }),
            frameRate: 16,
            repeat: -1
          })
          // ANIMATE Explosion
          this.anims.create({
            key: 'splash_coin',
            frames: this.anims.generateFrameNumbers('coin_effect', { start: 0, end: 5 }),
            frameRate: 5,
            repeat: 0
          })
          // ANIMATE Explosion
          this.anims.create({
            key: 'splash_enemy',
            frames: this.anims.generateFrameNumbers('mobile_enemy_effect', { start: 0, end: 8 }),
            frameRate: 8,
            repeat: 0
          })

          // ADD COINS
          this.coins = this.physics.add.group()
          this.value = Phaser.Math.Between(1, 6)
          createRandomCoins(this.value, this.coins)
          // console.log('level walls' + this.level.walls);
          this.physics.add.collider(this.coins, this.level)
          this.physics.add.overlap(this.player, this.coins, takeCoin, null, this)

          // SCORE
          scoreText = this.add.text(100, 10, 'Score: 0', { fontSize: '12px', fill: '#000' })

          // ENEMIES

          this.enemies = this.physics.add.group()
          this.enemies.create(500 / 2 + 130, 200 / 2, 'enemy')

          this.physics.add.collider(this.enemies, this.level)
          this.physics.add.overlap(this.player, this.enemies, die, null, this)

          // collide mobile enemy
          this.physics.add.collider(this.player, this.mobileEnemy, lesslive, null, this)
          // this.physics.add.overlap(this.player, this.mobileEnemy, die, null, this)
          //  An explosion pool
          // explosions = this.add.group()
          // explosions.createMultiple(30, 'boom')

          // PREPARE LOSER TEXT
          this.loserText = this.add.text(500 / 2 - 50, 200 / 2 - 50, 'LOOSER!', { fontSize: '30px', fill: '#000' }).setVisible(false)
          this.gameOverText = this.add.text(500 / 2 - 100, 200 / 2 - 50, 'GAME OVER!', { fontSize: '40px', fill: '#fff' }).setVisible(false)

          this.cameras.main.on('camerashakestart', () => {
            this.loserText.setVisible(true)
          })

          this.cameras.main.on('camerashakecomplete', () => {
            this.loserText.setVisible(false)
          })
        },
        update () {
          scoreText.setText('Score: ' + score)

          livesText.setText('Lives: ' + lives)

          if (this.enemyLeft) {
            if (this.mobileEnemy.body.x <= this.wall1) {
              this.enemyRight = true
              this.enemyLeft = false
            }
            this.mobileEnemy.setVelocityX(-200)
          } else if (this.enemyRight) {
            if (this.mobileEnemy.body.x + this.mobileEnemy.body.width >= this.wall2) {
              this.enemyRight = false
              this.enemyLeft = true
            }
            this.mobileEnemy.setVelocityX(+200)
          }

          this.player.anims.play('idle', true)

          // INPUT EVENTS
          if (this.cursors.left.isDown) {
            this.player.setVelocityX(-160)
            this.player.setFrame(2)
          } else if (this.cursors.right.isDown) {
            this.player.setVelocityX(160)
            this.player.setFrame(1)
          } else {
            this.player.setVelocityX(0)
          }

          var onTheGround = this.player.body.touching.down

          // If the player is touching the ground, let him have 2 jumps
          if (onTheGround) {
            this.jumps = 2
            this.jumping = false
          }
          if (this.cursors.up.isDown && onTheGround) {
            this.player.body.velocity.y = this.JUMP_SPEED
            this.jumps--
          }
          if (this.cursors.up.isUp && this.jumps > 0) {
            this.jumping = true
          }
          if (this.cursors.up.isDown && !onTheGround && this.jumping) {
            this.player.body.velocity.y = this.JUMP_SPEED
            this.jumping = false
            this.jumps--
          }
          // out os creen
          if (this.player.body.y + this.player.height > config['height']) {
            // player.y > height
            this.levelWorld.create(this.player.body.x, config['height'] + this.player.height)
            this.physics.add.overlap(this.player, this.levelWorld, die, null, this)
          } else
          if (this.player.body.x + this.player.width > config['width']) {
            // player.x > width
            this.levelWorld.create(config['width'] + this.player.width, this.player.body.y)
            this.physics.add.overlap(this.player, this.levelWorld, die, null, this)
          } else
          if (this.player.body.x < 0) {
            // player.x < 0
            this.levelWorld.create(0 - this.player.width, this.player.body.y)
            this.physics.add.overlap(this.player, this.levelWorld, die, null, this)
          }
        }
      }
    }
    // eslint-disable-next-line
    new Phaser.Game(config)
  }
}
</script>
