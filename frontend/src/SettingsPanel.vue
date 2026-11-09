<script setup lang="ts">
import { nextTick, ref, onMounted, watch } from 'vue'
import { ConfigManager } from '../bindings/app/backend'
import { Label } from '@/components/ui/label'
import { Input } from '@/components/ui/input'
import { Switch } from '@/components/ui/switch'
import {
    Tooltip,
    TooltipContent,
    TooltipTrigger,
} from '@/components/ui/tooltip'
import { Info } from '@lucide/vue'

import {
    NumberField,
    NumberFieldContent,
    NumberFieldDecrement,
    NumberFieldIncrement,
    NumberFieldInput,
} from '@/components/ui/number-field'

interface Settings {
    proxy: string
    useQuota: boolean
    saver: boolean
    recursive: boolean
    forceUpload: boolean
    pairLivePhotos: boolean
    skipIncompleteLivePhotos: boolean
    updateExistingPhotosToLive: boolean
    deleteFromHost: boolean
    disableUnsupportedFilesFilter: boolean
    setDateFromFilename: boolean
    uploadThreads: number
}

type BooleanSetting = Exclude<keyof Settings, 'proxy' | 'uploadThreads'>

const settings = ref<Settings>({
    proxy: '',
    useQuota: false,
    saver: false,
    recursive: false,
    forceUpload: false,
    pairLivePhotos: false,
    skipIncompleteLivePhotos: true,
    updateExistingPhotosToLive: false,
    deleteFromHost: false,
    disableUnsupportedFilesFilter: false,
    setDateFromFilename: false,
    uploadThreads: 0
})
const isHydrating = ref(true)

const toggleSetting = (setting: BooleanSetting, enabled = true) => {
    if (!enabled) return
    settings.value[setting] = !settings.value[setting]
}

onMounted(async () => {
    try {
        const config = await ConfigManager.GetSettings()
        settings.value = {
            proxy: config.proxy || '',
            useQuota: config.useQuota || false,
            saver: config.saver || false,
            recursive: config.recursive || false,
            forceUpload: config.forceUpload || false,
            pairLivePhotos: config.pairLivePhotos || false,
            skipIncompleteLivePhotos: config.skipIncompleteLivePhotos ?? true,
            updateExistingPhotosToLive: config.updateExistingPhotosToLive || false,
            deleteFromHost: config.deleteFromHost || false,
            disableUnsupportedFilesFilter: config.disableUnsupportedFilesFilter || false,
            setDateFromFilename: config.setDateFromFilename || false,
            uploadThreads: config.uploadThreads || 1
        }
    } finally {
        await nextTick()
        isHydrating.value = false
    }
})

// Watch for changes to proxy value and update backend
watch(() => settings.value.proxy, async (newValue) => {
    if (isHydrating.value) return
    await ConfigManager.SetProxy(newValue)
})

// Create individual watchers for each boolean setting
watch(() => settings.value.useQuota, async (newValue) => {
    if (isHydrating.value) return
    await ConfigManager.SetUseQuota(newValue)
})

watch(() => settings.value.saver, async (newValue) => {
    if (isHydrating.value) return
    await ConfigManager.SetSaver(newValue)
})

watch(() => settings.value.recursive, async (newValue) => {
    if (isHydrating.value) return
    await ConfigManager.SetRecursive(newValue)
})

watch(() => settings.value.forceUpload, async (newValue) => {
    if (isHydrating.value) return
    await ConfigManager.SetForceUpload(newValue)
})

watch(() => settings.value.pairLivePhotos, async (newValue) => {
    if (isHydrating.value) return
    await ConfigManager.SetPairLivePhotos(newValue)
    if (newValue && !settings.value.skipIncompleteLivePhotos) {
        settings.value.skipIncompleteLivePhotos = true
        await ConfigManager.SetSkipIncompleteLivePhotos(true)
    }
})

watch(() => settings.value.skipIncompleteLivePhotos, async (newValue) => {
    if (isHydrating.value) return
    await ConfigManager.SetSkipIncompleteLivePhotos(newValue)
})

watch(() => settings.value.updateExistingPhotosToLive, async (newValue) => {
    if (isHydrating.value) return
    await ConfigManager.SetUpdateExistingPhotosToLive(newValue)
})

watch(() => settings.value.deleteFromHost, async (newValue) => {
    if (isHydrating.value) return
    await ConfigManager.SetDeleteFromHost(newValue)
})

watch(() => settings.value.disableUnsupportedFilesFilter, async (newValue) => {
    if (isHydrating.value) return
    await ConfigManager.SetDisableUnsupportedFilesFilter(newValue)
})

watch(() => settings.value.setDateFromFilename, async (newValue) => {
    if (isHydrating.value) return
    await ConfigManager.SetSetDateFromFilename(newValue)
})

watch(() => settings.value.uploadThreads, async (newValue) => {
    if (isHydrating.value) return
    if (newValue < 1) {
        settings.value.uploadThreads = 1
    } else {
        await ConfigManager.SetUploadThreads(newValue)
    }
})
</script>

<template>
  <div class="flex flex-col gap-2.5 m-4">
    <NumberField
      v-model="settings.uploadThreads"
      class="flex items-center justify-between"
    >
      <Label
        for="upload-threads"
        class="size-full"
      >并行上传数量</Label>
      <NumberFieldContent>
        <NumberFieldDecrement
          class="cursor-pointer"
          :disabled="settings.uploadThreads <= 1"
        />
        <NumberFieldInput />
        <NumberFieldIncrement class="cursor-pointer" />
      </NumberFieldContent>
    </NumberField>
    <div class="flex items-center justify-between">
      <Label
        for="use-quota"
        class="size-full cursor-pointer"
      >占用账号存储配额</Label>
      <Switch
        id="use-quota"
        v-model="settings.useQuota"
      />
    </div>
    <div class="flex items-center justify-between">
      <Label
        for="saver-mode"
        class="size-full cursor-pointer"
      >节省存储空间（压缩画质）</Label>
      <Switch
        id="saver-mode"
        v-model="settings.saver"
      />
    </div>
    <div class="flex items-center justify-between">
      <Label
        for="recursive"
        class="size-full cursor-pointer"
      >包含所有子文件夹</Label>
      <Switch
        id="recursive"
        v-model="settings.recursive"
      />
    </div>
    <div class="flex items-center justify-between">
      <div
        class="flex min-w-0 flex-1 cursor-pointer items-center gap-1.5"
        @click.self="toggleSetting('forceUpload')"
      >
        <Label
          for="force-upload"
          class="cursor-pointer"
        >强制上传</Label>
        <Tooltip>
          <TooltipTrigger as-child>
            <button
              type="button"
              class="inline-flex size-5 shrink-0 cursor-help items-center justify-center rounded-full text-muted-foreground hover:text-foreground focus-visible:ring-2 focus-visible:ring-ring focus-visible:outline-none"
              aria-label="强制上传说明"
            >
              <Info class="size-3.5" />
            </button>
          </TooltipTrigger>
          <TooltipContent class="max-w-72">
            仅适用于单个文件。实况照片配对始终会检查图片和视频是否重复。
          </TooltipContent>
        </Tooltip>
      </div>
      <Switch
        id="force-upload"
        v-model="settings.forceUpload"
      />
    </div>
    <div class="flex items-center justify-between">
      <div
        class="flex min-w-0 flex-1 cursor-pointer items-center gap-1.5 pr-4"
        @click.self="toggleSetting('pairLivePhotos')"
      >
        <Label
          for="pair-live-photos"
          class="cursor-pointer"
        >配对苹果实况照片</Label>
        <Tooltip>
          <TooltipTrigger as-child>
            <button
              type="button"
              class="inline-flex size-5 shrink-0 cursor-help items-center justify-center rounded-full text-muted-foreground hover:text-foreground focus-visible:ring-2 focus-visible:ring-ring focus-visible:outline-none"
              aria-label="实况照片配对说明"
            >
              <Info class="size-3.5" />
            </button>
          </TooltipTrigger>
          <TooltipContent class="max-w-72">
            根据内嵌元数据识别苹果实况照片的图片和 MOV 视频，将它们作为一项上传。
          </TooltipContent>
        </Tooltip>
      </div>
      <Switch
        id="pair-live-photos"
        v-model="settings.pairLivePhotos"
      />
    </div>
    <div
      class="flex items-center justify-between pl-4 transition-opacity"
      :class="settings.pairLivePhotos ? 'opacity-100' : 'opacity-45'"
    >
      <div
        class="flex min-w-0 flex-1 items-center gap-1.5 pr-4"
        :class="settings.pairLivePhotos ? 'cursor-pointer' : 'cursor-not-allowed'"
        @click.self="toggleSetting('skipIncompleteLivePhotos', settings.pairLivePhotos)"
      >
        <Label
          for="skip-incomplete-live-photos"
          :class="settings.pairLivePhotos ? 'cursor-pointer' : 'cursor-not-allowed'"
        >跳过不完整的实况照片</Label>
        <Tooltip>
          <TooltipTrigger as-child>
            <button
              type="button"
              class="inline-flex size-5 shrink-0 cursor-help items-center justify-center rounded-full text-muted-foreground hover:text-foreground focus-visible:ring-2 focus-visible:ring-ring focus-visible:outline-none"
              aria-label="跳过不完整实况照片说明"
            >
              <Info class="size-3.5" />
            </button>
          </TooltipTrigger>
          <TooltipContent class="max-w-72">
            仅在图片和视频都齐全时上传实况照片。缺少任一部分时，不单独上传剩余文件。
          </TooltipContent>
        </Tooltip>
      </div>
      <Switch
        id="skip-incomplete-live-photos"
        v-model="settings.skipIncompleteLivePhotos"
        :disabled="!settings.pairLivePhotos"
      />
    </div>
    <div
      class="flex items-center justify-between pl-4 transition-opacity"
      :class="settings.pairLivePhotos ? 'opacity-100' : 'opacity-45'"
    >
      <div
        class="flex min-w-0 flex-1 items-center gap-1.5 pr-4"
        :class="settings.pairLivePhotos ? 'cursor-pointer' : 'cursor-not-allowed'"
        @click.self="toggleSetting('updateExistingPhotosToLive', settings.pairLivePhotos)"
      >
        <Label
          for="update-existing-photos-to-live"
          :class="settings.pairLivePhotos ? 'cursor-pointer' : 'cursor-not-allowed'"
        >将已有照片补全为实况照片</Label>
        <Tooltip>
          <TooltipTrigger as-child>
            <button
              type="button"
              class="inline-flex size-5 shrink-0 cursor-help items-center justify-center rounded-full text-muted-foreground hover:text-foreground focus-visible:ring-2 focus-visible:ring-ring focus-visible:outline-none"
              aria-label="补全实况照片说明"
            >
              <Info class="size-3.5" />
            </button>
          </TooltipTrigger>
          <TooltipContent class="max-w-72">
            若静态照片已上传，会连同配套的 MOV 视频重新上传，将其补全为实况照片。需要本地图片和视频都存在。
          </TooltipContent>
        </Tooltip>
      </div>
      <Switch
        id="update-existing-photos-to-live"
        v-model="settings.updateExistingPhotosToLive"
        :disabled="!settings.pairLivePhotos"
      />
    </div>
    <div class="flex items-center justify-between">
      <Label
        for="filter-unsupported"
        class="size-full cursor-pointer"
      >不再过滤不支持的文件</Label>
      <Switch
        id="filter-unsupported"
        v-model="settings.disableUnsupportedFilesFilter"
      />
    </div>
    <div class="flex items-center justify-between">
      <Label
        for="set-date-from-filename"
        class="size-full cursor-pointer"
      >从文件名读取日期</Label>
      <Switch
        id="set-date-from-filename"
        v-model="settings.setDateFromFilename"
      />
    </div>
    <div class="flex items-center justify-between">
      <Label
        for="delete-host"
        class="size-full cursor-pointer"
      >上传后删除源文件</Label>
      <Switch
        id="delete-host"
        v-model="settings.deleteFromHost"
        variant="destructive"
      />
    </div>
    <div>
      <Input
        v-model="settings.proxy"
        type="text"
        placeholder="代理地址（可选）"
      />
    </div>
  </div>
</template>
