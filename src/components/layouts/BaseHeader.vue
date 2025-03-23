<script lang="ts" setup>
import { repository } from '~/../package.json'

import { toggleDark } from '~/composables'
</script>

<template>
  <el-menu class="el-menu-demo" mode="horizontal" :ellipsis="false" router>
    <el-menu-item index="/">
      <div class="flex items-center justify-center gap-2">
        <div class="text-xl" i-ep-element-plus />
        <span>ZoysiaGenomeDataBase</span>
      </div>
    </el-menu-item>
    <el-sub-menu index="nav/Overview">
      <template #title>
        Overview
      </template>
      <el-sub-menu index="nav/Overview-1">
        <template #title>
          Overview-1
        </template>
        <el-menu-item index="nav/Overview-1/Overview-1-1">
          item one
        </el-menu-item>
      </el-sub-menu>
      <el-menu-item index="nav/Overview-2">
        item two
      </el-menu-item>
    </el-sub-menu>
    <el-menu-item index="blast">
      BLAST
    </el-menu-item>

    <el-menu-item h="full" @click="toggleDark()">
      <button
        class="w-full cursor-pointer border-none bg-transparent"
        style="height: var(--ep-menu-item-height)"
      >
        <i inline-flex i="dark:ep-moon ep-sunny" />
      </button>
    </el-menu-item>

    <el-menu-item h="full">
      <a class="size-full flex items-center justify-center" :href="repository.url" target="_blank">
        <div i-ri-github-fill />
      </a>
    </el-menu-item>
  </el-menu>
</template>

<style lang="scss">
.el-menu-demo {
  &.ep-menu--horizontal > .ep-menu-item:nth-child(1) {
    margin-right: auto;
  }
}
</style>
