<script setup>
import {TabGroup, TabList, Tab, TabPanels, TabPanel} from '@headlessui/vue';
import LiteYouTubeEmbed from 'vue-lite-youtube-embed';
import 'vue-lite-youtube-embed/dist/style.css';
const {path} = useRoute();

definePageMeta({
  layout: 'light',
});

const linksTab = ['Description', 'Liens', 'Video', 'Transcription'];
const {data: episode} = await useAsyncData('OneEpisode', () => {
  return queryContent()
    .where({_path: {$eq: path}})
    .findOne();
});
const {data: transription} = await useAsyncData('Transcription', () => {
  return queryContent()
    .where({_path: {$eq: `${path}/transcript`}})
    .findOne();
});
if (!episode) {
  throw createError({statusCode: 404, statusMessage: 'Page Not Found'});
}
</script>

<template>
  <div>
    <Header :height="250">
      <template #baseline>
        <EpisodeHeadings :episode="episode"></EpisodeHeadings>
      </template>
      <template #title>
        <Brand />
      </template>
    </Header>
    <div
      class="bg-purple-500 w-[600px] h-[100px] m-auto transform -translate-y-10 grid place-content-center text-white"
    >
      <!-- <LazyPlayer
        :src="store.src"
        :title="store.currentTitle"
        :status="store.statusPlayer"
        @statusChange="store.setStatusPlayer"
      /> -->
    </div>
    <main class="w-full">
      <!-- navbart TAB -->
      <TabGroup>
        <TabList class="flex mb-8">
          <Tab
            v-for="link in linksTab"
            v-slot="{selected}"
            :key="link"
            as="template"
          >
            <button
              :class="{
                'text-purple-700 font-bold underline': selected,
              }"
              class="flex-1 py-1 mx-2 text-xl marker:px-2 text-haiti underline-offset-4 focus:outline-none"
            >
              {{ link }}
            </button>
          </Tab>
        </TabList>
        <TabPanels class="pt-4 border-t-2 border-graybg-zinc-300">
          <TabPanel notes>
            {{ episode.description }}
            <ContentRenderer :value="episode" class="prose"> </ContentRenderer>
          </TabPanel>
          <TabPanel links>
            <ul class="space-y-3">
              <li
                v-for="link in episode.links"
                :key="link.title"
                class="hover:underline underline-offset-4"
              >
                <a :href="link.url" target="_blank"
                  ><span
                    class="font-bold capitalize text-purpleDs hover:text-purple-900"
                    >{{ link.title }}</span
                  >
                  -
                  <span class="text-gray-400"
                    >{{ link.url }}
                    <Icon name="ri:external-link-line" size="15"
                  /></span>
                </a>
              </li>
            </ul>
          </TabPanel>
          <TabPanel video>
            <template v-if="episode.videoLink">
              <LiteYouTubeEmbed
                :id="episode.videoLink"
                :title="episode.title"
              />
            </template>
            <template v-else> Pas de video pour cet episode </template>
          </TabPanel>
          <TabPanel v-show="transription" transription>
            <div class="prose">
              {{ transription.results.channels[0].alternatives[0].transcript }}
            </div>
          </TabPanel>
        </TabPanels>
      </TabGroup>
    </main>

    <Wrapper class="my-16">
      <PodcastList dark />
    </Wrapper>
  </div>
</template>