<template>
    <div id="game"></div>
</template>

<script>
import Phaser from 'phaser'
import tile from '../assets/tilesets/slopes32mud.png'
import coin from '../assets/img/coin7.png'
import Player from '../assets/img/player.png'
import map from '../assets/tilemaps/map.json'
var player
var cursors
let CoinLayer
let coins
let mapGame
let coinScore = 0
let text
// let coinScore = 0
function createCoins () {
  CoinLayer.forEach(object => {
    let obj = coins.create(object.x, object.y, 'coin')
    obj.setScale(object.width / 32, object.height / 32)
    obj.setOrigin(0)
    obj.body.width = object.width
    obj.body.height = object.height
    obj.anims.play('spin', coins)
  })
}
function collectCoin (player, coin) {
  coin.destroy(coin.x, coin.y) // remove the tile/coin
  coinScore++ // increment the score
  text.setText(`Coins: ${coinScore}x`) // set the text to show the current score
  return false
}

export default {
  name: 'Game',
  mounted () {
    // PHASER 2.6 -> phaser-ce
    // PHASER 3.0 -> phaser
    let config = {
      width: 900,
      height: 700,
      backgroundColor: '#000000',
      parent: 'phaser-example',
      pixelArt: true,
      physics: {
        default: 'arcade',
        arcade: {
          gravity: { y: 300 }
        } },
      scene: {
        preload: preload,
        create: create,
        update: update
      }
    }
    // eslint-disable-next-line
      new Phaser.Game(config)
  }
}

function preload () {
  console.log('PRELOAD')
  this.load.image('tiles', tile)
  this.load.spritesheet('coin', coin, { frameWidth: 32, frameHeight: 32 })
  this.load.spritesheet('player', Player, { frameWidth: 28, frameHeight: 22 })
  this.load.tilemapTiledJSON('map', map)
}
function create () {
  console.log('Created')
  mapGame = this.make.tilemap({ key: 'map' })
  const tileset = mapGame.addTilesetImage('slopes32mud', 'tiles')
  const worldLayer = mapGame.createStaticLayer('collision', tileset, 0, 0)
  const innerWorldLayer = mapGame.createStaticLayer('map', tileset, 0, 0)
  CoinLayer = mapGame.getObjectLayer('CoinLayer')['objects']

  // coins
  coins = this.physics.add.staticGroup()

  worldLayer.setCollisionBetween(0, 140)
  innerWorldLayer.setCollisionBetween(0, 140)
  //
  player = this.physics.add.sprite(500 / 2, mapGame.heightInPixels - mapGame.heightInPixels / 5, 'player')
  this.physics.add.collider(player, worldLayer)
  this.physics.add.collider(player, innerWorldLayer)

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
    frameRate: 16,
    repeat: -1
  })
  createCoins()
  this.physics.add.overlap(player, coins, collectCoin, null, this)
  this.JUMP_SPEED = 400
  this.cameras.main.setBounds(0, 0, mapGame.widthInPixels, mapGame.heightInPixels)
  this.cameras.main.startFollow(player)

  cursors = this.input.keyboard.createCursorKeys()

  var help = this.add.text(16, 16, 'Arrows to move. "up" to jump.', {
    fontSize: '18px',
    fill: '#ffffff'
  })
  help.setScrollFactor(0)
  // score
  text = this.add.text(570, 70, `Coins: ${coinScore}x`, {
    fontSize: '20px',
    fill: '#ffffff'
  })
  text.setScrollFactor(0)
}
function update () {
  player.anims.play('idle', true)
  if (cursors.left.isDown) {
    player.setAccelerationX(-200)
    player.setFrame(2)
  } else if (cursors.right.isDown) {
    player.setAccelerationX(200)
    player.setFrame(1)
  } else {
    player.setAccelerationX(0)
  }
  var onTheGround = player.body.onFloor()

  if (onTheGround) {
    this.jumps = 2
    this.jumping = false
  }
  if (cursors.up.isDown && onTheGround) {
    console.log('hola salto 1')
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
</script>
<style>

    canvas {
        display: block;
        margin: auto;
    }
    /*#game{*/
    /**/
    /*}*/
</style>
