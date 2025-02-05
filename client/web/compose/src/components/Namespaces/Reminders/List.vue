<template>
  <div
    class="d-flex flex-column h-100"
  >
    <div class="flex-fill overflow-auto px-2">
      <div
        v-for="(r, i) in sortedReminders"
        :key="r.reminderID"
        :style="`${!!r.dismissedAt ? 'opacity:.6;' : ''}`"
      >
        <hr v-if="r.dismissedAt && sortedReminders[i - 1] ? !sortedReminders[i - 1].dismissedAt : false ">

        <div
          class="overflow-auto border card shadow-sm my-2 p-1"
        >
          <div
            class="d-flex flex-row flex-nowrap align-items-center"
          >
            <b-form-checkbox
              v-b-tooltip.hover.left.350
              data-test-id="checkbox-dismiss-reminder"
              :checked="!!r.dismissedAt"
              :title="$t(`reminder.${!!r.dismissedAt ? 'undismiss' : 'dismiss'}`)"
              class="my-2 ml-2"
              @change="$emit('dismiss', r, $event)"
            />

            <div
              data-test-id="span-reminder-title"
              class="text-break text-truncate"
              :style="`${!!r.dismissedAt ? 'text-decoration: line-through;' : ''}`"
            >
              {{ r.payload.title || r.link || rlLabel(r) || r.linkLabel }}
            </div>

            <div
              class="d-flex align-items-center text-primary ml-auto px-2"
            >
              <b-button-group
                size="sm"
              >
                <b-button
                  v-if="r.payload.link"
                  :to="recordViewer(r.payload.link)"
                  :title="$t('reminder.recordPageLink')"
                  variant="outline-light"
                  class="d-flex align-items-center py-2 text-primary border-0"
                >
                  <font-awesome-icon :icon="['far', 'file-alt']" />
                </b-button>

                <b-button
                  data-test-id="button-edit-reminder"
                  variant="outline-light"
                  :title="$t('reminder.edit.label')"
                  class="d-flex align-items-center py-2 text-primary border-0"
                  @click="$emit('edit', r)"
                >
                  <font-awesome-icon :icon="['far', 'edit']" />
                </b-button>

                <b-button
                  data-test-id="button-delete-reminder"
                  variant="outline-light"
                  :title="$t('reminder.delete')"
                  class="d-flex align-items-center py-2 text-danger border-0"
                  @click.prevent="$emit('delete', r)"
                >
                  <font-awesome-icon
                    :icon="['far', 'trash-alt']"
                  />
                </b-button>
              </b-button-group>
            </div>
          </div>

          <div
            v-if="r.remindAt"
            class="text-secondary small px-2 pb-1"
          >
            <font-awesome-icon
              data-test-id="icon-remind-at"
              :title="$t('reminder.snooze.count', { count: r.snoozeCount })"
              :icon="['far', 'bell']"
              class="text-primary"
            />
            {{ r.remindAt | locFullDateTime }}
          </div>

          <div
            v-if="r.payload.notes"
            class="text-secondary text-truncate px-2 pb-2 small"
          >
            {{ r.payload.notes }}
          </div>
        </div>
      </div>
    </div>

    <div
      class="text-center bg-white py-3 sticky-top"
    >
      <b-button
        data-test-id="button-add-reminder"
        size="sm"
        variant="outline-primary"
        @click="$emit('edit')"
      >
        + {{ $t('reminder.add') }}
      </b-button>
    </div>
  </div>
</template>

<script>
import { fmt } from '@cortezaproject/corteza-js'

export default {
  i18nOptions: {
    namespaces: 'general',
  },

  props: {
    reminders: {
      type: Array,
      required: true,
      default: () => [],
    },
  },

  computed: {
    sortedReminders () {
      return [...this.reminders].sort(this.stdSort)
    },
  },

  methods: {
    // Determine abs. link for given router-link
    rlLabel (r) {
      const rl = r.routerLink
      if (!rl) {
        return
      }
      return `${document.location.origin}${this.$router.resolve(rl).href}`
    },

    stdSort (a, b) {
      if (!a.dismissedAt) {
        return -1
      }
      if (!b.dismissedAt) {
        return 0
      }

      return a.dismissedAt - b.dismissedAt
    },

    makeTooltip ({ remindAt }) {
      return fmt.fullDateTime(remindAt)
    },

    recordViewer ({ params } = {}) {
      return params ? { name: 'page.record', params } : undefined
    },
  },
}
</script>
