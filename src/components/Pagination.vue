<template>
	<section>
      <div class="container" v-if="repos">
        <div class="button-list">
          <div class="btn btnPrimary" @click="prevPage" >&#8592;</div>
          <div class="btn btnPrimary" @click="nextPage" >&#8594;</div>
        </div>
      </div>
    </section>
</template>

<script>
export default {
	props: {
    repos: {
      type: Array,
      default: false
		},
		page: {
		  type: Object,
    	required: true
		}
  },
  data() {
		return {
			finished: false
		}
	},
	methods: {
	//Pagination
    prevPage () {
      if (this.page.current > 1) this.page.current -= 1
    },
    nextPage () {
      if (this.page.current * this.page.length < this.repos.length)
      this.page.current += 1
    },
	}
}
</script>
<style lang="scss">
.container {
  display: flex;
  align-items: center;
  flex-direction: column;
}
.button-list {
	display: flex;
	align-items: center;
  :last-child {
    margin-right: 0px;
  }
}
.btn {
  margin-right: 20px;
}
</style>