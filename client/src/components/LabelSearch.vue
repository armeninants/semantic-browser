<template>
  <div class="my-label-search">
    <div class="input-group">
      <!-- <span class="input-group-addon"></span> -->
      <input
        type="text"
        class="form-control"
        autocomplete="off"
        placeholder="label lookup"
        v-model="valueInternal"
        v-focus="focusOnInput"
        @focus="focusOnInputHandler"
        @keydown.up="upHandler"
        @keydown.esc="hideDropdown"
        @keydown.down="downHandler"
        @keydown.enter="fetch"
      >

      <span class="input-group-btn">
        <button
          type="button"
          :class="['btn btn-primary ', { disabled: loading || buttonDisabled }]"
          :disabled="buttonDisabled"
          @click="fetch"
        >
          <i :class="['fa fa-fw', loading ? 'fa-spinner fa-spin' : 'fa-search']"></i>
          {{ loading ? 'abort': '' }}
        </button>
      </span>
    </div>
    <ul 
      class="dropdown-menu pre-scrollable my-label-search-dropdown"
      v-show="dropdownVisible"
      @keydown.down="downHandler"
      @keydown.up="upHandler"
    >
      <li
        v-for="(item, i) in options"
        :key="i + '_' + item.id"
      >
        <a
          href="#"
          tabindex="0"
          @click.prevent="itemSelectHandler(i)"
          @keydown.enter="itemSelectHandler(i)"
          @keydown.space="itemSelectHandler(i)"
          @keydown.esc="hideDropdown"
          @focus="focusOnItemHandler(i)"
          v-focus="!focusOnInput && dropdownVisible && currentOptionIndex === i"
        >
          <span class="my-option-label">{{item.label}}</span>
          <span :class="['my-otion-key', {parentheses: item.label }]">{{item.id}}</span>
        </a>
      </li>
    </ul>
  </div>
</template>

<script>
/**
 * @vue
 * @author Armen Inants <armen@inants.com>
 */

import { focus } from 'vue-focus'
import { mapGetters, mapActions } from 'vuex'

export default {
  directives: { focus },

  data() {
    return {
      valueInternal: '',
      options: [],
      currentOptionIndex: undefined,
      focusOnInput: false,
      dropdownVisible: false,
      loading: false,
    }
  },

  computed: {
    buttonDisabled() {
      return this.valueInternal.length < 3;
    },

    ...mapGetters([
      'language',
      'sparqlEndpoint',
    ]),
  },

  watch: {
    options(to) {
      if (to.length) { this.showDropdown() }
    },
  },

  mounted() {
    $(window).on('click.labelsearch', event => {
      if (!$.contains(this.$el, event.target)) {
        this.focusOnInput = false;
        this.hideDropdown();
      }
    });
  },

  beforeDestroy() {
    $(window).off('click.labelsearch');
  },

  methods: {
    processResponse(resp) {
      this.options = resp.results.bindings.map(row => ({ id: row.r.value, label: row.l.value }));
    },

    fetch() {
      if (this.loading && this.xhr && this.xhr.abort) {
        this.xhr.abort();
        return;
      }

      if (this.valueInternal.length < 3) { return }

      this.loading = true;
      this.xhr = this.$sparqlGet({
        endpoint: this.sparqlEndpoint,
        sparql: this.labelSearchSparql(this.valueInternal),
      })
        .always(() => this.loading = false)
        .done(resp => this.processResponse(resp))
    },

    labelSearchSparql(keyword) {
      const keywordEscaped = this.doubleEscapeRegExp(keyword);
      const langFilter = this.language !== '*' ? ` && (lang(?l) = '' || LANGMATCHES(lang(?l), '${this.language}'))` : '';

      return `SELECT ?r ?l
        WHERE {
          ?r rdfs:label ?l .
          FILTER (regex(?l, "^${keywordEscaped}", "i")${langFilter})
        } LIMIT 100`;
    },
    showDropdown() {
      this.dropdownVisible = true;
    },

    hideDropdown() {
      this.dropdownVisible = false;
    },

    toggleDropdown() {
      this.dropdownVisible = !this.dropdownVisible;
    },

    focusOnInputHandler() {
      this.focusOnInput = true;
      if (this.options.length) {
        this.showDropdown();
      }
    },

    focusOnItemHandler(index) {
      this.currentOptionIndex = index;
    },

    itemSelectHandler(index) {
      this.valueInternal = this.options[index].label;
      this.setResource(this.options[index].id);
      this.focusOnInput = false;
      this.hideDropdown();
    },

    downHandler() {
      if (!this.dropdownVisible) {
        this.showDropdown();
      }
      this.focusOnInput = false;
      var idx = typeof this.currentOptionIndex !== 'undefined' ? this.currentOptionIndex : -1;
      this.currentOptionIndex = (idx + 1) % this.options.length;
    },

    upHandler() {
      if (!this.dropdownVisible) return;
      this.focusOnInput = false;
      var idx = typeof this.currentOptionIndex !== 'undefined' ? this.currentOptionIndex : 0;
      this.currentOptionIndex = (idx - 1 + this.options.length) % this.options.length;
    },

    ...mapActions([
      'setResource',
    ]),
  },
}
</script>

<style lang="scss" scoped>
.my-label-search-dropdown {
  position: absolute;
  display: block;
}

.my-label-search {
  position: relative;
}

.my-option-label {
  font-weight: bold;
}

.parentheses {
  &:before {
    content: '(';
  }
  &:after {
    content: ')';
  }
}
</style>
