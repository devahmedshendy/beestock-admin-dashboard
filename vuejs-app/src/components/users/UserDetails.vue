<template>

  <div>
    <div v-if="isLoading">
      <div class="content-wrap" id="content-wrap">
        <main id="content" class="content" role="main">
          <vuestic-pre-loader ref="preLoader" class="pre-loader"></vuestic-pre-loader>
        </main>
      </div>
    </div>

    <div v-if="isError">
      Error
    </div>

    <div class="user-details" v-if="isUserDetails">
      <div class="Set row no-gutters">
        <div class="header col-12">
          <div class="row justify-content-between">
            <div class="header-text col">
              <h2>
                <span class="mr-3">{{ userDetails.fullName }}</span>
                <div class="d-inline-block" v-if="isActiveUser">
                  <span class="badge badge-pill badge-success align-top">{{ 'userDetails.heads.activeUser' | translate }}</span>
                </div>
                <div class="d-inline-block" v-else>
                  <span class="badge badge-pill badge-danger align-top">{{ 'userDetails.heads.inactiveUser' | translate }}</span>
                </div>
              </h2>
              <router-link :to="{path: '/users'}">{{'userDetails.back' | translate}}</router-link>
            </div>

            <div class="col-md-4 mt-2 mr-4 text-right">
              <button class="btn btn-danger btn-micro" style="font-size: 90% !important;" @click="confirmDeleteUser()">
                <div class="btn-with-icon-content">
                  <i class="fa fa-trash"></i>
                  DELETE
                </div>
              </button>
            </div>
          </div>
        </div>
      </div>

      <modal
        :show.sync="showConfirmDeleteUserModal"
        ref="confirmDeleteUserModal"
        :okText="'modal.confirm' | translate"
        :cancelText="'modal.cancel' | translate"
        :okClass="'btn btn-danger'"
        @ok="deleteUser">
        <div slot="title">Confirm Delete User</div>
      </modal>

      <div class="user-settings row">
        <div class="col-md-12">
          <div class="widget">
            <div class="widget-header row no-gutters justify-content-between">
              <div class="col-2 text-left ma-0 pa-0">
                <div>{{ 'userDetails.heads.form'  | translate }}</div>
              </div>

              <div class="text-right" v-if="newChanges">
                <vue-simple-spinner
                  :speed=".8"
                  :line-size="5"
                  :line-fg-color="'#41DF7C'" v-if="savingChanges" >
                </vue-simple-spinner>

                <div style="color: #41DF7C" v-else-if="changesSaved">
                  <span class="align-middle mr-1">SAVED</span>
                  <span aria-hidden="true" class="fa fa-check-circle align-middle" style="font-size: 25px;"></span>
                </div>

                <div style="color: #CE403E" v-else>
                  <span class="align-middle mr-1">FAILED</span>
                  <span aria-hidden="true" class="fa fa-times-circle align-middle" style="font-size: 25px;"></span>
                </div>
              </div>
            </div>

            <div class="widget-body">
              <form>
                <div class="row justify-content-center">
                  <div class="col-md-4">
                    <fieldset>
                      <div class="form-group with-icon-right" :class="{'has-error': errors.has('firstName')}">
                        <div class="input-group">
                          <input
                            id="firstName"
                            name="firstName"
                            type="text"
                            v-model="firstName"
                            v-validate="'required|alpha_spaces|min:3'"
                            data-vv-delay="1"
                            required/>
                          <i class="fa fa-exclamation-triangle error-icon icon-right input-icon"></i>
                          <label class="control-label" for="firstName">{{'userDetails.form.inputs.firstName' | translate}}</label><i class="bar"></i>
                          <small v-show="errors.has('firstName')" class="help text-danger">
                            {{ errors.first('firstName') }}
                          </small>
                        </div>
                      </div>
                    </fieldset>
                  </div>

                  <div class="col-md-4">
                    <fieldset>
                      <div class="form-group with-icon-right" :class="{'has-error': errors.has('lastName')}">
                        <div class="input-group">
                          <input
                            id="lastName"
                            name="lastName"
                            v-model="lastName"
                            v-validate="'required|alpha_spaces|min:3'"
                            required/>
                          <i class="fa fa-exclamation-triangle error-icon icon-right input-icon"></i>
                          <label class="control-label" for="lastName">{{'userDetails.form.inputs.lastName' | translate}}</label>
                          <i class="bar"></i>
                          <small v-show="errors.has('lastName')" class="help text-danger">
                            {{ errors.first('lastName') }}
                          </small>
                        </div>
                      </div>
                    </fieldset>
                  </div>
                </div>

                <div class="row justify-content-center">

                  <div class="col-md-8">
                    <fieldset>
                      <div class="form-group with-icon-right">
                        <div class="input-group">
                          <input
                            id="email"
                            name="email"
                            v-model="email"
                            v-validate="'required|email'"
                            required/>
                          <i class="fa fa-exclamation-triangle error-icon icon-right input-icon"></i>
                          <i class="fa fa-check valid-icon icon-right input-icon"></i>
                          <label class="control-label" for="email">{{'userDetails.form.inputs.email'  | translate}} </label>
                          <i class="bar"></i>
                          <small v-show="errors.has('email')" class="help text-danger">
                            {{ errors.first('email') }}
                          </small>
                        </div>
                      </div>
                    </fieldset>
                  </div>
                </div>

                <div class="row justify-content-center">
                  <div class="col-md-4">
                    <fieldset>
                      <multiselect
                        v-model="country"
                        :options="countryList"
                        :custom-label="nameWithMobileCode"
                        :placeholder="'userDetails.form.selects.country' | translate"
                        label="name"
                        track-by="name">
                      </multiselect>
                    </fieldset>
                  </div>

                  <div class="col-md-4">
                    <fieldset>
                      <div class="form-group">
                        <div class="input-group">
                          <input
                            id="mobileNumber"
                            name="mobileNumber"
                            v-model="mobileNumber"
                            v-validate="'required|numeric'"
                            required/>
                          <i class="fa fa-exclamation-triangle error-icon icon-right input-icon"></i>
                          <i class="fa fa-check valid-icon icon-right input-icon"></i>
                          <label class="control-label" for="mobileNumber">{{'userDetails.form.inputs.mobileNumber.number' | translate}}</label><i class="bar"></i>
                          <i class="bar"></i>
                          <small v-show="errors.has('mobileNumber')" class="help text-danger">
                            {{ errors.first('mobileNumber') }}
                          </small>
                        </div>
                      </div>
                    </fieldset>
                  </div>
                </div>

              </form>
            </div>
          </div>

        </div>
      </div>

      <div class="user-recent-photos row" v-if="userRecentPhotos_500.length > 0">
        <div class="col-md-12">
          <div class="widget">
            <div class="widget-header row no-gutters justify-content-between">
              <div class="col-2 text-left ma-0 pa-0">
                <div>{{ 'userDetails.heads.recentPhotos'  | translate }}</div>
              </div>

              <div class="col-3 text-right">
                <a class="btn btn-primary btn-micro">{{ 'actions.showAll' | translate }}</a>
              </div>
            </div>

            <div class="widget-body">
              <gallery :images="userRecentPhotos_1000" :index="galleryIndex" @close="galleryIndex = null"></gallery>
              <div class="col-md-12">
                <div class="row justify-content-start">
                  <div
                    class="image col-md-4"
                    v-for="(photo, photoIndex) in userRecentPhotos_500"
                    :key="photoIndex"
                    @click="galleryIndex = photoIndex"
                    :style="{ backgroundImage: 'url(' + photo + ')', height: '11rem' }"
                  ></div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>


  </div>
</template>

<script>
import store from '../../store/index'

import VueGallery from 'vue-gallery'
import Spinner from 'vue-simple-spinner'
import Multiselect from 'vue-multiselect'
import CountriesList from './data/country-list'
import Modal from '../vuestic-components/vuestic-modal/VuesticModal'
import VuesticWidget from '../vuestic-components/vuestic-widget/VuesticWidget'
import VuesticPreLoader from '../vuestic-components/vuestic-preloader/VuesticPreLoader'
import VuesticSimpleSelect from '../vuestic-components/vuestic-simple-select/VuesticSimpleSelect'

import userService from '../../services/user/index'
import photoService from '../../services/photo/index'
import routerHelper from '../../helpers/router/index'

const inactiveRole = store.getters.roles.inactive

export default {
  name: 'user-details',
  metaInfo () {
    return {
      title: this.userDetails.fullName ? this.userDetails.fullName : 'User Details'
    }
  },
  components: {
    Modal,
    Multiselect,
    VuesticWidget,
    VuesticPreLoader,
    VuesticSimpleSelect,
    'gallery': VueGallery,
    'vue-simple-spinner': Spinner
  },
  props: {
    uuid: {
      type: String,
      required: true
    }
  },
  data () {
    return {
      isLoading: true,
      isError: false,
      isUserDetails: false,
      showConfirmDeleteUserModal: true,
      firstName: '',
      lastName: '',
      email: '',
      mobileNumber: '',
      country: {},
      isActiveUser: false,
      countryList: CountriesList,
      userRecentPhotos_500: [],
      userRecentPhotos_1000: [],
      userDetails: {
        firstName: '',
        lastName: '',
        fullName: '',
        email: '',
        mobileNumber: '',
        country: {}
      },
      galleryIndex: null,
      newChanges: false,
      savingChanges: false,
      changesSaved: false,
      beforeUpdateTimers: {}
    }
  },
  methods: {
    deleteUser () {
      userService.deleteByUUID(this.uuid)
        .then((response) => {
          routerHelper.deleteUserDone(response.status)
        })
        .catch((error) => {
          routerHelper.deleteUserDone(error.response.status)
        })
    },
    nameWithMobileCode ({ name, mobileCode }) {
      return `${name} — [${mobileCode}]`
    },
    showUserDetails () {
      this.isUserDetails = true
      this.isLoading = false
      this.isError = false
    },
    showErrorAlert () {
      this.isError = true
      this.isLoading = false
      this.isUserDetails = false
    },
    fetchUser (uuid) {
      userService.findByUUID(uuid)
        .then((response) => {
          this.storeUserDetails(response.data.user)
          this.setFormFields()
          this.showUserDetails()
        })
        .catch((error) => {
          routerHelper.findUserFailed(error.response.status)
        })
    },
    setFormFields () {
      this.firstName = this.userDetails.firstName
      this.lastName = this.userDetails.lastName
      this.email = this.userDetails.email
      this.mobileNumber = this.userDetails.mobileNumber
      this.isActiveUser = this.userDetails.isActiveUser
      this.country = this.userDetails.country
    },
    storeUserDetails (user) {
      this.userDetails.firstName = user.first_name
      this.userDetails.lastName = user.last_name
      this.userDetails.fullName = user.full_name
      this.userDetails.email = user.email
      this.userDetails.mobileNumber = user.mobile_number.number
      this.userDetails.isActiveUser = !user.access_info.roles.includes(inactiveRole)

      for (const country of CountriesList) {
        if (user.country === country.abbr) {
          this.userDetails.country.name = country.name
          this.userDetails.country.mobileCode = country.mobileCode
        }
      }
    },
    fetchUserRecentPhotos () {
      photoService.findAll({
        page: 1,
        limit: 5
      })
      .then((response) => {
        this.setUserRecentPhotos(response.data.photos)
      })
      .catch((error) => {
        console.error('ERROR', error)
      })
    },
    setUserRecentPhotos (photos) {
      this.userRecentPhotos_500.length = 0
      for (const photo of Object.values(photos)) {
        this.userRecentPhotos_500.push(photo.file_storage.sizes.size_500)
        this.userRecentPhotos_1000.push(photo.file_storage.sizes.size_1000)
      }
    },
    runSavingSpinner () {
      this.newChanges = true
      this.savingChanges = true
    },
    updateUserWithUpdatedField (fieldName, queryParams = {}) {
      this.stopSavingSpinner()
      this.clearBeforeUpdateTimers(fieldName)

      queryParams[fieldName] = this[fieldName]

      if (this.isFieldUpdatedWithValidValue(fieldName)) {
        this.beforeUpdateTimers[fieldName] = setTimeout(() => {
          this.runSavingSpinner()

          userService.updateByUUID(this.uuid, queryParams)
            .then((repsonse) => {
              this.storeUserDetails(repsonse.data.user)
              this.showChangesSaved()
            })
            .catch((error) => {
              if (error.response.status === 400) {
                this.showFailedToSaveChanges()
              }
            })
        }, 1500)
      }
    },
    stopSavingSpinner () {
      this.newChanges = false
      this.savingChanges = false
    },
    showChangesSaved () {
      this.stopSavingSpinner()
      this.newChanges = true
      this.changesSaved = true
    },
    showFailedToSaveChanges () {
      this.stopSavingSpinner()
      this.newChanges = true
      this.changesSaved = false
    },
    isFieldUpdatedWithValidValue (fieldName) {
      switch (fieldName) {
        case 'firstName':
        case 'lastName':
          this.$validator.validateAll()
          return !this.errors.has(fieldName) && this.userDetails[fieldName] !== this[fieldName]
      }
    },
    clearBeforeUpdateTimers (fieldName) {
      clearTimeout(this.beforeUpdateTimers[fieldName])
    },
    confirmDeleteUser () {
      this.$refs.confirmDeleteUserModal.open()
    }
  },

  watch: {
    '$route': 'fetchUser',
    firstName () {
      this.updateUserWithUpdatedField('firstName')
    },
    lastName () {
      this.updateUserWithUpdatedField('lastName')
    }
  },

  created () {
    this.fetchUser(this.uuid)
    this.fetchUserRecentPhotos()
  }
}
</script>

<style lang="scss" scoped>
  @import "../../sass/variables";
  @import "../../sass/variables";
  @import '../../../node_modules/bootstrap/scss/mixins/breakpoints';
  @import '../../../node_modules/bootstrap/scss/variables';

  .Set{
    .header {
      background-color: white;
      padding: 1.75rem 0 1.125rem 0;

      .header-text {
        text-align: left;
        padding-left: 2.5rem;
        h2 {
          margin-bottom: 0;
        }
      }

      .search {
        display: flex;
        align-items: center;
        justify-content: center;
        .input-group {
          width: 13.25rem;
        }
      }

      .range {
        .vue-slider-wrap {
          width: 9.3rem !important;
          .vue-slider-process {
            background-color: $brand-primary;
          }
          .vue-slider-tooltip {
            background-color: $brand-primary;
            border-color: $brand-primary;
          }
          .vue-slider-dot {
            background-color: $brand-primary;
            box-shadow: none;
          }
        }

        display: flex;
        align-items: center;
        justify-content: center;
        text-align: center;
        h2, h4 {
          margin: .5rem;
        }
      }

      margin-bottom: 2rem;
    }

    .row {
      .icon-grid-container {
        padding-right: .625rem;
        padding-left: .625rem;
        margin: 0 0 .5rem;
        text-align: center;
        height: 6rem;
        position: relative;
        min-height: 1px;
        float: left;

        &:hover {
          background-color: $brand-primary;
          color: white;
          cursor: pointer;
        }
        .icon {
          padding: 1rem 0 0;
          margin: 0 0 .5rem;
          .iconText {
            font-size: .6rem;
            text-align: center;
          }
        }
      }
    }
  }


  .widget {
    background: $widget-bg;
    box-shadow: $widget-box-shadow;
    margin-bottom: $widget-mb;
    padding: 0;

    &.no-header {
      .widget-body {
        height: $widget-body-no-header-height;
      }
    }

    &.larger-padding {
      .widget-body {
        padding: $widget-larger-padding;
      }
    }

    &.no-h-padding {
      .widget-body {
        padding-left: 0;
        padding-right: 0;
      }
    }

    &.no-v-padding {
      .widget-body {
        padding-top: 0;
        padding-bottom: 0;
      }
    }

    &.brand-info {
      background-color: $brand-info;
      color: $white;
      box-shadow: $widget-info-shadow;
    }

    &.brand-danger {
      background-color: $brand-danger;
      color: $white;
      box-shadow: $widget-danger-shadow;
    }

    &.info-widget {
      border-top: $info-widget-border;

      &.brand-info {
        border-top-color: $brand-info;
      }

      &.brand-danger {
        border-top-color: $brand-danger;
      }
    }

    .widget-body {
      height: $widget-body-with-header-height;
      padding: $widget-padding;
    }

    .widget-header {
      height: $widget-header-height;
      padding: 0 $widget-padding;
      border-bottom: $widget-header-border;
      font-size: $font-size-larger;
      font-weight: 600;
      display: flex;
      align-items: center;
    }
  }

  .vue-js-switch#toggle-status {
    font-size: 16px;
    margin-top: .5em;
  }

  .widget-header {
    height: 4rem !important;
  }

  .btn-micro {
    font-size: 65% !important;
  }

  .image {
    /*float: left;*/
    background-size: cover;
    background-repeat: no-repeat;
    background-position: center center;
    border: 1px solid #ebebeb;
    /*margin: 5px;*/
    height: 250px
  }

  .badge {
    min-width: 4rem;
    display: inline-block;
    font-weight: bold;
    text-transform: uppercase;
    font-size: .7rem;
    letter-spacing: .1rem;
    padding: 0.2rem .7rem;
  }

  .btn.btn-micro {
    padding: 0.5rem 1.2rem 0.4rem 1.2rem;
  }
</style>
