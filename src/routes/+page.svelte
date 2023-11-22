<script>
	// @ts-nocheck

	import { Spritesheet, BaseTexture, AnimatedSprite, Application, SCALE_MODES } from 'pixi.js';
	import { onMount } from 'svelte';
	import playerSheet from '$lib/assets/player-sheet.png';

	onMount(async () => {
		const appWidth = 500;
		const appHeight = 400;
		const app = new Application({
			width: appWidth,
			height: appHeight,
			backgroundColor: 'blue',
			antialias: true // pixel smoothing related?
		});

		// @ts-ignore
		document.getElementById('game')?.appendChild(app.view);

		let keys = {};
		window.addEventListener('keydown', (e) => (keys[e.key] = true));
		window.addEventListener('keyup', (e) => (keys[e.key] = false));

		// Player width and height
		const scale = 2;
		const pw = 32 * scale;
		const ph = 32 * scale;

		let playerInfo = {
			speed: 1.5
		};

		// Create object to store sprite sheet data
		const generateSpriteData = (name, offset) => ({
			frame: { x: offset * pw, y: 0, w: pw, h: ph },
			sourceSize: { w: pw, h: ph },
			spriteSourceSize: { x: 0, y: 0, w: pw, h: ph }
		});
		const playerSpriteNames = [
			'left_s',
			'left_0',
			'left_1',
			'down_s',
			'down_0',
			'down_1',
			'up_s',
			'up_0',
			'up_1',
			'right_s',
			'right_0',
			'right_1'
		];
		const playerAnimData = {
			frames: playerSpriteNames.reduce((accum, name, index) => {
				accum[name] = generateSpriteData(name, index);
				return accum;
			}, {}),
			meta: {
				image: playerSheet,
				// format: 'RGBA8888',
				size: { w: 384 * scale, h: ph * scale },
				scale: '1'
			},
			animations: {
				walkLeft: ['left_0', 'left_s', 'left_1', 'left_s'],
				walkRight: ['right_0', 'right_s', 'right_1', 'right_s'],
				walkDown: ['down_0', 'down_s', 'down_1', 'down_s'],
				walkUp: ['up_0', 'up_s', 'up_1', 'up_s']
			}
		};

		console.log('plaYER ANIM DATA', playerAnimData);

		// Create the SpriteSheet from data and image
		const spritesheet = new Spritesheet(
			BaseTexture.from(playerAnimData.meta.image),
			playerAnimData
		);

		// Generate all the Textures asynchronously
		await spritesheet.parse();

		const createPlayer = () => {
			// spritesheet is ready to use!
			const playerSprite = new AnimatedSprite([spritesheet.textures.down_s]);
			// set the animation speed
			playerSprite.animationSpeed = 0.1;
			playerSprite.anchor.set(0.5);
			playerSprite.loop = false;
			// player.scale = 2;
			playerSprite.x = app.view.width / 2;
			playerSprite.y = app.view.height / 2;

			// playerSprite.scale.x = playerSprite.scale.x / scale;
			// playerSprite.scale.y = playerSprite.scale.y / scale;
			playerSprite.texture.baseTexture.scaleMode = SCALE_MODES.NEAREST;
			app.stage.addChild(playerSprite);
			return playerSprite;
		};

		const player = createPlayer();

		const movePlayer = () => {
			const updateAnim = (animationName) => {
				if (!player.playing || playerInfo.anim !== animationName) {
					playerInfo.anim = animationName;
					player.textures = spritesheet.animations[animationName];
					player.play();
				}
			};
			if (keys['ArrowRight']) {
				updateAnim('walkRight');
				player.x = player.x + playerInfo.speed;
			} else if (keys['ArrowLeft']) {
				updateAnim('walkLeft');
				player.x = player.x - playerInfo.speed;
			} else if (keys['ArrowUp']) {
				updateAnim('walkUp');
				player.y = player.y - playerInfo.speed;
			} else if (keys['ArrowDown']) {
				updateAnim('walkDown');
				player.y = player.y + playerInfo.speed;
			}
			// If no keyboard movement is detected and an animation is playing, kill the animation.
			else if (
				['walkLeft', 'walkUp', 'walkDown', 'walkRight'].includes(playerInfo.anim) &&
				player.playing
			) {
				if (playerInfo.anim === 'walkLeft') {
					player.textures = [spritesheet.textures.left_s];
				} else if (playerInfo.anim === 'walkRight') {
					player.textures = [spritesheet.textures.right_s];
				} else if (playerInfo.anim === 'walkUp') {
					player.textures = [spritesheet.textures.up_s];
				} else if (playerInfo.anim === 'walkDown') {
					player.textures = [spritesheet.textures.down_s];
				}
				playerInfo.anim = null;
			}
		};

		const gameLoop = () => {
			movePlayer();
		};

		app.ticker.add(gameLoop);
	});
</script>

<h3>Hi there</h3>
<div id="game"></div>
