<template>

	<div class="wrap">

		<h3>{{facetTitle}}</h3>

		<ul class="scrollContainer">

			<li v-for="i in countSortedList" @click="setFilter(facet,i.genus)" :class="{focused: filterState[facet]==i.genus}">
			<span>{{i.genus}}</span> 
			<span class="count">{{i.detections.length}}</span>
			</li>

		</ul>
	</div>


</template>

<script>
export default {

  name: 'FacetList',
  props: ['listData', 'facet','facetTitle', 'filterState'],

  data () {
    return {

    }
  },

  computed:{
  	countSortedList(){
  		return this.listData.sort((a,b) => b.detections.length - a.detections.length)
  	}
  },

  methods:{
  	setFilter(facet,value){
  		this.$emit("setFilter",facet,value)
  	}
  }
}
</script>

<style lang="css" scoped>

	.wrap{
		display: block;
		margin:0 2rem 2rem 0;
	}

	ul, li{
		list-style: none;
		text-indent: none;
		margin:0;
		padding:0;
	}

	ul.scrollContainer{
		height:420px;
/*		width:40%;*/
		width:320px;
		display: inline-block;
		overflow-y: scroll;
		background-color: rgba(234,234,224,0.5);
	}

	ul.scrollContainer li{
		font-family: Arial, sans-serif;
		font-size:0.9rem;
		display: block;
		margin:0.25rem 0;
		padding:0.1rem 0.5rem ;
		cursor: pointer;
		position:relative;
	}

	span.count{
		position:absolute;
		right:0.5rem;
	}

	ul.scrollContainer li:hover{
		background-color: #ddd;
		font-weight: 600;
/*		filter:invert();*/
	}

	ul.scrollContainer li.focused{
		color:white;
		font-weight: 600;
		background-color: #929281;
		position:sticky;
		top:0;
		z-index:1;
	}


</style>