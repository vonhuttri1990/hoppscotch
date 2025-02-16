<template>
  <div class="flex flex-col flex-1">
    <div
      class="sticky z-10 flex items-center justify-between flex-shrink-0 pl-4 overflow-x-auto border-b bg-primary border-dividerLight top-upperMobileSecondaryStickyFold sm:top-upperSecondaryStickyFold"
    >
      <span class="flex items-center">
        <label class="font-semibold truncate text-secondaryLight">
          {{ t("authorization.type") }}
        </label>
        <tippy
          interactive
          trigger="click"
          theme="popover"
          :on-shown="() => tippyActions.focus()"
        >
          <span class="select-wrapper">
            <HoppButtonSecondary
              class="pr-8 ml-2 rounded-none"
              :label="authName"
            />
          </span>
          <template #content="{ hide }">
            <div
              ref="tippyActions"
              class="flex flex-col focus:outline-none"
              tabindex="0"
              @keyup.escape="hide()"
            >
              <HoppSmartItem
                label="None"
                :icon="authName === 'None' ? IconCircleDot : IconCircle"
                :active="authName === 'None'"
                @click="
                  () => {
                    auth.authType = 'none'
                    hide()
                  }
                "
              />
              <HoppSmartItem
                label="Basic Auth"
                :icon="authName === 'Basic Auth' ? IconCircleDot : IconCircle"
                :active="authName === 'Basic Auth'"
                @click="
                  () => {
                    auth.authType = 'basic'
                    hide()
                  }
                "
              />
              <HoppSmartItem
                label="Bearer Token"
                :icon="authName === 'Bearer' ? IconCircleDot : IconCircle"
                :active="authName === 'Bearer'"
                @click="
                  () => {
                    auth.authType = 'bearer'
                    hide()
                  }
                "
              />
              <HoppSmartItem
                label="OAuth 2.0"
                :icon="authName === 'OAuth 2.0' ? IconCircleDot : IconCircle"
                :active="authName === 'OAuth 2.0'"
                @click="
                  () => {
                    auth.authType = 'oauth-2'
                    hide()
                  }
                "
              />
              <HoppSmartItem
                label="API key"
                :icon="authName === 'API key' ? IconCircleDot : IconCircle"
                :active="authName === 'API key'"
                @click="
                  () => {
                    auth.authType = 'api-key'
                    hide()
                  }
                "
              />
            </div>
          </template>
        </tippy>
      </span>
      <div class="flex">
        <!-- <HoppSmartCheckbox
          :on="!URLExcludes.auth"
          @change="setExclude('auth', !$event)"
        >
          {{ $t("authorization.include_in_url") }}
        </HoppSmartCheckbox>-->
        <HoppSmartCheckbox
          :on="authActive"
          class="px-2"
          @change="authActive = !authActive"
          >{{ t("state.enabled") }}</HoppSmartCheckbox
        >
        <HoppButtonSecondary
          v-tippy="{ theme: 'tooltip' }"
          to="https://docs.hoppscotch.io/features/authorization"
          blank
          :title="t('app.wiki')"
          :icon="IconHelpCircle"
        />
        <HoppButtonSecondary
          v-tippy="{ theme: 'tooltip' }"
          :title="t('action.clear')"
          :icon="IconTrash2"
          @click="clearContent"
        />
      </div>
    </div>
    <div
      v-if="auth.authType === 'none'"
      class="flex flex-col items-center justify-center p-4 text-secondaryLight"
    >
      <img
        :src="`/images/states/${colorMode.value}/login.svg`"
        loading="lazy"
        class="inline-flex flex-col object-contain object-center w-16 h-16 my-4"
        :alt="`${t('empty.authorization')}`"
      />
      <span class="pb-4 text-center">{{ t("empty.authorization") }}</span>
      <HoppButtonSecondary
        outline
        :label="t('app.documentation')"
        to="https://docs.hoppscotch.io/features/authorization"
        blank
        :icon="IconExternalLink"
        reverse
        class="mb-4"
      />
    </div>
    <div v-else class="flex flex-1 border-b border-dividerLight">
      <div class="w-2/3 border-r border-dividerLight">
        <div v-if="auth.authType === 'basic'">
          <HttpAuthorizationBasic v-model="auth" />
        </div>
        <div v-if="auth.authType === 'bearer'">
          <div class="flex flex-1 border-b border-dividerLight">
            <SmartEnvInput v-model="auth.token" placeholder="Token" />
          </div>
        </div>
        <div v-if="auth.authType === 'oauth-2'">
          <div class="flex flex-1 border-b border-dividerLight">
            <SmartEnvInput v-model="auth.token" placeholder="Token" />
          </div>
          <HttpOAuth2Authorization v-model="auth" />
        </div>
        <div v-if="auth.authType === 'api-key'">
          <HttpAuthorizationApiKey v-model="auth" />
        </div>
      </div>
      <div
        class="sticky flex-shrink-0 h-full p-4 overflow-auto overflow-x-auto bg-primary top-upperTertiaryStickyFold min-w-46 max-w-1/3 z-9"
      >
        <div class="pb-2 text-secondaryLight">
          {{ t("helpers.authorization") }}
        </div>
        <HoppSmartAnchor
          class="link"
          :label="t('authorization.learn')"
          :icon="IconExternalLink"
          to="https://docs.hoppscotch.io/features/authorization"
          blank
          reverse
        />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import IconHelpCircle from "~icons/lucide/help-circle"
import IconTrash2 from "~icons/lucide/trash-2"
import IconExternalLink from "~icons/lucide/external-link"
import IconCircleDot from "~icons/lucide/circle-dot"
import IconCircle from "~icons/lucide/circle"
import { computed, ref } from "vue"
import { HoppRESTAuth } from "@hoppscotch/data"
import { pluckRef } from "@composables/ref"
import { useI18n } from "@composables/i18n"
import { useColorMode } from "@composables/theming"
import { useVModel } from "@vueuse/core"

const t = useI18n()

const colorMode = useColorMode()

const props = defineProps<{
  modelValue: HoppRESTAuth
}>()

const emit = defineEmits<{
  (e: "update:modelValue", value: HoppRESTAuth): void
}>()

const auth = useVModel(props, "modelValue", emit)

const AUTH_KEY_NAME = {
  basic: "Basic Auth",
  bearer: "Bearer",
  "oauth-2": "OAuth 2.0",
  "api-key": "API key",
  none: "None",
} as const

const authType = pluckRef(auth, "authType")
const authName = computed(() =>
  AUTH_KEY_NAME[authType.value] ? AUTH_KEY_NAME[authType.value] : "None"
)
const authActive = pluckRef(auth, "authActive")

const clearContent = () => {
  auth.value = {
    authType: "none",
    authActive: true,
  }
}

// Template refs
const tippyActions = ref<any | null>(null)
</script>
