<template>
  <b-container fluid>
    <b-row>
    <b-card class="card-accent-primary">
      <!-- User Interface controls -->
      <b-row>
        <b-col md="6" class="my-0">
          <b-form-group horizontal label="Saring" class="mb-0">
            <b-input-group>
              <b-form-input v-model="filter" placeholder="Ketik untuk pencarian" />
              <b-input-group-append>
                <b-btn :disabled="!filter" @click="filter = ''">Kosongkan</b-btn>
              </b-input-group-append>
            </b-input-group>
          </b-form-group>
        </b-col>
        <b-col md="6" class="my-1">
          <b-form-group horizontal label="Data/Halaman" class="mb-0">
            <b-form-select :options="pageOptions" v-model="perPage" />
          </b-form-group>
        </b-col>
      </b-row>

      <!-- Main table element -->
        <b-table show-empty
                stacked="md"
                :items="items"
                :fields="fields"
                :per-page="perPage"
                :current-page="currentPage"
                :filter="filter"
                @filtered="onFiltered"
                :sort-by.sync="sortBy"
                :sort-desc.sync="sortDesc"
                :sort-direction="sortDirection"
                >
          <template slot="index" slot-scope="data">
            {{data.index + 1}}
          </template>
          
          <template slot="aksi" slot-scope="data" >
            <router-link :to="{ path: '/tambah/'+data.item.id }" ><b-btn variant="info" size="sm" class="mb-1 mr-1">Info Detail</b-btn></router-link>
            <router-link :to="{ path: '/tambah/'+data.item.id }" >  <b-btn class="mb-1 mr-1" size="sm" variant="warning">Ubah Data</b-btn></router-link>
            <b-button class="mb-1 mr-1" type="submit" size="sm"  variant="danger" v-on:click="hapus(data.item)">Hapus</b-button>
          </template>
        </b-table>

      <b-row>
        <b-col md="6" class="my-1">
          <b-pagination :total-rows="totalRows" :per-page="perPage" v-model="currentPage" class="my-0" />
        </b-col>
      </b-row>

      <b-modal 
        ref="modalHapus" hide-footer 
        title="Hapus OPD"
        header-bg-variant="warning"
        >
        Hapus data barang: <strong>{{hapustitle}}</strong>
        <p></p>
        <b-btn class="mr-2" @click="doHapus" variant="danger">Hapus</b-btn>
        <b-btn @click="hideModal" variant="primary">Batal</b-btn>
      </b-modal>

    </b-card>
    </b-row>
    </b-container>
</template>

<script>
import axios from 'axios'
import API from '../../config'

export default {
  name: 'DaftarDemo',
  data () {
    return {
      isBusy: false,
      curItem: null,
      hapustitle: '',
      fields: [
        { key: 'index', label: 'No.' },
        { key: 'judul', label: 'Judul', sortable: true },
        { key: 'deskripsi', label: 'Deskripsi', sortable: true },
        { key: 'aksi', label: '' }
      ],
      hakaksesadmin: false,
      items: [],
      currentPage: 1,
      perPage: 30,
      pageOptions: [ 15, 30, 60, 100 ],
      sortBy: 'nama_item',
      sortDesc: false,
      sortDirection: 'asc',
      totalRows: 0,
      filter: null,
      modalInfo: { title: '', content: '' },
      errors: []
    }
  },
  computed: {
    sortOptions () {
      // Create an options list from our fields
      return this.fields
        .filter(f => f.sortable)
        .map(f => { return { text: f.label, value: f.key } })
    }
  },
  methods: {
    showModalGantiPassword () {
      this.$refs.modalGantiPassword.show()
    },
    showModalHapusUser () {
      
    },
    hideModal (evt) {
      this.$refs.modalHapus.hide()
      this.curItem = null
      this.refreshData()
    },
    formatDate (dt) {
      let tgl = new Date(dt)
      return tgl.toLocaleString()
    },
    resetModal () {
      this.modalInfo.title = ''
      this.modalInfo.content = ''
    },
    onFiltered (filteredItems) {
      // Trigger pagination to update the number of buttons/pages due to filtering
      this.totalRows = filteredItems.length
      this.currentPage = 1
    },
    
    hapus (data) {
      this.curItem = data
      this.hapustitle = this.curItem.judul + ' ?'
      //console.log('trigger hapus user:', data)
      this.$refs.modalHapus.show()
    },
    doHapus () {
      console.log('do hapus item')
      let id = this.curItem.id
      //let headers = this.$store.getters.auth.headers
      //console.log('headers', JSON.stringify(headers))
      // console.log('auth:',auth)
      axios.delete(API.apiURI + 'demo/'+ id)
        .then(
          response => {
            if (response.status !== 200) {
              this.errors.push(response.statusText)
            } else {
              this.curUser = null
              this.hideModal()
              this.refreshData()
            }
          })
        .catch(
          error => {
            this.hideModal()
            console.log('fail error',JSON.stringify(error))
            this.errors.push(error.toString())
          })
    },
    refreshData () {
      //let auth = this.$store.getters.auth
      axios.get(API.apiURI + 'demo')
        .then(
          response => {
            console.log('axios get')
            if (response.status !== 200) {
              //this.errors.push(response.statusText)
            } else {
              this.items = response.data
              //console.log(this.items)
            }
          })
        .catch(
          error => {
            if (error.response) {
              if (error.response.status === 401) {
                this.$store.commit('forcelogout')
              }
              this.errors.push(error.response.statusText)
              return []
            }
            this.errors.push(error)
            return []
          })
    },
    editData (value) {
      console.log(value)
      this.$emit('edit-user', value)
    }
  },
  mounted () {
    this.refreshData()
    //this.hakaksesadmin = this.$store.state.user.admin
    // console.log(this.$store.state.user)
  }
}
</script>
