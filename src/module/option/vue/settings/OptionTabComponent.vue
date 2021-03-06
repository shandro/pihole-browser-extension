<template>
  <b-tabs v-model="current_tab_index">
    <b-tab v-for="(pi_hole_setting, index) in tabs" :key="'dyn-tab-' + index" :title="'PiHole ' + (index + 1)">
      <b-form-group :label="translate(i18nOptionsKeys.options_pi_hole_address)" style="margin-top: 1rem">
        <b-form-input v-model="pi_hole_setting.pi_uri_base"
                      :placeholder="PiHoleSettingsDefaults.pi_uri_base"
                      :state="is_invalid_url_schema(pi_hole_setting.pi_uri_base)"
                      required
        ></b-form-input>
        <b-form-invalid-feedback>{{ translate(i18nOptionsKeys.options_url_invalid_warning) }}
        </b-form-invalid-feedback>
      </b-form-group>
      <b-form-group :label="translate(i18nOptionsKeys.options_api_key)">
        <b-form-input v-model="pi_hole_setting.api_key"
                      :state="is_invalid_api_key(pi_hole_setting.api_key)"
        ></b-form-input>
        <b-form-invalid-feedback>{{ translate(i18nOptionsKeys.options_api_key_invalid_warning) }}
        </b-form-invalid-feedback>
      </b-form-group>

    </b-tab>

    <!-- New Tab Button (Using tabs-end slot) -->
    <template v-slot:tabs-end style="font-size: 20px;">
      <b-nav-item v-if="tabs.length < 4" :title="translate(i18nOptionsKeys.options_add_button)" href="#"
                  link-classes="no-white-hover-border" role="presentation"
                  @click.prevent="add_new_settings_tab">
        <b-icon-plus-circle style="width: 20px;height: 20px" variant="success"></b-icon-plus-circle>
      </b-nav-item>
      <b-nav-item v-if="tabs.length > 1" :title="translate(i18nOptionsKeys.options_remove_button)"
                  href="#"
                  link-classes="no-white-hover-border" role="presentation"
                  @click.prevent="remove_last_settings_tab">
        <b-icon-x-circle style="width: 20px;height: 20px" variant="danger"
        ></b-icon-x-circle>
      </b-nav-item>
    </template>

  </b-tabs>
</template>

<script lang="ts">
import {Component, Watch} from "vue-property-decorator";
import {PiHoleSettingsStorage} from "../../../../service/browser/StorageService";
import BaseComponent from "../../../general/BaseComponent.vue";

@Component
/**
 * Component for the different PiHole Settings
 **/
export default class OptionTabComponent extends BaseComponent {
  // Data prop of the current tabs as array
  private tabs: Array<PiHoleSettingsStorage> = [this.default_empty_option_tab()];
  // Data prop of the currents tab index
  private current_tab_index = 0;

  mounted() {
    this.update_tabs_settings();
  }

  @Watch('tabs', {deep: true})
  private on_tabs_changed(): void {
    for (const pi_hole_setting of this.tabs) {
      if (typeof pi_hole_setting.pi_uri_base !== "undefined") {
        pi_hole_setting.pi_uri_base = pi_hole_setting.pi_uri_base.replace(/\s+/g, '');
      } else {
        pi_hole_setting.pi_uri_base = '';
      }
      if (typeof pi_hole_setting.api_key !== "undefined") {
        pi_hole_setting.api_key = pi_hole_setting.api_key.replace(/\s+/g, '');
      } else {
        pi_hole_setting.api_key = '';
      }
    }
    this.get_storage_service().save_pi_hole_settings_array(this.tabs);
  }

  /**
   * Validation Function for the api key
   * @param api_key
   */
  private is_invalid_api_key(api_key: string): boolean | null {
    return (!api_key.match('^[a-f0-9]{64}$') && api_key.length !== 0) ? false : null;
  }

  /**
   * Validation Function for the pi hole url
   */
  private is_invalid_url_schema(pi_hole_uri: string): boolean | null {
    return (!pi_hole_uri.match('^(http|https):\\/\\/[^ "]+$') || pi_hole_uri.length < 1) ? false : null;
  }

  /**
   * Getter for an empty pihole settings storage
   */
  private default_empty_option_tab(): PiHoleSettingsStorage {
    return {
      pi_uri_base: '',
      api_key: ''
    }
  }

  /**
   * Adds a new tab
   */
  private add_new_settings_tab(): void {
    this.tabs.push(this.default_empty_option_tab());
    setTimeout(() => this.current_tab_index = this.tabs.length - 1, 0);
  }

  /**
   * Removes the last tab
   */
  private remove_last_settings_tab(): void {
    this.tabs.pop();
  }

  /**
   * Updates the tabs with the storage settings
   */
  private update_tabs_settings(): void {
    this.get_storage_service().get_pi_hole_settings_array().then(results => {
      if (typeof results !== "undefined" && results.length > 0) {
        this.tabs = results;
      }
    })
  }
}

</script>
<style lang="scss" scoped>

.no-white-hover-border:hover {
  border-color: rgba(255, 255, 255, 0);
}

.no-white-hover-border:focus {
  border-color: rgba(255, 255, 255, 0);
}

</style>
