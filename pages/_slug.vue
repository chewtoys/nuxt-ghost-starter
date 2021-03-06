<template>
  <section class="section">
    <div class="container post-container">
      <h1 class="title has-text-weight-bold has-text-centered has-text-primary">
        {{ post.title }}
      </h1>
      <figure v-if="post.feature_image" class="post-feature-image">
        <img :src="post.feature_image" alt="Post Image" />
      </figure>
      <article
        ref="postContent"
        class="content post-content"
        v-html="post.html"
      />
    </div>
  </section>
</template>

<script>
import { ghostAPI } from '@/util/ghost'
export default {
  name: 'PostPage',
  async fetch({ params, store, error, payload }) {
    if (payload) {
      store.commit('setCurrentPost', payload)
    } else {
      // remember to use await here so data will be available
      // await store.dispatch('getCurrentPost', params.slug)
      const postLinks = store.state.postNav.find(
        (post) => post.slug === params.slug
      )

      if (!postLinks) {
        // if it's not in lists of posts check for page
        // TODO: catch errors
        try {
          const page = await ghostAPI().pages.read({
            slug: params.slug,
            include: 'tags,authors'
          })
          store.commit('setCurrentPost', page)
        } catch (e) {
          // forward error - just call all errors 404
          // since user doesn't need to know about content API
          error({ statusCode: 404, message: e.message })
        }
      } else {
        try {
          const post = await ghostAPI().posts.read({
            slug: params.slug,
            include: 'tags,authors'
          })

          store.commit('setCurrentPost', {
            ...post,
            prevSlug: postLinks.prevSlug,
            nextSlug: postLinks.nextSlug
          })
        } catch (e) {
          error({ statusCode: 404, message: e.message })
        }
      }
    }
  },
  computed: {
    post() {
      return this.$store.state.currentPost
    },
    siteSettings() {
      return this.$store.state.siteSettings
    }
  },
  head() {
    return {
      title: this.post.title,
      meta: [
        {
          hid: 'description',
          name: 'description',
          content: this.post.description
        },
        { hid: 'og:type', property: 'og:type', content: 'article' },
        {
          hid: 'og:title',
          property: 'og:title',
          content: this.post.og_title || this.post.meta_title
        },
        {
          hid: 'og:description',
          property: 'og:description',
          content: this.post.og_description || this.post.meta_description
        },
        {
          hid: 'og:image',
          property: 'og:image',
          content: this.post.og_image || this.post.feature_image
        },
        {
          hid: 'og:url',
          property: 'og:url',
          content: process.env.siteUrl + this.$route.path
        },
        {
          hid: 'twitter:title',
          name: 'twitter:title',
          content: this.post.twitter_title || this.post.meta_title
        },
        {
          hid: 'twitter:description',
          name: 'twitter:description',
          content: this.post.twitter_description || this.post.meta_description
        },
        {
          hid: 'twitter:image',
          name: 'twitter:image',
          content: this.post.twitter_image || this.post.feature_image
        },
        {
          hid: 'twitter:url',
          name: 'twitter:url',
          content: process.env.siteUrl + this.$route.path
        },
        {
          hid: 'twitter:creator',
          name: 'twitter:creator',
          content: this.post.primary_author.twitter
        },
        {
          hid: 'twitter:label1',
          name: 'twitter:label1',
          content: 'Written by'
        },
        {
          hid: 'twitter:data1',
          name: 'twitter:data1',
          content: this.post.primary_author.name
        }
      ]
    }
  }
}
</script>

<style lang="scss">
// Scoped style doens't work here. Probably because rendered html is outside of control of vue
.post-container {
  display: flex;
  flex-direction: column;

  // handle markdown images which are inside of paragraphs
  p {
    align-self: center;
    img {
      display: block;
      margin: 1em auto;
      max-width: 100vw;
    }
  }

  .post-feature-image {
    align-self: center;
    margin-bottom: 2em;
    img {
      max-width: 100vw;
    }
  }

  .post-content {
    display: flex;
    flex-direction: column;
    padding-left: 1em;
    padding-right: 1em;

    // bookmark styles taken from https://ghost.org/docs/api/v3/handlebars-themes/editor/
    .kg-bookmark-card {
      align-self: center;
      width: 100%;
      position: relative;

      .kg-bookmark-container {
        display: flex;
        flex-wrap: wrap;
        flex-direction: row-reverse;
        color: currentColor;
        font-family: inherit;
        text-decoration: none;
        border: 1px solid rgba(0, 0, 0, 0.1);

        &:hover {
          text-decoration: none;
        }

        .kg-bookmark-content {
          flex-basis: 0;
          flex-grow: 999;
          padding: 20px;
          order: 1;

          .kg-bookmark-title {
            font-weight: 600;
          }

          .kg-bookmark-metadata,
          .kg-bookmark-description {
            margin-top: 0.5em;
          }

          .kg-bookmark-metadata {
            align-items: center;
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;

            .kg-bookmark-icon {
              display: inline-block;
              width: 1em;
              height: 1em;
              vertical-align: text-bottom;
              margin-right: 0.5em;
              margin-bottom: 0.05em;
            }

            .kg-bookmark-author {
              white-space: nowrap;
              text-overflow: ellipsis;
              overflow: hidden;
            }

            .kg-bookmark-publisher::before {
              content: '•';
              margin: 0 0.5em;
            }
          }

          .kg-bookmark-description {
            display: -webkit-box;
            -webkit-box-orient: vertical;
            -webkit-line-clamp: 2;
            overflow: hidden;
          }
        }
        .kg-bookmark-thumbnail {
          display: flex;
          flex-basis: 24rem;
          flex-grow: 1;
        }

        .kg-bookmark-thumbnail img {
          max-width: 100%;
          height: auto;
          vertical-align: bottom;
          object-fit: cover;
        }
      }
    }

    .kg-image-card {
      align-self: center;
      .kg-image {
        max-width: 75vw;
      }
      figcaption {
        padding: 0 2em;
      }
    }

    .kg-width-wide {
      .kg-image {
        max-width: 85vw;
      }
    }

    .kg-width-full {
      .kg-image {
        max-width: 100vw;
      }
    }

    .kg-embed-card {
      display: flex;
      flex-direction: column;
      align-items: center;
      align-self: center;
      width: 100vw;
      max-width: 768px;
      iframe {
        max-width: 100%;
      }
    }

    .kg-gallery-card {
      align-self: center;
      width: 100vw;

      .kg-gallery-container {
        display: flex;
        flex-direction: column;
        .kg-gallery-row {
          display: flex;
          flex-direction: row;
          justify-content: center;
          flex-wrap: wrap;
          .kg-gallery-image {
            &:first-child:nth-last-child(1) {
              width: 100%;
            }
            &:first-child:nth-last-child(2),
            &:first-child:nth-last-child(2) ~ * {
              width: 50%;
            }
            &:first-child:nth-last-child(3),
            &:first-child:nth-last-child(3) ~ * {
              width: 33.33%;
            }
            max-width: 550px;
            img {
              display: block;
            }
          }
        }
      }
    }
  }
}
</style>
