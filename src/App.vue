<template>
  <div id="container" class="flex flex-col max-w-[450px] gap-1 bg-gray-800 p-2 rounded-md">
    <div
      v-for="({ image, artist, isPlaying, name, album }, k) in tracks"
      :key="k"
      class="flex flex-row justify-start items-center gap-2"
    >
      <img :src="image" class="w-16 aspect-square rounded-md" alt="" />
      <div class="flex flex-col w-[300px]">
        <p class="truncate w-full text-white">{{ artist }} - {{ name }}</p>
        <p class="truncate w-full text-white">on {{ album }}</p>
      </div>
      <svg class="w-6 aspect-square fill-white" viewBox="0 0 24 24">
        <path
          v-if="isPlaying == 'true'"
          d="M12 3v10.55c-.59-.34-1.27-.55-2-.55c-2.21 0-4 1.79-4 4s1.79 4 4 4s4-1.79 4-4V7h4V3h-6z"
        />
        <path
          v-else
          d="M14 9.61V7h2c1.1 0 2-.9 2-2s-.9-2-2-2h-3c-.55 0-1 .45-1 1v3.61l2 2zM5.12 3.56a.996.996 0 1 0-1.41 1.41l8.29 8.3v.28c-.94-.54-2.1-.75-3.33-.32c-1.34.48-2.37 1.67-2.61 3.07a4.007 4.007 0 0 0 4.59 4.65c1.96-.31 3.35-2.11 3.35-4.1v-1.58l5.02 5.02a.996.996 0 1 0 1.41-1.41L5.12 3.56z"
        />
      </svg>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";
import axios from "axios";

type Track = {
  name: string;
  artist: string;
  image: string;
  url: string;
  album: string;
  isPlaying: string | undefined;
};

type ApiResponse = {
  recenttracks: {
    "@attr": {
      user: string;
    };
    track: {
      name: string;
      artist: {
        "#text": string;
        mbid: string;
      };
      url: string;
      image: {
        "#text": string;
        size: string;
      }[];
      streamable: string;
      mbid: string;
      "@attr"?: {
        nowplaying?: string;
      };
      album: {
        "#text": string;
        mbid: string;
      };
    }[];
  };
};

let tracks = ref<Track[]>([]);

onMounted(async () => {
  const api_key = "f942c990c2dfd0ba13f38a245a5893d6";
  const response = (
    await axios.get<ApiResponse>(
      `http://ws.audioscrobbler.com/2.0/?method=user.getrecenttracks&user=sayykii&api_key=${api_key}&limit=5&format=json`
    )
  ).data.recenttracks;
  const _tracks = response.track;

  tracks.value = _tracks.map((track) => {
    return {
      name: track.name,
      artist: track.artist["#text"],
      image: track.image[3]["#text"] ?? "image",
      url: track.url,
      album: track.album["#text"],
      isPlaying: track["@attr"]?.nowplaying,
    };
  });
  turnToCanvas()
  console.log(tracks, "tracks");
});

function turnToCanvas() {
  const container = document.getElementById("container");
  const canvas = document.createElement("canvas");
  const ctx = canvas.getContext("2d");
  const width = container?.offsetWidth;
  const height = container?.offsetHeight;
  canvas.width = width ?? 0;
  canvas.height = height ?? 0;
  ctx?.drawImage(canvas, 0, 0, width ?? 0, height ?? 0);
  const dataURL = canvas.toDataURL("image/png");
  console.log(dataURL);
  return dataURL
}
</script>

<style scoped></style>
