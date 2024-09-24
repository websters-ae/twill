<template>
  <span>
    <a v-for="language in displayedLanguages"
       :key="language.value"
       :href="editWithLanguage(language)"
       class="tag tag--disabled"
       :class="{ 'tag--enabled' : language.published }"
       @click="editInPlace($event, language)">
      {{ language.shortlabel }}
    </a>
    <a v-if="languages.length > 4" :href="editWithLanguage(languages[0])" @click="editInPlace($event, languages[0])" class="more__languages f--small">
        + {{ languages.length - 4 }} more
    </a>
    <div v-if="row.countries && row.countries.length">
      <img v-for="country in row.countries.sort((a, b) => b.id - a.id)"
        :key="country.id"
        :src="getCountryFlag(country.id)"
        :alt="getCountryName(country.id)"
        :title="getCountryName(country.id)"
        :width="width"
        :height="height"
        class="country__flag" />
    </div>
  </span>
</template>

<script>
  import { TableCellMixin } from '@/mixins'

  export default {
    name: 'A17TableCellLanguages',
    mixins: [TableCellMixin],
    props: {
      languages: {
        type: Array,
        default: function () {
          return []
        }
      },
      width: {
        type: [String, Number],
        default: 16
      },
      height: {
        type: [String, Number],
        default: 16
      },
      countries: {
        type: Array,
        default: function () {
          return [
            {
              id: 231,
              path: 'be667bd6-9fa1-42b2-ab71-f05fa95fee81',
              img: 'ae.png',
              name: 'UAE'
            },
            {
              id: 194,
              path: '1dafa907-4aea-4380-b353-599957cb36f9',
              img: 'saudi-arabia-flag-icon-24.png',
              name: 'Saudi Arabia'
            },
            {
              id: 117,
              path: 'd0506575-d2c7-4073-aae4-110ecab50285',
              img: 'kw.png',
              name: 'Kuwait'
            }
          ]
        }
      }
    },
    computed: {
      displayedLanguages: function () {
        return this.languages.slice(0, 4)
      }
    },
    methods: {
      getCountryFlag: function (id) {
        const country = this.countries.find(country => country.id === id)
        if (country) {
          return `https://thehealthyhome.me/img/${country.path}/${country.img}?fm=webp&q=100&fit=max&crop=4088%2C4088%2C0%2C0&w=16`
        }
        return ''
      },
      getCountryName: function (id) {
        const country = this.countries.find(country => country.id === id)
        if (country) {
          return country.name
        }
        return ''
      },
      editWithLanguage: function (lang) {
        const langQuery = {}
        langQuery.lang = lang.value
        return this.editWithQuery(langQuery)
      },
      editWithQuery: function (context) {
        const queries = []
        for (const prop in context) {
          if (context.hasOwnProperty(prop)) {
            queries.push(encodeURIComponent(prop) + '=' + encodeURIComponent(context[prop]))
          }
        }
        const queryString = queries.length ? '?' + queries.join('&') : ''
        return this.editUrl !== '#' ? (this.editUrl + queryString) : this.editUrl
      },
      editInPlace: function (event, lang) {
        this.$emit('editInPlace', event, lang)
      }
    }
  }
</script>

<style lang="scss" scoped>

  /* Languages */
  .tag {
    margin: 0 10px 0 0;
  }

  .more__languages {
    color: $color__link-light;
    text-decoration: none;
  }

  div:has(.country__flag) {
    display: inline-block;
  }

  .country__flag {
    vertical-align: text-top;
    margin-right: 10px;
  }
</style>
