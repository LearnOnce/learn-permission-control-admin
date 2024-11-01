<script setup lang="ts">
import { useAppConfig } from '@/stores/useAppConfig'
import { useAuthorize } from '@/stores/useAuthorize'
import type { MenuOption } from 'naive-ui'
import { Icon } from '@iconify/vue'
import { RouterLink } from 'vue-router'

const route = useRoute()
const { menuTree } = useAuthorize()
const appConfig = useAppConfig()

// 侧边栏折叠
const collapsedIconSize = computed(() => (appConfig.collapse ? 20 : 16))

// 当前路由的 name
const activeRouteName = computed(() => route.name as string)

// 菜单组件 label
function renderMenuLabel(option: MenuOption) {
  if (!option.children || !option.children.length) {
    return h(
      RouterLink,
      {
        to: {
          name: option.key as string,
        },
      },
      { default: () => option.label },
    )
  } else {
    return h('span', option.label as string)
  }
}

// 菜单组件图标
function renderMenuIcon(option: MenuOption) {
  const meta: ObjectAny = option?._meta as unknown as ObjectAny
  if (!meta) return null
  if (!meta?.icon) return null
  return h(Icon, { icon: meta.icon, width: collapsedIconSize.value })
}

// 将路由树转为菜单树可使用树
function resolveMenu(tree: ObjectAny | ObjectAny[]) {
  return (Array.isArray(tree) ? tree : [tree])
    .map(item => {
      const newItem: MenuOption & { _meta: ObjectAny } = {
        key: item.name,
        label: item.title,
        _meta: item,
      }
      if (item.children && item.children.length) {
        newItem['children'] = resolveMenu(item.children)
      }
      return newItem
    })
    .filter(item => item._meta.showAside)
}
</script>

<template>
  <n-menu
    :value="activeRouteName"
    :indent="22"
    :collapsed="appConfig.collapse"
    :collapsed-width="appConfig.menubarCollapseWidth"
    :options="resolveMenu(menuTree)"
    :render-label="renderMenuLabel"
    :render-icon="renderMenuIcon"
    default-expand-all
  />
</template>
