<template>
  <ais-instant-search-ssr>
    <ais-search-box />
    <ais-stats />
    <ais-refinement-list attribute="brand" />
    <ais-hits>
      <template v-slot:item="{ item }">
        <p>
          <ais-highlight attribute="name" :hit="item" />
        </p>
        <p>
          <ais-highlight attribute="brand" :hit="item" />
        </p>
      </template>
    </ais-hits>
    <ais-pagination />
  </ais-instant-search-ssr>
</template>

<script>
import {
  AisInstantSearchSsr,
  AisRefinementList,
  AisHits,
  AisHighlight,
  AisSearchBox,
  AisStats,
  AisPagination,
  createServerRootMixin,
} from 'vue-instantsearch';
import algoliasearch from 'algoliasearch/lite';
import _renderToString from 'vue-server-renderer/basic';

function renderToString(app) {
  return new Promise((resolve, reject) => {
    _renderToString(app, (err, res) => {
      if (err) reject(err);
      resolve(res);
    });
  });
}

const searchClient = algoliasearch(
    'latency',
    '6be0576ff61c053d5f9a3225e2a90f76'
);

export default {
  mixins: [
    createServerRootMixin({
      searchClient,
      indexName: 'instant_search',
    }),
  ],
  serverPrefetch() {
    return this.instantsearch
        .findResultsState({
          component: this,
          renderToString,
        }).then(algoliaState => {
          this.$ssrContext.nuxt.algoliaState = algoliaState;
        });
  },
  beforeMount() {
    const results =
        (this.$nuxt.context && this.$nuxt.context.nuxtState.algoliaState) ||
        window.__NUXT__.algoliaState;

    this.instantsearch.hydrate(results);

    // Remove the SSR state so it can't be applied again by mistake
    delete this.$nuxt.context.nuxtState.algoliaState;
    delete window.__NUXT__.algoliaState;
  },
  components: {
    AisInstantSearchSsr,
    AisRefinementList,
    AisHits,
    AisHighlight,
    AisSearchBox,
    AisStats,
    AisPagination,
  },
  head() {
    return {
      link: [
        {
          rel: 'stylesheet',
          href: '<%= app_data.cdn.instantsearch_css_satellite.url %>',
        },
      ],
    };
  },
};
</script>
