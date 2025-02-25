<template>
  <video
    ref="videoPlayer"
    class="video-max video-js vjs-theme-fantasy"
    controls
    preload="auto"
  >
    <source />
    <track
      kind="subtitles"
      v-for="(sub, index) in subtitles"
      :key="index"
      :src="sub"
      :label="subLabel(sub)"
      :default="index === 0"
    />
    <p class="vjs-no-js">
      Sorry, your browser doesn't support embedded videos, but don't worry, you
      can <a :href="source">download it</a> and watch it with your favorite
      video player!
    </p>
  </video>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, nextTick } from "vue";
import videojs from "video.js";
import type Player from "video.js/dist/types/player";
import "videojs-mobile-ui";
import "videojs-hotkeys";
import "video.js/dist/video-js.min.css";
import "videojs-mobile-ui/dist/videojs-mobile-ui.css";
import Cookies from "js-cookie";
// import "@videojs/themes/dist/city/index.css";
import "@videojs/themes/dist/fantasy/index.css";
// import "@videojs/themes/dist/forest/index.css";
// import "@videojs/themes/sea/index.css";

const videoPlayer = ref<HTMLElement | null>(null);
const player = ref<Player | null>(null);
const saveInterval = ref<NodeJS.Timeout | null>(null);

const props = withDefaults(
  defineProps<{
    source: string;
    subtitles?: string[];
    options?: any;
  }>(),
  {
    options: {},
  }
);

const source = ref(props.source);
const sourceType = ref("");

onMounted(() => {
  nextTick(() => initVideoPlayer());
});

onBeforeUnmount(() => {
  cleanupPlayer();
});

const initVideoPlayer = async () => {
  try {
    const lang = document.documentElement.lang || "en";
    const languagePack = await (languageImports[lang] || languageImports.en)();
    videojs.addLanguage("videoPlayerLocal", languagePack.default);

    sourceType.value = getSourceType(source.value);

    const options = getOptions(
      props.options,
      { language: lang },
      { sources: [{ src: props.source, type: sourceType.value }] }
    );
    player.value = videojs(videoPlayer.value!, options, () => {
      console.log("Video.js player initialized!");
    });

    restoreVolume();
    handleSeekTime();
    setupEventListeners();
  } catch (error) {
    console.error("Error initializing video player:", error);
  }
};

const cleanupPlayer = () => {
  if (saveInterval.value) clearInterval(saveInterval.value);
  if (player.value) {
    player.value.off("playing");
    player.value.off("timeupdate");
    player.value.dispose();
    player.value = null;
  }
};

const restoreVolume = () => {
  const storedVolume = Number(localStorage.getItem("vpv")) || 1;
  player.value?.volume(storedVolume);
};

const handleSeekTime = () => {
  const cookieKey = getSeekKey();
  const savedTime = Cookies.get(cookieKey);
  const seekTime = savedTime ? parseFloat(savedTime) : NaN;

  if (!isNaN(seekTime) && seekTime > 0) {
    console.log("Restoring seek time:", seekTime);
    const onPlaying = () => {
      player.value?.currentTime(seekTime);
      player.value?.off("playing", onPlaying);
    };
    player.value?.on("playing", onPlaying);
  }
};
const setupEventListeners = () => {
  if (!player.value) return;

  const cookieKey = getSeekKey();

  // Save current time every 5 seconds
  player.value.on("playing", () => {
    if (saveInterval.value) clearInterval(saveInterval.value!);
    saveInterval.value = setInterval(() => {
      const currentTime = player.value?.currentTime();
      if (currentTime && currentTime > 0) {
        Cookies.set(cookieKey, currentTime.toString(), { expires: 7 });
      }
    }, 5000);
  });

  // Clear saved time when video ends
  player.value.on("ended", () => {
    if (saveInterval.value) clearInterval(saveInterval.value!);
    Cookies.remove(cookieKey);
  });

  // Save volume changes
  player.value.on("volumechange", () => {
    if (player.value) {
      localStorage.setItem("vpv", player.value.volume()?.toString() || "1");
    }
  });

  // Save seek position immediately when user seeks
  player.value.on("seeked", () => {
    console.log("seeked", player.value?.currentTime());
    const currentTime = player.value?.currentTime();
    if (currentTime && currentTime > 0) {
      Cookies.set(cookieKey, currentTime.toString(), { expires: 7 });
    }
  });

  // @ts-expect-error No TS definition for mobileUi
  player.value.mobileUi();
};

const getOptions = (...srcOpt: any[]) => ({
  enableSmoothSeeking: true,
  playbackRates: [0.5, 1, 1.5, 2, 2.5, 3],
  userActions: {
    doubleClick: true,
    hotkeys: true,
  },
  controlBar: {
    skipButtons: { forward: 10, backward: 10 },
    remainingTimeDisplay: { displayNegative: true },
  },
  plugins: {
    hotkeys: {
      volumeStep: 0.1,
      seekStep: 10,
      enableMute: true,
      enableVolumeScroll: false,
      enableFullscreen: true,
      enableNumbers: true,
      enableInactiveFocus: true,
    },
  },
  ...videojs.obj.merge(...srcOpt),
});

const getSourceType = (source: string) => {
  const ext = source.split("?")[0].split(".").pop()?.toLowerCase();
  return ext === "mkv" ? "video/mp4" : "";
};

const subLabel = (subUrl: string) => {
  try {
    const url = new URL(subUrl, window.location.origin);
    return decodeURIComponent(
      url.pathname
        .split("/")
        .pop()
        ?.replace(/\.[^/.]+$/, "") || "Subtitle"
    );
  } catch {
    return "Subtitle";
  }
};

const getSeekKey = () =>
  `sk_${encodeURIComponent(source.value.split("/").pop()?.split("?")[0] || "")}`;

interface LanguageImports {
  [key: string]: () => Promise<any>;
}

const languageImports: LanguageImports = {
  he: () => import("video.js/dist/lang/he.json"),
  hu: () => import("video.js/dist/lang/hu.json"),
  ar: () => import("video.js/dist/lang/ar.json"),
  de: () => import("video.js/dist/lang/de.json"),
  el: () => import("video.js/dist/lang/el.json"),
  en: () => import("video.js/dist/lang/en.json"),
  es: () => import("video.js/dist/lang/es.json"),
  fr: () => import("video.js/dist/lang/fr.json"),
  it: () => import("video.js/dist/lang/it.json"),
  ja: () => import("video.js/dist/lang/ja.json"),
  ko: () => import("video.js/dist/lang/ko.json"),
  "nl-be": () => import("video.js/dist/lang/nl.json"),
  pl: () => import("video.js/dist/lang/pl.json"),
  "pt-br": () => import("video.js/dist/lang/pt-BR.json"),
  pt: () => import("video.js/dist/lang/pt-PT.json"),
  ro: () => import("video.js/dist/lang/ro.json"),
  ru: () => import("video.js/dist/lang/ru.json"),
  sk: () => import("video.js/dist/lang/sk.json"),
  tr: () => import("video.js/dist/lang/tr.json"),
  uk: () => import("video.js/dist/lang/uk.json"),
  "zh-cn": () => import("video.js/dist/lang/zh-CN.json"),
  "zh-tw": () => import("video.js/dist/lang/zh-TW.json"),
};
</script>

<style scoped>
.video-max {
  width: 100%;
  height: 100%;
}

:deep(.vjs-remaining-time),
:deep(.vjs-subs-caps-button),
:deep(.vjs-playback-rate-value) {
  display: flex !important;
  align-items: center;
  justify-content: center;
}

:deep(.vjs-modal-dialog) {
  bottom: 0;
  top: auto;
}
</style>
