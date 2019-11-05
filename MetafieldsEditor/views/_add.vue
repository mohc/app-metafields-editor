<template>
  <div style="position:relative;z-index:1073">
    <div class="modal fade" style="display:block" :class="{ show: active }">
      <div class="modal-dialog modal- modal-dialog-centered">
        <div class="modal-content">
          <div class="modal-body p-0">
            <div class="card bg-secondary border-0 mb-0">
              <button type="button" class="close" @click="$parent.add = false">
                <span>×</span>
              </button>
              <div class="card-header bg-transparent pb-1">
                <div class="mt-0 mb-3 h3">Add metafield</div>
              </div>
              <div class="card-body" :class="{ processing: loading }">
                <div class="row y-lg-2">
                  <div class="col-6">
                    <div class="form-group">
                      <label class="text-muted"><small>Namespace</small></label>
                      <input type="text" class="form-control text-xs h-auto" v-model="model.namespace">
                    </div>
                  </div>
                  <div class="col-6">
                    <div class="form-group">
                      <label class="text-muted"><small>Key</small></label>
                      <input type="text" class="form-control text-xs h-auto" v-model="model.key">
                    </div>
                  </div>
                </div>
                <div class="row y-lg-2">
                  <div class="col-6">
                    <div class="form-group">
                      <label class="text-muted"><small>Value</small></label>
                      <input type="text" class="form-control text-xs h-auto" v-model="model.value">
                    </div>
                  </div>
                  <div class="col-6">
                    <div class="form-group">
                      <label class="text-muted"><small>Type</small></label>
                      <select class="form-control text-xs h-auto" v-model="model.value_type">
                        <option value="string">String</option>
                        <option value="integer">Integer</option>
                        <option value="json_string">JSON</option>
                      </select>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="modal-footer justify-content-end">
            <div>
              <button type="button" class="btn btn-primary" :class="{ disabled: loading }" @click="addMetafield">Save</button>
              <button type="button" class="btn btn-secondary" @click="$parent.add = false">Close</button>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="modal-backdrop fade" :class="{ show: active }" @click="$parent.add = false"></div>
  </div>
</template>

<script>
module.exports = {
  props: ['handle', 'id', 'active'],
  data: function() {
    return {
      loading: false,
      model: {
        namespace: '',
        key: '',
        value: '',
        value_type: 'string'
      }
    }
  },
  watch: {
    active: function(val) {
      if (val) {
        this.loading = false
        this.model = {
          namespace: '',
          key: '',
          value: '',
          value_type: 'string'
        }
      }
    }
  },
  methods: {
    addMetafield: function() {
      var self = this
      required = true

      Object.keys(this.model).forEach(function(key) {
        if (self.model[key] == '') {
          required = false
        }
      })

      if (!required) {
        alert('All fields are required')
        return
      }

      this.loading = true

      switch(this.handle) {
        case 'shop':
          endpoint = 'metafields/' + this.id
        break;

        default:
          endpoint = this.handle + '/' + this.id + '/metafields'
      }

      url = '/api/shopify'
      params = {
        method: 'POST',
        headers: this.$root.fetchHeaders,
        body: JSON.stringify({
          endpoint: endpoint,
          method: 'POST',
          params: { metafield: this.model }
        })
      }

      fetch(url, params)
      .then(errorCheck)
      .then(function(res) {
        if (res.errors) {
          self.loading = false
          throw new Error(res.errors.value[0]);
        }
        self.$parent.add = false
        self.$parent.loading = true
        self.$parent.getMetafields()
      })
      .catch((error) => {
        alert(error)
        self.loading = false
      })
    }
  }
}
</script>