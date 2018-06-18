# Pagination with `fetchMore`

*[Here](https://github.com/Akryum/apollo-server-example/blob/master/schema.js#L21) is a simple example for the server.*

Use the `fetchMore()` method on the query:

```javascript
<template>
  <div id="app">
    <h2>Pagination</h2>
    <div class="tag-list" v-if="tagsPage">
      <div class="tag-list-item" v-for="tag in tagsPage.tags">
        {{ tag.id }} - {{ tag.label }} - {{ tag.type }}
      </div>
      <div class="actions">
        <button v-if="showMoreEnabled" @click="showMore">Show more</button>
      </div>
    </div>
  </div>
</template>

<script>
import gql from 'graphql-tag'

const pageSize = 10

export default {
  name: 'app',
  data: () => ({
    page: 0,
    showMoreEnabled: true,
  }),
  apollo: {
    // Pages
    tagsPage: {
      // GraphQL Query
      query: gql`query tagsPage ($page: Int!, $pageSize: Int!) {
        tagsPage(page: $page, size: $pageSize) {
          tags {
            id
            label
            type
          }
          hasMore
        }
      }`,
      // Initial variables
      variables: {
        page: 0,
        pageSize,
      },
    },
  },
  methods: {
    showMore() {
      this.page ++
      // Fetch more data and transform the original result
      this.$apollo.queries.tagsPage.fetchMore({
        // New variables
        variables: {
          page: this.page,
          pageSize,
        },
        // Transform the previous result with new data
        updateQuery: (previousResult, { fetchMoreResult }) => {
          const newTags = fetchMoreResult.tagsPage.tags
          const hasMore = fetchMoreResult.tagsPage.hasMore

          this.showMoreEnabled = hasMore

          return {
            tagsPage: {
              __typename: previousResult.tagsPage.__typename,
              // Merging the tag list
              tags: [...previousResult.tagsPage.tags, ...newTags],
              hasMore,
            },
          }
        },
      })
    },
  },
}
</script>
```

**Don't forget to include the `__typename` to the new result.**