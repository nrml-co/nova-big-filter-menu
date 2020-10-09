<template>
  <div v-if="filters.length > 0" class="bg-30 border-b border-60">
    <scroll-wrap :height="card.filterMaxHeight ? card.filterMaxHeight : 350">
      <div v-if="! card.filterHideTitle" class="py-2 w-full block text-xs uppercase tracking-wide text-center text-80 dim font-bold focus:outline-none">
        {{this.card.filterMenuTitle ? this.card.filterMenuTitle : 'Filter Menu'}}
      </div>
      <button v-if="card.showToggleButton"
              @click="showFilters = !showFilters"
              class="btn btn-default btn-primary">
        <svg height="24" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg">
          <path fill="currentColor" d="m12 21c-5.196 0-9.815-3.067-11.767-7.814-.31-.753-.31-1.618 0-2.371 1.952-4.748 6.571-7.815 11.767-7.815s9.815 3.067 11.767 7.814c.31.753.31 1.618 0 2.371-1.952 4.748-6.571 7.815-11.767 7.815zm0-17c-4.789 0-9.045 2.824-10.842 7.194-.21.512-.21 1.099 0 1.611 1.797 4.371 6.053 7.195 10.842 7.195s9.045-2.824 10.842-7.194c.21-.512.21-1.099 0-1.611-1.797-4.371-6.053-7.195-10.842-7.195z"/>
          <path fill="currentColor" d="m12 16c-2.206 0-4-1.794-4-4s1.794-4 4-4 4 1.794 4 4-1.794 4-4 4zm0-7c-1.654 0-3 1.346-3 3s1.346 3 3 3 3-1.346 3-3-1.346-3-3-3z"/>
        </svg>
      </button>

      <!-- Custom Filters -->
      <div v-show="showFilters" v-for="filters in this.filterRows">
        <div class="float-left nova-big-filter-col">
          <component
              v-if="filters[0]"
              :resource-name="resourceName"
              :key="filters[0].name"
              :filter-key="filters[0].class"
              :is="filters[0].component"
              @input="filterChanged"
              @change="filterChanged"
          />
        </div>
        <div class="float-left nova-big-filter-col">
          <component
              v-if="filters[1]"
              :resource-name="resourceName"
              :key="filters[1].name"
              :filter-key="filters[1].class"
              :is="filters[1].component"
              @input="filterChanged"
              @change="filterChanged"
          />
        </div>
        <div class="float-left nova-big-filter-col">
          <component
              v-if="filters[2]"
              :resource-name="resourceName"
              :key="filters[2].name"
              :filter-key="filters[2].class"
              :is="filters[2].component"
              @input="filterChanged"
              @change="filterChanged"
          />
        </div>
      </div>

      <!-- Soft Deletes -->
      <div v-show="showFilters" v-if="softDeletes && showTrashedOption">
        <h3 slot="default" class="text-sm uppercase tracking-wide text-80 bg-30 p-3">
          {{ __('Trashed') }}
        </h3>

        <div class="p-2">
          <select
              slot="select"
              class="block w-full form-control-sm form-select"
              dusk="trashed-select"
              :value="trashed"
              @change="trashedChanged"
          >
            <option value="" selected>&mdash;</option>
            <option value="with">{{ __('With Trashed') }}</option>
            <option value="only">{{ __('Only Trashed') }}</option>
          </select>
        </div>
      </div>

      <div v-show="showFilters" v-if="filtersAreApplied" class="bg-30 border-b border-60">
        <button
            @click="clearSelectedFilters"
            class="py-2 w-full block text-xs uppercase tracking-wide text-center text-80 dim font-bold focus:outline-none"
        >
          {{ __('Reset Filters') }}
        </button>
      </div>

    </scroll-wrap>
  </div>
</template>

<script>
import { Filterable, InteractsWithQueryString } from 'laravel-nova'
export default {
  mixins: [ Filterable, InteractsWithQueryString ],
  props: {
    card: {
      filterMenuTitle: String,
      filterMaxHeight: Number,
      filterHideTitle: {
        type: Boolean,
        default: false
      }
    },
    resourceName: String,
    softDeletes: Boolean,
    viaResource: String,
    viaHasOne: Boolean,
    trashed: {
      type: String,
      validator: value => ['', 'with', 'only'].indexOf(value) !== -1,
    },
    perPage: [String, Number],
    showTrashedOption: {
      type: Boolean,
      default: true,
    },
    showFilters: {
      type: Boolean,
      default: false,
    },
  },
  methods: {
    trashedChanged(event) {
      this.$emit('trashed-changed', event.target.value)
    },

    perPageChanged(event) {
      this.$emit('per-page-changed', event.target.value)
    },
  },

  computed: {
    /**
     * Get the name of the page query string variable.
     */
    pageParameter() {
      return this.viaRelationship
          ? this.viaRelationship + '_page'
          : this.resourceName + '_page'
    },
    /**
     * Return the filters from state
     */
    filters() {
      return this.$store.getters[`${this.resourceName}/filters`]
    },

    /**
     * Determine via state whether filters are applied
     */
    filtersAreApplied() {
      return this.$store.getters[`${this.resourceName}/filtersAreApplied`]
    },

    /**
     * Return the number of active filters
     */
    activeFilterCount() {
      return this.$store.getters[`${this.resourceName}/activeFilterCount`]
    },

    filterRows(){
      if( this.filters.length > 3)
      {
        return _.chunk(this.filters, 3)
      }
      else
      {
        return [ this.filters ]
      }
    }
  },
}
</script>
