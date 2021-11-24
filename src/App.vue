<template>
  <div id="app" class="app-container">
    <b-input-group class="align-self-center mt-4 mb-4" id="input-group">
      <b-input-group-prepend is-text>
        <b-icon icon="search"/>
      </b-input-group-prepend>
      <b-form-input v-model="search" type="search" placeholder="Filter by authors.."></b-form-input>
    </b-input-group>
    <b-container>
      <b-row>
        <v-col class="col-md-4" v-for="post in filteredPosts" :key="post.id">
          <b-card class="post-card mb-3 text-start">
            <b-card-title class="text-primary">{{post.title}}</b-card-title>
            <b-card-text>{{post.body}}</b-card-text>
            <small class="text-muted">{{post.author}}</small>
          </b-card>
        </v-col>
      </b-row>
    </b-container>
    <b-spinner class="align-self-center mb-4" variant="primary" v-if="loader"></b-spinner>
    <p v-if="filteredPosts && !filteredPosts.length && !error && !loader">Oops! Sorry, I don't have posts for you. You can try something else :)</p>
    <p v-if="this.error">Something went wrong. Please try again later.</p>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'App',
  data: function () {
    return {
      search: '',
      posts: [],
      error: false,
      loader: false,
    }
  },
  computed: {
    filteredPosts: function() {
      return this.posts?.filter(post => post.author.includes(this.search));
    }
  },
  async mounted() {
    this.loader = true;
    const promises = [this.downloadPosts(), this.downloadAuthors()];
    const [posts, authors] = await Promise.all(promises)
    if(posts && authors) {
      this.posts =  posts.map((post) => {
        let author = authors.find((author) => {
          if(author.id === post.userId) return author;
        })
        post.author = author.name;
        return post;
      })
      this.loader = false;
    }
  },
  methods: {
    async downloadPosts() {
      const response = await axios.get('http://jsonplaceholder.typicode.com/posts').catch(() => {
        this.error = true;
      });
      return response?.data;
    },
    async downloadAuthors() {
      const response = await axios.get('http://jsonplaceholder.typicode.com/users').catch(() => {
         this.error = true;
      });
      return response?.data;
    }
  }
}
</script>

<style scoped lang="less">

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  display: flex;
  justify-content: center;
  flex-direction: column;
  background: #EDF1F6;
  #input-group {
    max-width: 250px;
    /deep/ .input-group-text {
      display: block;
    }
  }
  .post-card {
    max-height: min-content;
  }
}

</style>
