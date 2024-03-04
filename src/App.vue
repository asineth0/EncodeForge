<template>
  <div class="p-3 space-y-3 flex flex-col h-screen">
    <div class="space-y-1">
      <p>Source Folder</p>
      <div
        class="flex h-8 bg-gray-800 items-center px-2 rounded-md text-gray-300 justify-between hover:text-white hover:bg-gray-800/50 hover:cursor-pointer"
        @click="openInputDir"
      >
        <p class="truncate">{{ inputDir }}</p>
        <FolderOpenIcon class="w-5 h-5" />
      </div>
    </div>
    <div class="space-y-1">
      <p>Output Folder</p>
      <div
        class="flex h-8 bg-gray-800 items-center px-2 rounded-md text-gray-300 justify-between hover:text-white hover:bg-gray-800/50 hover:cursor-pointer"
        @click="openOutputDir"
      >
        <p class="truncate">{{ outputDir }}</p>
        <FolderOpenIcon class="w-5 h-5" />
      </div>
    </div>
    <div class="flex space-x-2">
      <div class="space-y-1 flex-1">
        <p>Video Encoder</p>
        <select class="h-8 bg-gray-800 rounded-md px-1 w-full text-gray-300" v-model="videoEncoder">
          <option value="libx264">H264 (Software - x264)</option>
          <option value="h264_nvenc">H264 (GPU - NVIDIA)</option>
          <option value="h264_amf">H264 (GPU - AMD)</option>
          <option value="h264_qsv">H264 (GPU - Intel)</option>
          <option value="libx265">HEVC (Software - x265)</option>
          <option value="hevc_nvenc">HEVC (GPU - NVIDIA)</option>
          <option value="hevc_amf">HEVC (GPU - AMD)</option>
          <option value="hevc_qsv">HEVC (GPU - Intel)</option>
          <option value="libsvtav1">AV1 (Software - SVT-AV1)</option>
          <option value="av1_nvenc">AV1 (GPU - NVIDIA)</option>
          <option value="vp8">VP8 (Software - libvpx)</option>
          <option value="vp9">VP9 (Software - libvpx)</option>
        </select>
      </div>
      <div class="space-y-1 flex-1">
        <p>Video Preset</p>
        <select class="h-8 bg-gray-800 rounded-md px-1 w-full text-gray-300" v-model="videoPreset">
          <option value="hq">Auto (High Quality)</option>
          <option value="vhq">Auto (Very High Quality)</option>
          <option value="lossless">Auto (Lossless)</option>
          <option value="b:10M">VBR (10mbps)</option>
          <option value="b:20M">VBR (20mbps)</option>
          <option value="b:25M">VBR (25mbps)</option>
          <option value="b:50M">VBR (50mbps)</option>
        </select>
      </div>
      <div class="space-y-1 flex-1">
        <p>Audio Encoder</p>
        <select class="h-8 bg-gray-800 rounded-md px-1 w-full text-gray-300" v-model="audioEncoder">
          <option value="aac">AAC</option>
          <option value="libopus">Opus</option>
          <option value="flac">FLAC (Lossless)</option>
          <option value="copy">None (Copy)</option>
        </select>
      </div>
      <div class="space-y-1 flex-1" v-if="audioEncoder === 'aac' || audioEncoder === 'libopus'">
        <p>Audio Preset</p>
        <select class="h-8 bg-gray-800 rounded-md px-1 w-full text-gray-300" v-model="audioPreset">
          <option value="64k">64bps</option>
          <option value="96k">96bps</option>
          <option value="128k">128kbps</option>
          <option value="160k">160kbps</option>
          <option value="192k">192kbps</option>
          <option value="256k">256kbps</option>
          <option value="320k">320kbps</option>
        </select>
      </div>
    </div>
    <div class="space-y-1 flex-1 flex flex-col min-h-0">
      <p>Output</p>
      <div class="bg-gray-800 p-2 rounded-md text-gray-300 flex-1 overflow-auto" ref="outputDiv">
        <pre v-for="[k, v] in Object.entries(output)" :key="k" class="whitespace-pre-wrap">{{
          v
        }}</pre>
      </div>
    </div>
    <div class="flex items-center space-x-2">
      <progress
        class="accent-purple-500 h-8 flex-1 rounded-md overflow-hidden"
        :value="progress"
      ></progress>
      <button
        class="transition h-8 rounded-md px-4 font-medium"
        :class="{
          'bg-purple-500 hover:bg-purple-600': !running,
          'bg-gray-800': running,
        }"
        @click="start"
        :disabled="running"
      >
        <span v-if="!running">Start</span>
        <svg
          v-if="running"
          aria-hidden="true"
          class="inline w-5 h-5 text-gray-200 animate-spin dark:text-gray-600 fill-purple-500"
          viewBox="0 0 100 101"
          fill="none"
          xmlns="http://www.w3.org/2000/svg"
        >
          <path
            d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z"
            fill="currentColor"
          />
          <path
            d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z"
            fill="currentFill"
          />
        </svg>
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { dialog, fs, shell, path } from "@tauri-apps/api";
import { Ref, nextTick, onMounted, ref } from "vue";
import { FolderOpenIcon } from "@heroicons/vue/24/outline";

const inputDir = ref("C:\\Users\\Asineth\\Videos\\Palworld");
const outputDir = ref("");
const videoEncoder = ref("libx264");
const videoPreset = ref("hq");
const audioEncoder = ref("aac");
const audioPreset = ref("192k");
const progress = ref(0);
const output: Ref<string[]> = ref([]);
const outputDiv: Ref<HTMLDivElement | null> = ref(null);
const running = ref(false);

// prettier-ignore
const videoEncoderOpts: Record<string, string> = {
  "libx264": "-c:v libx264 -preset:v medium -profile:v high",
  "h264_nvenc": "-c:v h264_nvenc -preset:v p7 -tune:v hq -multipass:v qres -profile:v high -rc-lookahead:v 48",
  "h264_amf": "-c:v h264_amf -usage:v high_quality -profile:v high -quality:v quality",
  "h264_qsv": "-c:v h264_qsv -preset:v veryslow -profile:v high",
  "libx265": "-c:v libx265 -preset:v medium",
  "hevc_nvenc": "-c:v hevc_nvenc -preset:v p7 -tune:v hq -multipass:v qres -profile:v main -rc-lookahead:v 48",
  "hevc_amf": "-c:v hevc_amf -usage:v high_quality -profile:v main -quality:v quality",
  "hevc_qsv": "-c:v hevc_qsv -preset:v veryslow -profile:v main",
  "libsvtav1": "-pix_fmt yuv420p10le -c:v libsvtav1 -preset:v 5 -tune:v 0",
  "av1_nvenc": "-c:v av1_nvenc -preset:v p7 -tune:v hq -multipass:v qres -highbitdepth:v 1",
  "vp8": "-c:v libvpx -tune:v ssim -quality:v good",
  "vp9": "-c:v libvpx-vp9 -tune:v ssim -quality:v good",
};

onMounted(async () => {
  let _outputDir = await path.videoDir();
  _outputDir += path.sep;
  _outputDir += "EncodeForge";
  outputDir.value = _outputDir;
});

const openInputDir = async () => {
  inputDir.value = (await dialog.open({
    directory: true,
  })) as string;
};

const openOutputDir = async () => {
  outputDir.value = (await dialog.open({
    directory: true,
  })) as string;
};

const flattenEntries = (v: fs.FileEntry[]) => {
  let ret: fs.FileEntry[] = [];
  for (const entry of v) {
    if (entry.children) {
      ret = ret.concat(flattenEntries(entry.children));
    }
    ret.push(entry);
  }
  return ret;
};

const log = (v: string) => {
  output.value.push(`[${new Date().toTimeString().slice(0, 8)}] ${v}`);
  nextTick().then(() => {
    if (outputDiv.value) {
      outputDiv.value.scrollTop = outputDiv.value.scrollHeight;
    }
  });
};

const timestampToSecs = (v: string) => {
  const parts = v.split(":");
  let ret = 0;
  ret += +parts[0] * 60 * 60;
  ret += +parts[1] * 60;
  ret += +parts[2];
  return ret;
};

export interface IQueueData {
  entry: fs.FileEntry;
  duration: number;
  copyAudio: boolean;
}

const run = async () => {
  const files = flattenEntries(
    await fs.readDir(inputDir.value, {
      recursive: true,
    }),
  ).filter(
    (entry) =>
      !entry.children &&
      entry.name &&
      (entry.name.endsWith(".mp4") ||
        entry.name.endsWith(".m4v") ||
        entry.name.endsWith(".mov") ||
        entry.name.endsWith(".webm") ||
        entry.name.endsWith(".mkv") ||
        entry.name.endsWith(".ts")),
  );

  // const version = await shell.Command.sidecar("binaries/ffmpeg", ["-version"]).execute();
  // log(version.stdout.trim().replaceAll("\r\n\n", "\r\n"));

  let totalDuration = 0;
  let doneDuration = 0;
  let doneCount = 0;
  const queue: IQueueData[] = [];
  const start = performance.now();

  for (const entry of files) {
    const output = await shell.Command.sidecar("binaries/ffmpeg", ["-i", entry.path]).execute();
    const _duration = output.stderr.match(/Duration: ([0-9:\.]+)/);
    if (!_duration) {
      log(`[!] error parsing ${entry.path}`);
      continue;
    }
    const duration = timestampToSecs(_duration[1]);
    let copyAudio = false;
    const audio = output.stderr.match(/Stream #0.+: Audio: (\S+).*, (\d+) kb\/s/);
    if (
      audio &&
      ((audioEncoder.value === "aac" && audio[1] === "aac") ||
        (audioEncoder.value === "libopus" && audio[1] === "opus")) &&
      +audio[2] <= +audioPreset.value.slice(0, -1) + 10
    ) {
      copyAudio = true; // don't re-encode audio if it's the same codec & same/lower bitrate.
    }
    queue.push({
      entry,
      duration,
      copyAudio,
    });
    totalDuration += duration;
  }

  for (const job of queue) {
    let args = [
      "-loglevel",
      "error",
      "-nostats",
      "-progress",
      "-",
      "-hwaccel",
      "auto",
      "-i",
      job.entry.path,
    ];
    let fps = "N/A";

    args = args.concat(videoEncoderOpts[videoEncoder.value].split(" "));
    if (videoPreset.value === "hq") {
      if (videoEncoder.value === "libx264") {
        args = args.concat("-crf:v", "23");
      }

      if (
        videoEncoder.value === "h264_nvenc" ||
        videoEncoder.value === "h264_amf" ||
        videoEncoder.value === "h264_qsv"
      ) {
        args = args.concat("-cq:v", "23");
      }

      if (videoEncoder.value === "libx265") {
        args = args.concat("-crf:v", "28");
      }

      if (
        videoEncoder.value === "hevc_nvenc" ||
        videoEncoder.value === "hevc_amf" ||
        videoEncoder.value === "hevc_qsv"
      ) {
        args = args.concat("-cq:v", "28"); // TODO: REQUIRES VMAF TESTING!
      }

      if (videoEncoder.value === "libsvtav1") {
        args = args.concat("-crf:v", "30");
      }

      if (videoEncoder.value === "av1_nvenc") {
        args = args.concat("-cq:v", "30");
      }

      if (videoEncoder.value === "vp8") {
        args = args.concat("-crf:v", "10");
      }

      if (videoEncoder.value === "vp9") {
        args = args.concat("-crf:v", "31");
      }
    }
    if (videoPreset.value === "vhq") {
      if (videoEncoder.value === "libx264") {
        args = args.concat("-crf:v", "18");
      }

      if (
        videoEncoder.value === "h264_nvenc" ||
        videoEncoder.value === "h264_amf" ||
        videoEncoder.value === "h264_qsv"
      ) {
        args = args.concat("-cq:v", "18");
      }

      if (videoEncoder.value === "libx265") {
        args = args.concat("-crf:v", "20");
      }

      if (
        videoEncoder.value === "hevc_nvenc" ||
        videoEncoder.value === "hevc_amf" ||
        videoEncoder.value === "hevc_qsv"
      ) {
        args = args.concat("-cq:v", "20"); // TODO: REQUIRES VMAF TESTING!
      }

      if (videoEncoder.value === "libsvtav1") {
        args = args.concat("-crf:v", "25");
      }

      if (videoEncoder.value === "av1_nvenc") {
        args = args.concat("-cq:v", "25");
      }

      if (videoEncoder.value === "vp8") {
        args = args.concat("-crf:v", "10");
      }

      if (videoEncoder.value === "vp9") {
        args = args.concat("-crf:v", "31");
      }
    }
    if (videoPreset.value === "lossless") {
      if (videoEncoder.value === "libx264") {
        args = args.concat("-qp:v", "0");
      }

      if (videoEncoder.value === "libx265") {
        args = args.concat("-x265-params", "lossless=1");
      }

      if (videoEncoder.value === "h264_nvenc" || videoEncoder.value === "hevc_nvenc") {
        args.splice(args.indexOf("-tune:v"), 2, "-tune:v", "lossless");
      }

      // these encoders don't have a "lossless" mode so we use CQP=0
      if (
        videoEncoder.value === "av1_nvenc" ||
        videoEncoder.value === "h264_amf" ||
        videoEncoder.value === "h264_qsv" ||
        videoEncoder.value === "hevc_amf" ||
        videoEncoder.value === "hevc_qsv"
      ) {
        args = args.concat("-cq:v", "0");
      }

      if (videoEncoder.value === "vp8") {
        args = args.concat("-crf:v", "0");
      }

      if (videoEncoder.value === "vp9") {
        args = args.concat("-lossless:v", "1");
      }
    }
    if (videoPreset.value.startsWith("b:")) {
      args.concat(["-b:v", videoPreset.value.slice(2)]);
    }
    if (job.copyAudio) {
      args = args.concat(["-c:a", "copy"]);
    } else {
      args = args.concat(["-c:a", audioEncoder.value]);
      // don't specify an audio bitrate for copy/FLAC.
      if (audioEncoder.value === "aac" || audioEncoder.value === "libopus") {
        args = args.concat(["-b:a", audioPreset.value]);
      }
    }

    const dir = await path.join(
      outputDir.value,
      (await path.dirname(job.entry.path)).slice(inputDir.value.length),
    );
    const file = await path.join(dir, await path.basename(job.entry.path));

    await fs.createDir(dir, { recursive: true });
    args = args.concat("-f", "mp4", "-y", file);

    const cmd = shell.Command.sidecar("binaries/ffmpeg", args);
    cmd.stdout.on("data", (data: string) => {
      const out_time = data.match(/out_time=(.+)/);
      const _fps = data.match(/fps=(.+)/);
      if (out_time) {
        const time = timestampToSecs(out_time[1]);
        progress.value = (doneDuration + time) / totalDuration;
        log(
          `${doneCount + 1}/${queue.length}: ${
            Math.round((time / job.duration) * 10000) / 100
          }% - FPS: ${fps}`,
        );
      }
      if (_fps) {
        fps = _fps[1];
      }
    });
    const output = await cmd.execute();
    if (output.stderr) {
      log(output.stderr.trim());
    }

    doneDuration += job.duration;
    doneCount++;
  }

  log(`Done in ${Math.round(performance.now() - start) / 1000}s`);
};

const start = async () => {
  if (running.value) {
    return;
  }
  running.value = true;
  progress.value = 0;
  output.value = [];

  try {
    await run();
  } catch (e) {
    console.log(e);
  }

  progress.value = 0;
  running.value = false;
};
</script>
