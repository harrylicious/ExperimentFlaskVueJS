<template>
  <div class="animated fadeIn">
  
    <b-row>
      <b-col lg="12">
        <transition name="fade">
          <b-card no-body v-if="show">
            <div slot="header">
              <i class="fa fa-edit"></i> Daftar Data
              <div class="card-header-actions">
                <b-link href="#" class="card-header-action btn-setting">
                  <i class="icon-settings"></i>
                </b-link>
                <b-link class="card-header-action btn-minimize" v-b-toggle.collapse1>
                  <i class="icon-arrow-up"></i>
                </b-link>
                <b-link href="#" class="card-header-action btn-close" v-on:click="show = !show">
                  <i class="icon-close"></i>
                </b-link>
              </div>
            </div>
            <b-collapse id="collapse1" visible>
              <b-card-body>
                <b-form-group label="Judul:" label-for="elementsEmail" description="Here's some help text">
                  <b-input-group>
                    <b-form-input id="judul" type="text" v-model="judul"></b-form-input>
                  </b-input-group>
                </b-form-group>
                <b-form-group label="Deskripsi:" label-for="elementsAppend" description="Here's some help text">
                  <b-input-group>
                    <b-form-input id="deskripsi" type="text" v-model="deskripsi"></b-form-input>
                    </b-input-group>
                </b-form-group>
                
               
                <div class="form-actions">
                  <b-button type="submit" variant="primary" v-on:click="insertNewItem()">Save</b-button>
                  <b-button type="button" variant="secondary">Cancel</b-button>
                </div>
              </b-card-body>
            </b-collapse>
          </b-card>
        </transition>
      </b-col>
    </b-row>
    <b-modal 
      ref="modalSuccess" hide-footer 
      title="Sukses"
      header-bg-variant="info"
      >
      <div class="d-block text-center">
        Data berhasil disimpan
      </div>
      <b-btn class="mt-3" block @click="hideModal" variant="info">OK</b-btn>
    </b-modal>
  </div>
</template>

<script>
import axios from 'axios'
import API from '../../config'
export default {
  name: 'forms',
  data () {
    return {
      judul:'',
      deskripsi:'',
      selected: [], // Must be an array reference!
      show: true
    }
  },
   methods: {
    showModal () {
      this.$refs.modalSuccess.show()
    },
    hideModal (evt) {
      this.$refs.modalSuccess.hide()
      if (this.$route.params.visid=0) {
        this.prepareNewEntry(evt)
      }
      this.$router.push('/tambah/1')
    },
    postData (evt) {
      evt.preventDefault()
      let payload = {
        judul: this.judul,
        deskripsi: this.deskripsi,
        card_id: this.cardid
      }
      if (this.$route.params.visid>0){
        this.updateOPD(payload)
      } else {
        this.insertNewItem(payload)
      }
    },
    insertNewItem () {
      //let auth = this.$store.getters.auth
       let payload = {
        'judul': this.judul,
        'deskripsi': this.deskripsi
      }
      console.log(payload)
      axios.post(API.apiURI + 'demo', payload) 
        .then(
          response => {
            if (response.status !== 200) {
              this.errors.push(response.statusText)
            } else {
              this.showModal()
            }
          })
        .catch(
          error => {
            console.log(JSON.stringify(error))
            this.errors.push(error)
          })
    },

    updateOPD (payload) {
      let auth = this.$store.getters.auth
      axios.put(API.apiURI + 'fdataset', payload, auth)
        .then(
          response => {
            if (response.status !== 200) {
              this.errors.push(response.statusText)
            } else {
              this.showModal()
            }
          })
        .catch(
          error => {
            console.log(JSON.stringify(error))
            this.errors.push(error)
          })
    },
    prepareNewEntry (evt) {
      evt.preventDefault()
      // console.log('new entry clicked')
      this.success = false
      this.errors = []
      this.postresult = null
      // main attribute
      this.judul = ''
      this.deksripsi = ''
      this.cardid = ''
      this.$route.params.visid = '0'  
      
      // trick
      this.showForm = false
      this.$nextTick(() => { this.showForm = true })
    },
    trigger () {
      this.$nextTick(function () {
        //console.log('doodl') // => 'updated'
      })
    },
    
    loadData () {
    console.log('load opd')
    axios.get(API.apiURI + 'demo/' + this.$route.params.id)
      .then(
        response => {
          if (response.status !== 200) {
            this.errors.push(response.statusText)
          } else {
            let item = response.data
            this.judul = item.judul
            this.deskripsi = item.deskripsi
            this.itemloaded = true
          }
        })
      .catch(
        error => {
          // console.log(JSON.stringify(error))
          this.errors.push(error)
          return []
        })
    },
    updateOPD (payload) {
      //let auth = this.$store.getters.auth
      axios.put(API.apiURI + 'demo', payload)
        .then(
          response => {
            if (response.status !== 200) {
              this.errors.push(response.statusText)
            } else {
              this.showModal()
            }
          })
        .catch(
          error => {
            console.log(JSON.stringify(error))
            this.errors.push(error)
          })
    },

  },
   
  mounted () {
     if (this.$route.params.id>0) {
      this.loadData()
    }
    
  }
}
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}
</style>