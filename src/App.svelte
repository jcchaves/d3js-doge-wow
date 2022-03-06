<script>
	import { onMount } from "svelte";
	import * as d3 from "d3";

	let svg;
	let wow;
	let wowPosition = { x: 0, y: 0 };
	let muchRainbow;
	let muchRainbowPosition = { x: 0, y: 0 };
	let soColourful;
	let soColourfulPosition = { x: 0, y: 0 };
	let veryFancy;
	let veryFancyPosition = { x: 0, y: 0 };
	let locks = {
		wow: { movement: {}, rainbow: {} },
		muchRainbow: { movement: {}, rainbow: {} },
		soColourful: { movement: {}, rainbow: {} },
		veryFancy: { movement: {}, rainbow: {} },
	};
	const coinsAmount = 3;
	let coinsData = [];

	function setupCoinsData() {
		const curTimestamp = Date.now();
		[...Array(coinsAmount).keys()].forEach((d) => {
			coinsData.push({
				id: curTimestamp + d,
				x: Math.random() * 640,
				y: 0,
				url: "/assets/doge_coin.svg"
			});
		});
	}

	onMount(async () => {
		setupCoinsData();

		svg = d3.select("svg").attr("width", 640).attr("height", 480);

		wow = d3.xml("/assets/WOW.svg");
		muchRainbow = d3.xml("/assets/much_rainbow.svg");
		soColourful = d3.xml("/assets/so_colourful.svg");
		veryFancy = d3.xml("/assets/very_fancy.svg");

		setupCoins();

		setup(wow, "wow", locks.wow, wowPosition);
		setup(
			muchRainbow,
			"muchRainbow",
			locks.muchRainbow,
			muchRainbowPosition
		);
		setup(
			soColourful,
			"soColourful",
			locks.soColourful,
			soColourfulPosition
		);
		setup(veryFancy, "veryFancy", locks.veryFancy, veryFancyPosition);
		// Breaking change
		// setup(veryFancy, "veryFancy", locks.wow, wowPosition);
	});

	function setupCoins() {
		const nodes = svg
			.selectAll(".coin")
			.data(coinsData, (datum) => datum.id);
		
		const coins = nodes
			.enter()
			.append("image")
			.attr("xlink:href", (datum) => datum.url)
			.attr("x", (datum) => datum.x)
			.attr("y", (datum) => datum.y)
			.attr("width", "75px")
			.attr("height", "75px")
			.classed("coin", true);

		coins
			.style("opacity", 1)
			.transition()
			.duration(Math.random() * 1000 + 2000)
			.tween("attr:cy", () => {
				const i = d3.interpolateNumber(0, Math.random() * 480 + 100);
				return function (t) {
					this.setAttribute("y", i(t));
				};
			})
			.transition()
			.duration(350)
			.styleTween("opacity", () => {
				const i = d3.interpolateNumber(0, 1);
				return (t) => {
					return i(t) * -1 + 1;
				};
			})
			.on("end", (datum) => {
				coinsData = coinsData.filter((d) => d.id !== datum.id);
				addCoin(datum.id);
				setupCoins()
				console.log(coinsData);
			});

			nodes.exit().remove();
		
	}

	function addCoin(seed) {
		const prob = Math.random();
		let url = "/assets/doge_coin.svg";
		if (coinsAmount > 5 && prob < 0.05) {
			url = "/assets/shiba_inu_coin.svg"
		}
		const curTimestamp = Date.now();
		coinsData.push({
			id: curTimestamp + seed,
			x: Math.random() * 640,
			y: 0,
			url
		});
	}

	function setup(importedSvg, gNodeId, svgLocks, position) {
		importedSvg.then((data) => {
			svg.node().append(data.documentElement.getElementById(gNodeId));

			const gNode = d3
				.select(`#${gNodeId}`)
				.attr("transform", `translate(${position.x},${position.y})`);

			rainbow(gNode, svgLocks.movement, 3000);
			movement(gNode, svgLocks.rainbow, 2500, position, 0, 45, true);
		});
	}

	function rainbow(gNode, lock, duration) {
		d3.select(lock)
			.transition()
			.duration(duration)
			.ease(d3.easeLinear)
			.tween("style:fill", () => {
				return (t) => {
					gNode.style("fill", "hsl(" + t * 360 + ", 100%, 50%)");
				};
			})
			.on("end", () => rainbow(gNode, lock, duration));
	}

	function movement(
		wowGNode,
		lock,
		duration,
		position,
		startDeg,
		endDeg,
		rotateForward
	) {
		d3.select(lock)
			.transition()
			.duration(duration)
			.ease(d3.easeLinear)
			.tween("attr:transform", () => {
				const i = d3.interpolateNumber(startDeg, endDeg);
				const targetX = Math.random() * 640 * 2;
				const targetY = Math.random() * 480 * 2;
				let interpXEnd = targetX;
				if (targetX - position.x < 0) {
					interpXEnd = position.x;
				}
				let interpYEnd = targetY;
				if (targetY - position.y < 0) {
					interpYEnd = position.y;
				}
				const translationXInterp = d3.interpolateNumber(
					position.x,
					interpXEnd - position.x
				);
				const translationYInterp = d3.interpolateNumber(
					position.y,
					interpYEnd - position.y
				);
				return (t) => {
					position.x = translationXInterp(t);
					position.y = translationYInterp(t);
					const deg = i(t);
					wowGNode.attr(
						"transform",
						`scale(0.5) translate(${position.x}, ${position.y}) rotate(${deg} 135 35)`
					);
				};
			})
			.on("end", () =>
				movement(
					wowGNode,
					lock,
					duration,
					position,
					endDeg,
					rotateForward ? -45 : 45,
					!rotateForward
				)
			);
	}
</script>

<main>
	<svg id="container" />
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	svg {
		background: url("/assets/shiba.jpg");
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>
