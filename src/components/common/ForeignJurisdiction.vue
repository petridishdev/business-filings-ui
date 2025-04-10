<template>
  <v-card
    id="foreign-jurisdiction"
    flat
  >
    <v-row no-gutters>
      <v-col
        cols="12"
        sm="3"
      >
        <label class="title-label">Jurisdiction</label>
      </v-col>
      <v-col
        cols="12"
        sm="9"
      >
        <v-autocomplete
          id="country-selector"
          ref="countrySelectRef"
          v-model="selectedCountryCode"
          :items="getCountriesList()"
          item-text="name"
          item-value="code"
          filled
          placeholder="Jurisdiction Country"
          :rules="countryRules"
          hide-no-data
          :filter="customFilter"
          :menu-props="{
            auto: false,
            bottom: true
          }"
          @input="emitChangedCountry()"
        >
          <template #item="{ item }">
            {{ item.name }}
          </template>
        </v-autocomplete>

        <v-autocomplete
          v-if="canadaUsaRegions.length > 0"
          id="region-selector"
          ref="regionSelectRef"
          v-model="selectedRegionName"
          :items="canadaUsaRegions"
          item-text="name"
          filled
          placeholder="Jurisdiction Region"
          :rules="regionRules"
          hide-no-data
          :filter="customFilter"
          :menu-props="{
            auto: false,
            bottom: true
          }"
          @input="emitChangedRegion()"
        >
          <template #item="{ item }">
            {{ item.name }}
          </template>
        </v-autocomplete>
      </v-col>
    </v-row>
  </v-card>
</template>

<script lang="ts">
import { Component, Emit, Mixins, Prop, Watch } from 'vue-property-decorator'
import { CountriesProvincesMixin } from '@/mixins'

@Component({})
export default class ForeignJurisdiction extends Mixins(CountriesProvincesMixin) {
    // Refs
    $refs!: {
      countrySelectRef: any
      regionSelectRef: any
  }

  /** Country passed into this component. */
  @Prop({ default: () => '' }) readonly draftCountry!: string

  /** Region passed into this component. */
  @Prop({ default: () => '' }) readonly draftRegion!: string

  /** Prompt the validations. Used for global validations. */
  @Prop({ default: false }) readonly validateForm!: boolean

  selectedCountryCode = ''
  selectedRegionName = ''

  /** Restore the selected country and region from draft filing if applicable. */
  mounted (): void {
    if (this.draftCountry) {
      this.selectedCountryCode = this.draftCountry
      this.emitChangedCountry()
    }
    if (this.draftRegion) {
      this.selectedRegionName = this.getRegionNameFromCode(this.draftRegion)
      this.emitChangedRegion()
    }
  }

  /** Get the matching results from the country/region seach lists. */
  customFilter (item, query, itemText) {
    if (!query) return true
    /** Only show Canada and US once in the search results. */
    if (item.code === 'CA-1' || item.code === 'US-1') return false
    const text = itemText.toString().toLowerCase()
    const search = query.toLowerCase()
    return text.indexOf(search) !== -1
  }

  /** Get the respective regions of the country selected as an array of objects. */
  get canadaUsaRegions (): Array<any> {
    if (this.selectedCountryCode === 'CA') {
      let regions = this.getCountryRegions('CA') as any[]
      regions = regions.filter(province => province.short !== 'BC')
      regions.push({ name: 'Federal', short: 'FEDERAL' })
      return regions
    } else if (this.selectedCountryCode === 'US') {
      return this.getCountryRegions('US')
    }
    return []
  }

  /** Validation rules for the Jurisdiction Country dropdown. */
  get countryRules (): Array<(val) => boolean | string> {
    return [
      val => !!(val) || 'Jurisdiction Country is required.'
    ]
  }

  /** Prioritize Canada and US in the Countries List. */
  getCountriesList (): Array<object> {
    const countries = this.getCountries()
    // List of priority countries
    const priorityCountries = [
      { name: 'Canada', code: 'CA-1' },
      { name: 'United States of America', code: 'US-1' },
      { divider: true }
    ]

    return [
      ...priorityCountries,
      ...countries
    ]
  }

  /** Validation rules for the Jurisdiction Region dropdown. */
  get regionRules (): Array<(val) => boolean | string> {
    return [
      val => !!(val) || 'Jurisdiction Region is required.'
    ]
  }

  /** Helper function to get a region's name when given the short.
   * @example ('AB') -> 'Alberta'
   * @example ('NY') -> 'New York'
   */
  private getRegionNameFromCode (short: string): string {
    const region = this.canadaUsaRegions?.find(region => region.short === short)
    return region?.name
  }

  @Watch('draftCountry')
  onDraftCountryChanged (val: string): void {
    this.selectedCountryCode = val
  }

  @Watch('draftRegion')
  onDraftRegionChanged (val: string): void {
    this.selectedRegionName = val
  }

  /** Validate country field */
  @Watch('validateForm')
  validateForeignJurisdiction (): void {
    if (this.validateForm && !this.selectedCountryCode) {
      this.$refs.countrySelectRef.validate()
      this.$refs.countrySelectRef.error = true
    }
  }

  /** Validate region field */
  @Watch('selectedCountryCode')
  @Watch('validateForm')
  async onCountrychanged (): Promise<void> {
    if (this.validateForm) {
      await this.$nextTick()
      if (this.selectedCountryCode === 'CA' || this.selectedCountryCode === 'US') {
        this.$refs.regionSelectRef.validate()
        this.$refs.regionSelectRef.error = true
      }
    }
  }

  /** Emit the selected country's code whenever a new country is selected. */
  @Emit('update:country')
  emitChangedCountry (): string {
    if (['CA', 'CA-1'].includes(this.selectedCountryCode)) {
      this.selectedCountryCode = 'CA'
    } else if (['US', 'US-1'].includes(this.selectedCountryCode)) {
      this.selectedCountryCode = 'US'
    }
    this.selectedRegionName = ''
    this.emitChangedRegion()
    if (this.selectedCountryCode === 'CA' || this.selectedCountryCode === 'US') {
      this.emitValid(false)
    } else {
      this.emitValid(true)
    }
    return this.selectedCountryCode
  }

  /** Emit the selected country's region short whenever a new region is selected. */
  @Emit('update:region')
  emitChangedRegion (): string {
    if (this.selectedRegionName) {
      this.emitValid(true)
    }
    const region = this.canadaUsaRegions.find(region => region.name === this.selectedRegionName)
    return region?.short
  }

  /** Emits an event indicating whether or not the form is valid. */
  @Emit('valid')
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  emitValid (valid: boolean): void {}
}
</script>

<style lang="scss" scoped>
@import '@/assets/styles/theme.scss';
.title-label {
  color: $gray9;
  font-weight: bold;
}

// Vuetify Overrides
:deep(.v-select__selection--comma) {
  overflow: inherit;
}
</style>
