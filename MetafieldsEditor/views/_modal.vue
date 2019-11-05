<template>
  <div>
    <div class="modal fade" style="display:block" :class="{ show: id }">
      <div class="modal-dialog modal- modal-dialog-centered modal-lg">
        <div class="modal-content">
          <div class="modal-body p-0">
            <div class="card bg-secondary border-0 mb-0">
              <button type="button" class="close" @click="$parent.modal.id = ''">
                <span>×</span>
              </button>
              <div class="card-header bg-transparent pb-1">
                <div class="mt-0 mb-0 h3">Metafields for:</div>
                <div class="mb-3 h5">{{ title }}</div>
                <form @submit.prevent="searchRun">
                  <div class="row">
                    <div class="col-md-4">
                      <div class="form-group">
                        <input type="text" class="form-control text-xs h-auto" placeholder="Enter search phrase..." v-model="search">
                      </div>
                    </div>
                    <div class="col-md-4">
                      <div class="form-group">
                        <select class="form-control text-xs h-auto" v-model="searchField">
                          <option value="namespace">Namespace</option>
                          <option value="key">Key</option>
                        </select>
                      </div>
                    </div>
                    <div class="col-md-4">
                      <button class="btn btn-primary btn-md lh-120" type="submit" :class="{ disabled: loading }">Search</button>
                    </div>
                  </div>
                </form>
              </div>
              <div class="card-body">
                <div class="loading-skeleton" v-if="loading"><div></div><div></div><div></div></div>
                <div v-else>
                  <div class="row" v-for="(el, index) in list" :class="{ processing: el.processing }">
                    <div class="col-md-3">
                      <div class="form-group">
                        <label class="text-muted"><small>Namespace</small></label>
                        <input type="text" class="form-control text-xs h-auto" disabled v-model="el.namespace">
                      </div>
                    </div>
                    <div class="col-md-3">
                      <div class="form-group">
                        <label class="text-muted"><small>Key</small></label>
                        <input type="text" class="form-control text-xs h-auto" disabled v-model="el.key">
                      </div>
                    </div>
                    <div class="col-md-3">
                      <div class="form-group">
                        <label class="text-muted"><small>Value</small></label>
                        <input type="text" class="form-control text-xs h-auto" v-model="el.value" @change="updateMetafield(el, index)">
                      </div>
                    </div>
                    <div class="col-md-3">
                      <div class="form-group">
                        <label class="text-muted"><small>Type</small></label>
                        <div class="form-row">
                          <div class="col-9">
                            <select class="form-control text-xs h-auto" v-model="el.value_type" @change="updateMetafield(el, index)">
                              <option value="string">String</option>
                              <option value="integer">Integer</option>
                              <option value="json_string">JSON</option>
                            </select>
                          </div>
                          <div class="col-3">
                            <a class="btn btn-icon-only text-danger" @click.prevent="removeMetafield(el)">
                              <i class="fas fa-minus-circle"></i>
                            </a>
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>
                  <div class="row py-lg-2" v-if="!list.length">
                    <div class="col-12">
                      <div class="alert alert-primary bg-gradient-primary" role="alert">
                        <span class="alert-inner--icon mr-2"><i class="fa fa-info-circle"></i></span>
                        <span class="alert-inner--text font-italic">No metafields found</span>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="modal-footer justify-content-between">
            <nav>
              <ul class="pagination justify-content-center mb-0">
                <li class="page-item" :class="{ disabled: page.prev == null || loading }">
                  <a class="page-link" href="#" @click.prevent="prevMetafieldsPage">
                    <i class="fas fa-angle-left"></i>
                    <span class="sr-only">Previous</span>
                  </a>
                </li>
                <li class="page-item" :class="{ disabled: page.next == null || loading }">
                  <a class="page-link" href="#" @click.prevent="nextMetafieldsPage">
                    <i class="fas fa-angle-right"></i>
                    <span class="sr-only">Next</span>
                  </a>
                </li>
              </ul>
            </nav>
            <div>
              <button type="button" class="btn btn-primary" :class="{ disabled: loading }"  @click="add = true">Add metafield</button>
              <button type="button" class="btn btn-secondary" @click="$parent.modal.id = ''">Close</button>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="modal-backdrop fade" :class="{ show: id }" @click="$parent.modal.id = ''"></div>


    <component is="inc-MetafieldsEditor-add" :handle="handle" :id="id" :active="add" />

  </div>
</template>

<script>
module.exports = {
  props: ['handle', 'id', 'title'],
  data: function() {
    return {
      loading: true,
      list: [],
      page: {
        prev: null,
        current: null,
        next: 1
      },
      search: '',
      searchField: 'namespace',
      add: false
    }
  },
  watch: {
    id: function(val) {
      if (val) {
        this.loading = true
        this.page = {
          prev: null,
          current: null,
          next: 1
        }
        this.search = ''
        this.getMetafields()
      }
    }
  },
  methods: {
    getMetafields: function() {
      var self = this
      
      switch(this.handle) {
        case 'shop':
          endpoint = 'metafields'
        break;

        default:
          endpoint = this.handle + '/' + this.id + '/metafields'
      }

      url = '/api/shopify'
      data = {
        page_info: this.page.next === 1 ? null : this.page.next,
        fields: 'id,namespace,key,value,value_type'
      }

      if (this.search != '') {
        data[this.searchField] = this.search
      }

      params = {
        method: 'POST',
        headers: this.$root.fetchHeaders,
        body: JSON.stringify({
          endpoint: endpoint,
          method: 'GET',
          params: data
        })
      }

      fetch(url, params)
      .then(errorCheck)
      .then(function(res) {
        self.list = res.metafields
        self.loading = false
        Vue.set(self, 'page', {
          prev: res.prev ? res.prev : null,
          current: self.page.next,
          next: res.next ? res.next : null
        })
      })
      .catch((error) => {
        alert(error)
      })
    },
    updateMetafield: function(el, index) {
      var self = this

      switch(this.handle) {
        case 'shop':
          endpoint = 'metafields/' + el.id
        break;

        default:
          endpoint = this.handle + '/' + this.id + '/metafields/' + el.id
      }

      url = '/api/shopify'
      params = {
        method: 'POST',
        headers: this.$root.fetchHeaders,
        body: JSON.stringify({
          endpoint: endpoint,
          method: 'PUT',
          params: { metafield: el }
        })
      }

      fetch(url, params)
      .then(errorCheck)
      .catch((error) => {
        alert(error)
      })
    },
    removeMetafield: function(el) {
      var self = this
      Vue.set(el, 'processing', true)

      switch(this.handle) {
        case 'shop':
          endpoint = 'metafields/' + el.id
        break;

        default:
          endpoint = this.handle + '/' + this.id + '/metafields/' + el.id
      }

      url = '/api/shopify'
      params = {
        method: 'POST',
        headers: this.$root.fetchHeaders,
        body: JSON.stringify({
          endpoint: endpoint,
          method: 'DELETE',
          params: { }
        })
      }

      fetch(url, params)
      .then(errorCheck)
      .then(function(res) {
        self.page.next = self.page.current

        if (self.list.length == 1 && self.page.prev) {
          self.page.next = self.page.prev
        }

        self.getMetafields()
      })
      .catch((error) => {
        alert(error)
      })
    },
    searchRun: function() {
      this.page = {
        prev: null,
        current: null,
        next: 1
      }
      this.loading = true
      this.getMetafields()
    },
    nextMetafieldsPage: function() {
      this.loading = true
      this.getMetafields()
    },
    prevMetafieldsPage: function() {
      this.loading = true
      this.page.next = this.page.prev
      this.getMetafields()
    }
  }
}
</script>