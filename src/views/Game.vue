<template>
    <div id="game"></div>
</template>

<script>
// assets
import Phaser from 'phaser'
import tile from '../assets/tilesets/slopes32mud.png'
import coin from '../assets/img/coin7.png'
import Player from '../assets/img/player.png'
import Harpy from '../assets/img/fly_7.png'
import Spark from '../assets/particles/blue.png'
import map from '../assets/tilemaps/map.json'
import soundDie from '../assets/audio/dead.mp3'
import soundTakeCoin from '../assets/audio/coin.mp3'
// sounds levels
import soundlevel1 from '../assets/audio/levels/song_level1.mp3'
import soundlevel2 from '../assets/audio/levels/song_level2.mp3'

// variables of game
var player
var cursors
let CoinLayer
let EnemyLayer
let enemies
let coins
let mapGame
let coinScore = 0
let text
let emitter
var particles
let level2 = falsegit status
let help

// sounds
let soundDead
let soundCoin
// sound levels
let soundlvl1
let soundlvl2
// let configSoundlvl1 = {
//   mute: false,
//   volume: 1,
//   rate: 1,
//   detune: 0,
//   seek: 0,
//   loop: true,
//   delay: 0
// }

let worldLayer
let innerWorldLayer

function changeDirection (enemy) {
  enemy.speedX = enemy.speedX * -1
}
// let coinScore = 0
function createCoins () {
  CoinLayer.forEach(object => {
    let obj = coins.create(object.x, object.y, 'coin')
    // obj.setScale(object.width / 32, object.height / 32)
    // obj.setOrigin(0)
    obj.body.width = object.width
    obj.body.height = object.height
    obj.anims.play('spin', coins)
    // obj.body.allowGravity = false
    // obj.enableBody=true
  })
}
function createEnemies () {
  EnemyLayer.forEach(object => {
    let obj = enemies.create(object.x, object.y, 'harpy')
    obj.setScale(object.width / 64, object.height / 65)
    // obj.setOrigin(0)
    obj.body.width = object.width
    obj.body.height = object.height
    obj.staticY = object.y
    obj.id = object.id
    obj.speedX = -100
    obj.body.allowGravity = false
  })
  // enemies.refresh()
}
function collectCoin (player, coin) {
  soundCoin.play()
  coin.destroy(coin.x, coin.y) // remove the tile/coin
  coinScore++ // increment the score
  text.setText(`Coins: ${coinScore} - 16`) // set the text to show the current score
  if (coins.children.entries.length === 0) {
    if (level2) {
      this.scene.start('Winner')
    } else {
      level2 = true
      this.scene.restart()
      // help.setText(`Collect all coins and evade Enemies to win :)`) // set the text to show the current score

      // createEnemies()
      coinScore = 0
    }
  }

  return false
}

function gameOver () {
  if (soundlvl2) {
    soundlvl2.stop()
  }
  soundDead.play()
  this.scene.start('GameOver')
}
let config

export default {
  name: 'game',
  mounted () {
    // PHASER 2.6 -> phaser-ce
    // PHASER 3.0 -> phaser
    config = {
      width: 900,
      height: 700,
      backgroundColor: '#000000',
      parent: 'phaser-example',
      pixelArt: true,
      physics: {
        default: 'arcade',
        arcade: {
          gravity: { y: 300 },
          debug: true
        } },
      scene: [Loading, Game, GameOver, Winner]
    }
    // eslint-disable-next-line
      new Phaser.Game(config)
  }
}
class Game extends Phaser.Scene {
  constructor () {
    super({ key: 'Game' })
  }
  preload () {
    console.log('PRELOAD')
  }
  create () {
    console.log('CREATED')
    // *true* param enables looping
    // soundDead = this.sound.add('dieSound')
    if (!level2) {
      if (soundlvl2) {
        soundlvl2.stop()
      }
      soundlvl1 = this.sound.add('sound_level1', { loop: true })
      soundlvl1.play()
    } else {
      if (soundlvl1) {
        soundlvl1.stop()
      }
      soundlvl2 = this.sound.add('sound_level2', { loop: true })
      soundlvl2.play()
    }

    // soundlvl1 = new Phaser.Sound(this, 'sound_level1', 1, true)
    //
    // soundlvl1.play()
    mapGame = this.make.tilemap({ key: 'map' })
    const tileset = mapGame.addTilesetImage('slopes32mud', 'tiles')
    worldLayer = mapGame.createStaticLayer('collision', tileset, 0, 0)
    innerWorldLayer = mapGame.createStaticLayer('map', tileset, 0, 0)
    CoinLayer = mapGame.getObjectLayer('CoinLayer')['objects']
    EnemyLayer = mapGame.getObjectLayer('EnemyLayer')['objects']
    // this.physics.startSystem(Phaser.Physics.ARCADE);
    // coins
    coins = this.physics.add.staticGroup()
    enemies = this.physics.add.group()
    enemies.enableBody = true

    worldLayer.setCollisionBetween(0, 140)
    innerWorldLayer.setCollisionBetween(0, 140)
    //
    player = this.physics.add.sprite(500 / 2, mapGame.heightInPixels - mapGame.heightInPixels / 5, 'player')

    this.physics.add.collider(player, worldLayer)
    this.physics.add.collider(player, innerWorldLayer)

    // sounds
    soundDead = this.sound.add('dieSound')
    soundCoin = this.sound.add('coinSound')

    // anims
    this.anims.create({
      key: 'idle',
      frames: this.anims.generateFrameNumbers('player', { start: 3, end: 5 }),
      frameRate: 5,
      repeat: -1
    })
    // coin
    this.anims.create({
      key: 'spin',
      frames: this.anims.generateFrameNumbers('coin', { start: 0, end: 6 }),
      frameRate: 10,
      repeat: -1
    })
    this.anims.create({
      key: 'movement_harpy_right',
      frames: this.anims.generateFrameNumbers('harpy', { start: 4, end: 7 }),
      frameRate: 9,
      repeat: -1
    })
    this.anims.create({
      key: 'movement_harpy_left',
      frames: this.anims.generateFrameNumbers('harpy', { start: 0, end: 3 }),
      frameRate: 9,
      repeat: -1
    })
    createCoins()
    if (level2) {
      createEnemies()
    }

    this.physics.add.collider(enemies, worldLayer, changeDirection)
    this.physics.add.collider(enemies, innerWorldLayer, changeDirection)
    // Phaser.Actions.Call(EnemyLayer.getChildren(), function (letter) {
    //   letter.body.allowGravity = false;
    // });

    this.physics.add.overlap(player, coins, collectCoin, null, this)
    this.physics.add.overlap(player, enemies, gameOver, null, this)
    this.JUMP_SPEED = 400
    this.cameras.main.setBounds(0, 0, mapGame.widthInPixels, mapGame.heightInPixels)
    this.cameras.main.startFollow(player)

    cursors = this.input.keyboard.createCursorKeys()
    help = this.add.text(16, 16, level2 ? `Collect all coins and evade Enemies to win :)` : 'Arrows to move & jump. Collect all coins to get next level.', {
      fontSize: '18px',
      fill: '#ffffff'
    })
    help.setScrollFactor(0)
    // score
    text = this.add.text(570, 70, `Coins: ${coinScore} - 16`, {
      fontSize: '20px',
      fill: '#ffffff'
    })
    text.setScrollFactor(0)
    particles = this.add.particles('spark')

    emitter = particles.createEmitter({
      x: player.body.x + player.body.width / 2,
      y: player.body.y + player.body.height,
      angle: { min: 140, max: 180 },
      speed: 200,
      gravityY: 300,
      scale: { start: 0.2, end: 0 },
      lifespan: { min: 500, max: 1000 },
      blendMode: 'ADD'
    })
  }
  update () {
    if (enemies) {
      enemies.children.entries.forEach(function (enemy) {
        enemy.setVelocityX(enemy.speedX)
        // enemy.setY(enemy.staticY)
        enemy.speedX < 0 ? enemy.anims.play('movement_harpy_left', enemies) : enemy.anims.play('movement_harpy_right', enemies)
      })
    }
    player.anims.play('idle', true)
    if (cursors.left.isDown) {
      player.setAccelerationX(-200)
      player.setFrame(2)
      emitter.setPosition(player.body.x + player.body.width, player.body.y + player.body.height)
    } else if (cursors.right.isDown) {
      player.setAccelerationX(200)
      player.setFrame(1)
      emitter.setPosition(player.body.x, player.body.y + player.body.height)
    } else {
      player.setAccelerationX(0)
      emitter.setPosition(player.body.x + player.body.width / 2, player.body.y + player.body.height)
    }

    var onTheGround = player.body.onFloor()

    if (onTheGround) {
      this.jumps = 2
      this.jumping = false
    }
    if (cursors.up.isDown && onTheGround) {
      player.setVelocityY(-this.JUMP_SPEED)
      this.jumps--
    }
    if (cursors.up.isUp && this.jumps > 0) {
      this.jumping = true
    }
    if (cursors.up.isDown && !onTheGround && this.jumping) {
      player.setVelocityY(-this.JUMP_SPEED)
      this.jumping = false
      this.jumps--
    }
  }
}

class GameOver extends Phaser.Scene {
  constructor () {
    super({ key: 'GameOver' })
  }
  create () {
    // let clickCount = 0;
    this.clickCountText = this.add.text(180, config['height'] / 2 - 100, '')
    this.clickCountText.setText(`GAME OVER!`)
    this.clickCountText.setFontSize(100)

    this.clickButton = this.add.text(config['width'] / 2 - 50, config['height'] / 2 + 100, 'RESTART!', { fill: '#0f0' })
      .setInteractive()
      .on('pointerover', () => this.enterButtonHoverState())
      .on('pointerout', () => this.enterButtonRestState())
      .on('pointerdown', () => this.enterButtonActiveState())
      .on('pointerup', () => {
        // this.updateClickCountText(++clickCount);
        this.enterButtonHoverState()
        level2 = false
        coinScore = 0

        this.scene.start('Game')
      })
  }

  enterButtonHoverState () {
    this.clickButton.setStyle({ fill: '#ff0' })
  }

  enterButtonRestState () {
    this.clickButton.setStyle({ fill: '#0f0' })
  }

  enterButtonActiveState () {
    this.clickButton.setStyle({ fill: '#0ff' })
  }
}
class Winner extends Phaser.Scene {
  constructor () {
    super({ key: 'Winner' })
  }
  create () {
    // let clickCount = 0;
    this.clickCountText = this.add.text(200, config['height'] / 2 - 200, '')
    this.clickCountText2 = this.add.text(100, config['height'] / 2 - 100, '')
    this.clickCountText.setText(`You Are a `)
    this.clickCountText.setColor('#0ff')
    this.clickCountText.setFontSize(90)
    this.clickCountText2.setText(`crazy WINNER !`)
    this.clickCountText2.setColor('#ff0')
    this.clickCountText2.setFontSize(90)

    this.clickButton = this.add.text(config['width'] / 2 - 50, config['height'] / 2 + 100, 'New Game!', { fill: '#0f0' })
      .setInteractive()
      .on('pointerover', () => this.enterButtonHoverState())
      .on('pointerout', () => this.enterButtonRestState())
      .on('pointerdown', () => this.enterButtonActiveState())
      .on('pointerup', () => {
        // this.updateClickCountText(++clickCount);
        this.enterButtonHoverState()
        level2 = false
        coinScore = 0

        this.scene.start('Game')
      })
  }

  enterButtonHoverState () {
    this.clickButton.setStyle({ fill: '#ff0' })
  }

  enterButtonRestState () {
    this.clickButton.setStyle({ fill: '#0f0' })
  }

  enterButtonActiveState () {
    this.clickButton.setStyle({ fill: '#0ff' })
  }
}
class Loading extends Phaser.Scene {
  constructor () {
    super({ key: 'Loading' })
  }
  preload () {
    var progressBar = this.add.graphics()
    var progressBox = this.add.graphics()
    progressBox.fillStyle(0x222222, 0.8)
    progressBox.fillRect(240, 270, 320, 50)

    var width = this.cameras.main.width
    var height = this.cameras.main.height
    var loadingText = this.make.text({
      x: width / 2,
      y: height / 2 - 50,
      text: 'Loading...',
      style: {
        font: '20px monospace',
        fill: '#ffffff'
      }
    })
    loadingText.setOrigin(0.5, 0.5)

    var percentText = this.make.text({
      x: width / 2,
      y: height / 2 - 5,
      text: '0%',
      style: {
        font: '18px monospace',
        fill: '#ffffff'
      }
    })
    percentText.setOrigin(0.5, 0.5)

    var assetText = this.make.text({
      x: width / 2,
      y: height / 2 + 50,
      text: '',
      style: {
        font: '18px monospace',
        fill: '#ffffff'
      }
    })

    assetText.setOrigin(0.5, 0.5)

    this.load.on('progress', function (value) {
      percentText.setText(parseInt(value * 100) + '%')
      progressBar.clear()
      progressBar.fillStyle(0xffffff, 1)
      progressBar.fillRect(250, 280, 300 * value, 30)
    })

    this.load.on('fileprogress', function (file) {
      assetText.setText('Loading asset: ' + file.key)
    })

    this.load.on('complete', function () {
      progressBar.destroy()
      progressBox.destroy()
      loadingText.destroy()
      percentText.destroy()
      assetText.destroy()
    })
    this.load.image('tiles', tile)
    this.load.spritesheet('coin', coin, { frameWidth: 32, frameHeight: 32 })
    this.load.spritesheet('player', Player, { frameWidth: 28, frameHeight: 22 })
    this.load.tilemapTiledJSON('map', map)
    this.load.image('spark', Spark)
    this.load.spritesheet('harpy', Harpy, { frameWidth: 60, frameHeight: 65 })
    this.load.audio('dieSound', soundDie)
    this.load.audio('coinSound', soundTakeCoin)
    this.load.audio('sound_level1', soundlevel1)
    this.load.audio('sound_level2', soundlevel2)
  }

  create () {
    this.scene.start('Game')
  }
}
</script>
<style>

    canvas {
        display: block;
        margin: auto;
    }
</style>
