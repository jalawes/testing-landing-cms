<template>
  <section class="text-gray-600">
    <div class="py-12 mx-auto" v-if="posts.length > 0">
      <div class="flex flex-wrap -mx-4 -my-8">
        <nuxt-link :to="`${postType}/${post.slug}`" v-for="(post, index) in posts" :key="index" class="py-8 px-4 lg:w-1/3 rounded bg-white">
          <div class="h-full flex items-start">
            <div class="w-12 flex-shrink-0 flex flex-col text-center leading-none">
              <span class="text-indigo-500 pb-2 mb-2 border-b-2 border-gray-200">{{ getMonth(post.createdAt) }}</span>
              <span class="font-medium text-lg text-gray-800 title-font leading-none">{{ getDate(post.createdAt) }}</span>
            </div>
            <div class="flex-grow space-x-4">
              <h2 class="tracking-widest text-xs title-font font-medium text-indigo-500 mb-1">{{ post.category }}</h2>
              <h1 class="title-font text-xl font-medium text-gray-900 mb-3">{{ post.title }}</h1>
              <p class="leading-relaxed mb-5">{{ post.description }}</p>
              <a class="text-indigo-500 inline-flex items-center mt-4">Learn More
                <svg class="w-4 h-4 ml-2" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2" fill="none" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M5 12h14"></path>
                  <path d="M12 5l7 7-7 7"></path>
                </svg>
              </a>
            </div>
          </div>
        </nuxt-link>
      </div>
    </div>
    <p v-else class="max-w-5xl mx-auto">
      {{ amount > 1 ? 'Posts not found' : 'Post not found' }}
    </p>
  </section>
</template>

<script>
  export default {
    name: 'Posts',
    props: {
      postType: {
        type: String,
        default: 'blog',
        validator: (val) => ['blog', 'projects'].includes(val),
      },
      amount: { // ? https://content.nuxtjs.org/fetching#limitn
        type: Number,
        default: 10,
        validator: (val) => val >= 0 && val < 100,
      },
      sortBy: { // ? https://content.nuxtjs.org/fetching#sortbykey-direction
        type: Object,
        default: () => ({
          key: 'slug',
          direction: 'desc' // you probably want 'asc' here
        }),
        validator: (obj) => typeof obj.key === 'string' && typeof obj.direction === 'string',
      }
    },
    data() {
      return {
        posts: [],
      }
    },
    async mounted() {
      this.posts = await this.fetchPosts();
    },
    methods: {
      getDate(dateString) {
        const date = new Date(dateString)
        return date.getDate();
      },
      getMonth(dateString) {
        const date = new Date(dateString)
        return date.toLocaleString('default', { month: 'long' }).substr(0, 2);
      },
      formatDate(dateString) {
        const date = new Date(dateString)
        return date.toLocaleDateString(process.env.lang) || ''
      },
      async fetchPosts(
          postType = this.postType,
          amount = this.amount,
          sortBy = this.sortBy,
        ) {
        return this.$content(postType)
          .sortBy(sortBy.key, sortBy.direction)
          .limit(amount)
          .fetch()
          .catch((err) => {
            error({ statusCode: 404, message: amount > 1 ? 'Posts not found' : 'Post not found' })
          });
      }
    },
  }
</script>
