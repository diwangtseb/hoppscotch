<template>
  <AppSection label="headers">
    <div
      class="
        bg-primary
        border-b border-dividerLight
        flex flex-1
        top-upperSecondaryStickyFold
        pl-4
        z-10
        sticky
        items-center
        justify-between
      "
    >
      <label class="font-semibold text-secondaryLight">
        {{ $t("request.header_list") }}
      </label>
      <div class="flex">
        <ButtonSecondary
          v-tippy="{ theme: 'tooltip' }"
          to="https://docs.hoppscotch.io/features/headers"
          blank
          :title="$t('app.wiki')"
          svg="help-circle"
        />
        <ButtonSecondary
          v-tippy="{ theme: 'tooltip' }"
          :title="$t('action.clear_all')"
          svg="trash-2"
          :disabled="bulkMode"
          @click.native="clearContent"
        />
        <ButtonSecondary
          v-tippy="{ theme: 'tooltip' }"
          :title="$t('state.bulk_mode')"
          svg="edit"
          :class="{ '!text-accent': bulkMode }"
          @click.native="bulkMode = !bulkMode"
        />
        <ButtonSecondary
          v-tippy="{ theme: 'tooltip' }"
          :title="$t('add.new')"
          svg="plus"
          :disabled="bulkMode"
          @click.native="addHeader"
        />
      </div>
    </div>
    <div v-if="bulkMode" ref="bulkEditor"></div>
    <div v-else>
      <div
        v-for="(header, index) in headers$"
        :key="`header-${index}`"
        class="divide-x divide-dividerLight border-b border-dividerLight flex"
      >
        <SmartAutoComplete
          :placeholder="`${$t('count.header', { count: index + 1 })}`"
          :source="commonHeaders"
          :spellcheck="false"
          :value="header.key"
          autofocus
          styles="
            bg-transparent
            flex
            flex-1
            py-1
            px-4
            truncate
          "
          :class="{ '!flex flex-1': EXPERIMENTAL_URL_BAR_ENABLED }"
          @input="
            updateHeader(index, {
              key: $event,
              value: header.value,
              active: header.active,
            })
          "
        />
        <SmartEnvInput
          v-if="EXPERIMENTAL_URL_BAR_ENABLED"
          v-model="header.value"
          :placeholder="`${$t('count.value', { count: index + 1 })}`"
          styles="
            bg-transparent
            flex
            flex-1
            py-1
            px-4
          "
          @change="
            updateHeader(index, {
              key: header.key,
              value: $event,
              active: header.active,
            })
          "
        />
        <input
          v-else
          class="bg-transparent flex flex-1 py-2 px-4"
          :placeholder="`${$t('count.value', { count: index + 1 })}`"
          :name="'value' + index"
          :value="header.value"
          @change="
            updateHeader(index, {
              key: header.key,
              value: $event.target.value,
              active: header.active,
            })
          "
        />
        <span>
          <ButtonSecondary
            v-tippy="{ theme: 'tooltip' }"
            :title="
              header.hasOwnProperty('active')
                ? header.active
                  ? $t('action.turn_off')
                  : $t('action.turn_on')
                : $t('action.turn_off')
            "
            :svg="
              header.hasOwnProperty('active')
                ? header.active
                  ? 'check-circle'
                  : 'circle'
                : 'check-circle'
            "
            color="green"
            @click.native="
              updateHeader(index, {
                key: header.key,
                value: header.value,
                active: header.hasOwnProperty('active')
                  ? !header.active
                  : false,
              })
            "
          />
        </span>
        <span>
          <ButtonSecondary
            v-tippy="{ theme: 'tooltip' }"
            :title="$t('action.remove')"
            svg="trash"
            color="red"
            @click.native="deleteHeader(index)"
          />
        </span>
      </div>
      <div
        v-if="headers$.length === 0"
        class="
          flex flex-col
          text-secondaryLight
          p-4
          items-center
          justify-center
        "
      >
        <span class="text-center pb-4">
          {{ $t("empty.headers") }}
        </span>
        <ButtonSecondary
          filled
          :label="`${$t('add.new')}`"
          svg="plus"
          @click.native="addHeader"
        />
      </div>
    </div>
  </AppSection>
</template>

<script setup lang="ts">
import { ref, useContext, watch } from "@nuxtjs/composition-api"
import { useCodemirror } from "~/helpers/editor/codemirror"
import {
  addRESTHeader,
  deleteAllRESTHeaders,
  deleteRESTHeader,
  restHeaders$,
  setRESTHeaders,
  updateRESTHeader,
} from "~/newstore/RESTSession"
import { commonHeaders } from "~/helpers/headers"
import { useSetting } from "~/newstore/settings"
import { useReadonlyStream } from "~/helpers/utils/composables"
import { HoppRESTHeader } from "~/helpers/types/HoppRESTRequest"

const {
  $toast,
  app: { i18n },
} = useContext()
const t = i18n.t.bind(i18n)

const bulkMode = ref(false)
const bulkHeaders = ref("")
const bulkEditor = ref<any | null>(null)

useCodemirror(bulkEditor, bulkHeaders, {
  extendedEditorConfig: {
    mode: "text/x-yaml",
    placeholder: `${t("state.bulk_mode_placeholder")}`,
  },
  linter: null,
  completer: null,
})

watch(bulkHeaders, () => {
  try {
    const transformation = bulkHeaders.value.split("\n").map((item) => ({
      key: item.substring(0, item.indexOf(":")).trim().replace(/^\/\//, ""),
      value: item.substring(item.indexOf(":") + 1).trim(),
      active: !item.trim().startsWith("//"),
    }))
    setRESTHeaders(transformation)
  } catch (e) {
    $toast.error(`${t("error.something_went_wrong")}`, {
      icon: "error_outline",
    })
    console.error(e)
  }
})

const headers$ = useReadonlyStream(restHeaders$, [])

watch(
  headers$,
  (newValue) => {
    if (
      (newValue[newValue.length - 1]?.key !== "" ||
        newValue[newValue.length - 1]?.value !== "") &&
      newValue.length
    )
      addHeader()
  },
  { deep: true }
)

const addHeader = () => {
  addRESTHeader({ key: "", value: "", active: true })
}

const updateHeader = (index: number, item: HoppRESTHeader) => {
  updateRESTHeader(index, item)
}

const deleteHeader = (index: number) => {
  deleteRESTHeader(index)
}

const clearContent = () => {
  deleteAllRESTHeaders()
}
const EXPERIMENTAL_URL_BAR_ENABLED = useSetting("EXPERIMENTAL_URL_BAR_ENABLED")
</script>
