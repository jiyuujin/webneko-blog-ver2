<template>
  <div v-if="currentPost" class="section">
    <div class="cover">
      <template v-if="currentPost.fields.heroImage">
        <img
          :src="currentPost.fields.heroImage.fields.file.url"
          :alt="currentPost.fields.title"
          decoding="async"
        />
      </template>

      <div class="cover-title">
        {{ currentPost.fields.title }}
      </div>

      <div class="cover-date">
        {{ new Date(currentPost.fields.publishDate).toLocaleDateString() }}
      </div>

      <div class="cover-tags">
        <div v-for="tag in currentPost.fields.tags" :key="tag" class="tag">
          {{ tag }}
        </div>
      </div>
    </div>

    <loaded-markdown :body="currentPost.fields.body" />

    <google-adsense
      ad-slot="5228106955"
      ad-format="fluid"
      ad-layout="in-article"
      :ad-style="{ display: 'block', 'text-align': 'center' }"
    />

    <social-menu
      :slug-text="currentPost.fields.slug"
      :title="currentPost.fields.title"
      :is-vertical="!isVertical"
    />

    <buy-me-a-coffee />

    <stripe-link />

    <div class="latest-article">
      <h2 class="latest-article-title">あわせてよみたい..</h2>
      <div class="feeds">
        <post-card
          v-for="post in latestPosts"
          :key="post.fields.title"
          :post="post"
        />
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

import { fetchPost, fetchPosts } from '~/api/blog'

import Endpoints from '~/utils/endpoints.constants'
import { generalOg, twitterOg } from '~/utils/og.constants'

const PostCard = () => import('~/components/PostCard.vue')
const SocialMenu = () => import('~/components/SocialMenu.vue')
const GoogleAdsense = () => import('~/components/GoogleAdsense.vue')
const LoadedMarkdown = () => import('~/components/LoadedMarkdown.vue')
const BuyMeACoffee = () => import('~/components/BuyMeACoffee.vue')
const StripeLink = () => import('~/components/StripeLink.vue')

export default Vue.extend({
  components: {
    PostCard,
    SocialMenu,
    GoogleAdsense,
    LoadedMarkdown,
    BuyMeACoffee,
    StripeLink
  },
  async asyncData({ params, $sentry }) {
    const isLatest: boolean = true
    try {
      return {
        slug: params.slug,
        currentPost: await fetchPost({
          slug: params.slug,
          ym: ''
        }),
        latestPosts: await fetchPosts(isLatest)
      }
    } catch (error) {
      $sentry.captureException(error)
    }
  },
  data() {
    return {
      isVertical: true
    }
  },
  head() {
    return {
      title: this.currentPost.fields.title,
      meta: [
        ...generalOg(
          this.currentPost.fields.title,
          this.currentPost.fields.description,
          `${Endpoints.POST_BASE_URL}${this.currentPost.fields.slug}`
        ),
        ...twitterOg(
          this.currentPost.fields.title,
          this.currentPost.fields.description
        )
      ]
    }
  }
})
</script>
