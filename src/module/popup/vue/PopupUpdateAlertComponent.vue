<template>
  <b-alert v-if="updates_available && !notification_is_disabled" class="popup-update-alert" show variant="danger">
    {{
      translate(i18nPopupKeys.popup_update_card_info, [
        updates_available_amount,
        amount_of_piholes
      ])
    }}
  </b-alert>
</template>

<script lang="ts">

import {Component, Prop} from "vue-property-decorator";
import {PiHoleVersions} from "../../../service/api/models/pihole/PiHoleVersions";
import BaseComponent from "../../general/BaseComponent.vue";

@Component
export default class PopupUpdateAlertComponent extends BaseComponent {
  // Prop which is emitted to the parent. Gets updates after a version check
  @Prop({default: false})
  is_pi_hole_version_5_or_higher!: boolean;

  // Data Prop: Is an update needed?
  private updates_available: boolean = false;

  // Data Prop: How many Updates are available
  private updates_available_amount: number = 0;

  // Data Prop: How many PiHoles does the user use
  private amount_of_piholes: number = 0;

  // Is the notification disabled by the settings?
  private notification_is_disabled: boolean = false;

  mounted() {
    this.update_is_notification_disabled();
    this.check_for_updates();
  }

  /**
   * Updates the value from the storage
   */
  private update_is_notification_disabled(): void {
    this.get_storage_service().get_disable_update_notification().then((state: boolean | undefined) => {
      if (typeof state !== "undefined") {
        this.notification_is_disabled = state;
      }
    })
  }

  /**
   * Checks the pihole(s) for updates
   * Activates the alert if an update is available
   */
  private check_for_updates(): void {
    const get_pi_hole_version_callback = ((pi_hole_versions_array: PiHoleVersions[]) => {
      let update_available = false;
      let amount_updatable = 0;
      for (const pi_hole_version of pi_hole_versions_array) {
        if (pi_hole_version.core_update || pi_hole_version.web_update || pi_hole_version.FTL_update) {
          update_available = true;
          amount_updatable++;
        }

        if (!this.is_pi_hole_version_5_or_higher && pi_hole_version.FTL_current < 5 && pi_hole_version.FTL_current !== -1) {
          this.$emit('update:is_pi_hole_version_5_or_higher', false);
        }
      }
      if (update_available) {
        this.updates_available = update_available;
        this.updates_available_amount = amount_updatable;
        this.amount_of_piholes = pi_hole_versions_array.length;
      }
    })
    this.get_api_service().get_pi_hole_version().then(get_pi_hole_version_callback);

  }
}
</script>

<style lang="scss" scoped>
.popup-update-alert {
  padding: 0 !important;
  font-size: 14px;
  margin-top: 5px;
  font-weight: bold;
  margin-bottom: 0 !important;
  text-align: center;
}
</style>
