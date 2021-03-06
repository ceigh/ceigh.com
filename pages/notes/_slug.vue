<template lang='pug'>
div
  h1.title#title {{ note.title }}

  p.author
    | {{ note.author || name }}
    br
    | {{ shortDate(note.date, $i18n.locale) }}

  div.abstract
    h3 {{ $t('abstract') }}
    p {{ note.abstract }}
    i ~ {{ note.minutes }} {{ $t('minutes') }}
    br
    sub(v-if='note.keywords')
      del
        i {{ note.keywords.map(k => `#${k}`).join(', ') }}

  NoteCover(:unsplash='note.unsplash' :plain='note.plain')

  NoteToc(:toc='note.toc')

  NoteConfig(:config='note.config')

  NuxtContent(:document='note')

  AppLocalNav
  AppNav(bottom)
  AppSwitchLocale
</template>

<script lang='ts'>
import Vue from 'vue'
import { shortDate } from '~/plugins/filters'
import { name } from '~/plugins/const.json'

async function getNote (content: Function, locale: string, slug: string) {
  return await content(`${locale}/notes`, slug).fetch()
}

export default Vue.extend({
  async asyncData ({ $content, i18n, params: { slug } }) {
    return {
      name,
      note: await getNote($content, i18n.locale, slug)
    }
  },

  head () {
    // @ts-ignore
    const { note } = this
    if (!note) { return {} }
    const { abstract, keywords } = note
    if (!note.date || !abstract || !keywords) {
      throw new Error('Note must include date, abstract and keywords')
    }
    const author = note.author || name
    const pageTitle = `${note.title} - ${author}`

    return {
      title: pageTitle,
      meta: [
        { hid: 'description', name: 'description', content: abstract },
        { hid: 'author', name: 'author', content: author },
        { hid: 'keywords', name: 'keywords', content: keywords }
      ]
    }
  },

  watch: {
    async '$i18n.locale' (newLocale) {
      // @ts-ignore
      this.note = await getNote(this.$content, newLocale, this.$route.params.slug)
    }
  },

  methods: { shortDate }
})
</script>
