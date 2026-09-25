<template>
	<div class="carousel">

		<div class="carousel__viewport" ref="viewport" @scroll="onScroll">
			<div class="carousel__track">
				<div class="carousel__spacer" v-if="items.length > 1" aria-hidden="true"></div>

				<div
					class="carousel__slide"
					:class="{ single: items.length === 1 }"
					v-for="(item, i) in items"
					:key="item.occurrenceID || i"
					:ref="el => setSlideRef(el, i)"
				>
					<ObsCard :obs="item" @set-filter="(...args) => $emit('set-filter', ...args)" />
				</div>

				<div class="carousel__spacer" v-if="items.length > 1" aria-hidden="true"></div>
			</div>
		</div>

		<div class="carousel__nav" v-if="items.length > 1">
			<button
				class="carousel__btn prev"
				aria-label="Previous observation"
				:disabled="activeIndex === 0"
				@click="goTo(activeIndex - 1)"
			>&#8249;</button>

			<span class="carousel__pagination">{{ activeIndex + 1 }} of {{ items.length }}</span>

			<button
				class="carousel__btn next"
				aria-label="Next observation"
				:disabled="activeIndex === items.length - 1"
				@click="goTo(activeIndex + 1)"
			>&#8250;</button>
		</div>

	</div>
</template>

<script>

	import ObsCard from './ObsCard.vue';

	export default {

	  name: 'FocusCarousel',
	  props: ['items'],
	  components: { ObsCard },
	  emits: ['set-filter'],

	  data () {
	    return {
	    	activeIndex: 0,
	    	slideRefs: [],
	    	scrollRAF: null
	    }
	  },

	  watch: {
	  	items(){
	  		this.slideRefs = [];
	  		this.activeIndex = 0;
	  		this.$nextTick(() => this.scrollToIndex(0, false));
	  	}
	  },

	  mounted () {
	  	this.$nextTick(() => this.scrollToIndex(0, false));
	  },

	  methods: {

	  	setSlideRef(el, i){
	  		if (el) this.slideRefs[i] = el;
	  	},

	  	goTo(i){
	  		if (i < 0 || i > this.items.length - 1) return;
	  		this.scrollToIndex(i, true);
	  	},

	  	scrollToIndex(i, smooth){
	  		const slide = this.slideRefs[i];
	  		const viewport = this.$refs.viewport;
	  		if (!slide || !viewport) return;

	  		const target = slide.offsetLeft - (viewport.clientWidth - slide.clientWidth) / 2;
	  		viewport.scrollTo({ left: target, behavior: smooth ? 'smooth' : 'instant' });
	  		this.activeIndex = i;
	  	},

	  	onScroll(){
	  		if (this.scrollRAF) cancelAnimationFrame(this.scrollRAF);
	  		this.scrollRAF = requestAnimationFrame(() => {
	  			const viewport = this.$refs.viewport;
	  			if (!viewport) return;

	  			const center = viewport.scrollLeft + viewport.clientWidth / 2;
	  			let closest = 0;
	  			let closestDist = Infinity;

	  			this.slideRefs.forEach((slide, i) => {
	  				if (!slide) return;
	  				const slideCenter = slide.offsetLeft + slide.clientWidth / 2;
	  				const dist = Math.abs(slideCenter - center);
	  				if (dist < closestDist) {
	  					closestDist = dist;
	  					closest = i;
	  				}
	  			});

	  			this.activeIndex = closest;
	  		});
	  	},

	  }
	}
</script>

<style lang="css" scoped>

	.carousel{
		position:relative;
		margin: 0 auto;
		flex: 1.25;
    	min-width: 320px;
	}

	.carousel__viewport{
		position:relative;
		overflow-x: auto;
		overflow-y: hidden;
		scroll-snap-type: x mandatory;
		scroll-behavior: smooth;
		-webkit-overflow-scrolling: touch;
		overscroll-behavior-x: contain;
	}

	/* Gradient overlay blending the peeking side cards into the page background */
	.carousel::after{
		content: "";
		position: absolute;
		z-index:2;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;

		pointer-events: none;

		background: linear-gradient(
			to right,
			rgb(244,244,241,120) 0%,
			rgba(244,244,241,0) 10%,
			rgba(244,244,241,0) 90%,
			rgb(244,244,241,120) 100%
		);
	}

	/* hide scrollbar */
	.carousel__viewport{
		scrollbar-width: none;
		-ms-overflow-style: none;
	}
	.carousel__viewport::-webkit-scrollbar{
		display: none;
	}

	.carousel__track{
		display: flex;
		flex-direction: row;
		align-items: stretch;
	}

	.carousel__slide{
		flex: 0 0 84%;
		max-width: 84%;
		scroll-snap-align: center;
		box-sizing: border-box;
		padding: 0 0.5rem;
	}

	.carousel__slide.single{
		flex-basis: 100%;
		max-width: 100%;
	}

	/* spacers reserve half the peek gap at each end of the track so the
	   first/last slides can scroll fully to center, matching the peek
	   space that real neighbor slides provide in the middle of the list */
	.carousel__spacer{
		flex: 0 0 8%;
		max-width: 8%;
	}

	.carousel__nav{
		display: flex;
		flex-direction: row;
		align-items: center;
		justify-content: center;
		gap: 1rem;
		margin-top: 0.5rem;
	}

	.carousel__btn{
		background: none;
		border: none;
		font-size: 1.5rem;
		line-height: 1;
		cursor: pointer;
		color: #444;
		padding: 0.25rem 0.5rem;
	}

	.carousel__btn:disabled{
		opacity: 0.3;
		cursor: default;
	}

	.carousel__pagination{
		font-size: 80%;
		font-weight: 300;
		color: #444;
		min-width: 5rem;
		text-align: center;
	}

	@media (max-width: 768px){
		.carousel{
			max-width: 100%;
		}
	}

</style>
