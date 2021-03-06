<template>
  <div id="popup">
    <PopupStatusCardComponent v-if="is_active_by_badge_loaded"
                              :is_active_by_badge="is_active_by_badge"
                              :is_active_by_status.sync="is_active_by_real_status"/>
    <PopupListCardComponent
        v-if="is_list_feature_active()"
        :current_url="current_url" :is_pi_hole_version_5_or_higher.sync="is_pi_hole_version_5_or_higher"/>
    <PopupUpdateAlertComponent v-if="is_active_by_real_status"/>
  </div>
</template>

<script lang="ts">

import {Component} from 'vue-property-decorator';
import PopupStatusCardComponent from "./PopupStatusCardComponent.vue";
import PopupListCardComponent from "./PopupListCardComponent.vue";
import {ExtensionBadgeText} from "../../../service/browser/BadgeService";
import PopupUpdateAlertComponent from "./PopupUpdateAlertComponent.vue";
import BaseComponent from "../../general/BaseComponent.vue";

@Component({
  components: {PopupUpdateAlertComponent, PopupListCardComponent, PopupStatusCardComponent}
})
/**
 * The Main PopupComponent.
 */
export default class PopupComponent extends BaseComponent {
  // Data Prop: is the pi-hole active by using the status of the badge
  private is_active_by_badge: boolean = false;

  // Data Prop: Is the the badge status loaded. true will start rendering the cards
  private is_active_by_badge_loaded: boolean = false

  // Data Prop: How is the real status of the pi hole
  private is_active_by_real_status: boolean = false;

  // Data Prop: Has every pihole version 5 or higher?
  private is_pi_hole_version_5_or_higher: boolean = true;

  // Data Prop of the current url
  private current_url: string = '';

  // Is the list feature disabled by the settings?
  private list_feature_disabled = false

  mounted() {
    this.update_is_active_by_badge();
    this.update_current_url();
    this.update_list_feature_disabled();
  }

  /**
   * Gets the prop by the badge status
   */
  private update_is_active_by_badge(): void {
    this.get_badge_service().get_badge_text().then((text: string) => {
      this.is_active_by_badge = text === ExtensionBadgeText.enabled;
      this.is_active_by_badge_loaded = true;
    })
  }

  /**
   * Gets the current url and saves it to the prop
   */
  private update_current_url(): void {
    this.get_tab_service().get_current_tab_url_cleaned().then((url: string) => {
      if (url.length > 0) {
        this.current_url = url;
      }
    })
  }

  /**
   * Updates the prop by the storage
   */
  private update_list_feature_disabled(): void {
    this.get_storage_service().get_disable_list_feature().then((state: boolean | undefined) => {
      if (typeof state !== "undefined") {
        this.list_feature_disabled = state;
      }
    })
  }

  /**
   * Determines if the list feature should be shown or not
   */
  private is_list_feature_active(): boolean {
    return !this.list_feature_disabled && this.is_pi_hole_version_5_or_higher && this.is_active_by_real_status && this.current_url.length > 0;
  }
}


</script>

<style lang="scss">

$card-padding: 5px;

#popup {
  margin: 1px 1px 1px 1px;
  text-align: center;
}

body {
  width: 150px;
  overflow: hidden;
}

.card-body {
  padding: $card-padding !important;
  font-size: 14px;
}

.card:nth-child(n+2) {
  margin-top: 5px;
}

.card-header {
  padding: $card-padding !important;
  font-size: 16px;
}

.card-footer {
  padding: $card-padding !important;
}

.status {
  font-size: 16px;
  text-align: center;
  font-weight: bold;
}
</style>
