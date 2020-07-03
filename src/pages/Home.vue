<template>
  <div class="wrapper-content wrapper-content--fixed">
    <section>
      <div class="container">

        <!-- errors -->
        <div class="error" v-if="error" style="margin-bottom:20px;">
          <p>{{ error }}</p>
        </div>

        <!-- search -->
        <search
          :value="search" 
          placeholder="Type username..." 
          @search=" search = $event" />

        <!-- buttons -->
        <button v-if="!repos" class="btn btnPrimary" @click="getRepos">Search</button>
        <button v-else class="btn btnPrimary" @click="getRepos">Search Again</button>

        <!-- user -->
        <div class="user__wrapper">
          <div v-if="repos" class="user-item">
              <img :src="userInfo.img" alt="">
              <span class="user-item__name">{{userInfo.name}}</span>
              <span class="user-item__count">Published of repos: {{userInfo.publicRepos}}</span>
          </div>
        </div>

        <!-- wrapper -->
        <div class="repos__wrapper" v-if="repos">
          <table>
            <thead>
              <tr>
                <th @click="sortRepos('name')">Name &#8595;</th>
                <th @click="sortRepos('stargazers_count')">Stars &#8595;</th>
              </tr>
            </thead>
          </table>

          <!-- item -->
          <div class="repos-item" v-for="repo in reposSort" :key="repo.id">
            <div class="repos-info">
              <a class="link" target="_blank" :href="repo.html_url">{{repo.name}}</a>
              <span>{{ repo.stargazers_count }} &#127775;</span>
            </div>
          </div>

        </div>
      </div>
    </section>
    <pagination :repos="repos" :page="page" />
  </div>
</template>

<script>
import search from "@/components/Search.vue"
import pagination from "@/components/Pagination.vue"
import axios from "axios"

export default {
  components: {
    search,
    pagination
  },
  data() {
    return {
      search: 'github',
      error: '',
      repos: null,
      userInfo: {},
      currentSort: 'name',
      currentSortDir: 'asc',
      page: {
        current: 1,
        length: 5
      }
    }
  },
  mounted() {
    this.getRepos()
  },
  computed: {
    // Sorting
    reposSort() {
      return this.repos
        .sort((a, b) => {
          let mod = 1,
              first = a[this.currentSort],
              second = b[this.currentSort]

          if (this.currentSortDir === 'desc') mod = -1
          if (typeof (first && second) === 'number') return (first - second) * mod
          if (first.toLowerCase() < second.toLowerCase()) return -1 * mod
          return 1 * mod
        })
        .filter((row, index) => {
          let start = (this.page.current - 1) * this.page.length,
              end = this.page.current * this.page.length
          if (index >= start && index < end) return true
        })
    }
  },
  methods: {
    async getRepos() {
      this.repos = null
      try {
        const [API_URL_GITHUB, usersInfo] = await Promise.all([
            axios.get(`https://api.github.com/users/${this.search}/repos`), //запрос repos
            axios.get(`https://api.github.com/users/${this.search}`), //запрос name, published, avatar, id
          ])

          this.repos = API_URL_GITHUB.data
          this.error = ''

          this.userInfo = {
            id: usersInfo.data.id,
            name: usersInfo.data.name,
            img: usersInfo.data.avatar_url,
            publicRepos: usersInfo.data.public_repos
          }

      }
      catch(err) {
        console.log(err)
        this.repos = null
        this.error = `Cant't find this user`
      }
    },
    sortRepos(e) {
      if (e === this.currentSort) {
        this.currentSortDir = this.currentSortDir === "asc" ? "desc" : "asc"
      }
      this.currentSort = e
    },
  }
}
</script>

<style lang="scss" scoped>
.container {
  display: flex;
  align-items: center;
  flex-direction: column;
}
button {
  margin-top: 40px;
}
.repos__wrapper {
  width: 400px;
  margin-bottom: 30px;
}
.repos-info {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 10px;
  padding: 10px 0;
  border-bottom: 1px solid #dbdbdb;
}
.user__wrapper {
  width: 400px;
  margin: 30px 0;
}
.user-item {
  display: flex;
  align-items: left;
  flex-direction: column;
  &__name {
    font-size: 30px;
    font-weight: 600;
  }
}
img {
  border-radius: 20px;
}
span {
  margin-top: 25px;
  font-size: 20px;
}
table thead th {
    padding: 0px;
}

tr {
  display: flex;
  justify-content: space-between;
}
</style>