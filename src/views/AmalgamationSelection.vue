<template>
  <div id="amalgamation-selection">
    <TechnicalErrorDialog
      :dialog="showErrorDialog"
      attach="#amalgamation-selection"
      @close="showErrorDialog = false"
    />

    <!-- Main Body -->
    <v-container class="view-container">
      <v-row>
        <v-col
          cols="12"
          lg="9"
        >
          <article id="main-article">
            <!-- Page Title -->
            <header>
              <h1>
                Choose the type of amalgamation to be filed.
              </h1>
              <p>
                Choose the appropriate one for your filing.
              </p>
            </header>
          </article>
        </v-col>
      </v-row>

      <v-row>
        <!-- First Choice -->
        <v-col>
          <v-card
            id="start-horizontal-short-form-card"
            flat
            class="pa-8"
          >
            <h2>
              Horizontal short-form amalgamation
            </h2>
            <br>
            <p>
              A <strong>horizontal short-form amalgamation</strong> can be used if the amalgamating
              corporations are all wholly owned subsidiaries of the same holding body corporate.
            </p>
            <p>To complete a short-form amalgamation, the directors of each amalgamating corporation need to:</p>
            <ul>
              <li>Approve the amalgamation</li>
              <li>File articles of amalgamation</li>
            </ul>
            <br>
            <p>Shareholders of the amalgamating corporations do not need to approve the amalgamation.</p>
            <div class="btn-div">
              <v-btn
                id="horizontal-short-form-btn"
                color="primary"
                large
                @click="startAmalgamation(AmalgamationTypes.HORIZONTAL)"
              >
                <strong>Start Horizontal Short-form</strong>
              </v-btn>
            </div>
          </v-card>
        </v-col>

        <!-- Second Choice -->
        <v-col>
          <v-card
            id="start-vertical-short-form-card"
            flat
            class="pa-8"
          >
            <h2>
              Vertical short-form amalgamation
            </h2>
            <br>
            <p>
              A <strong>vertical short-form amalgamation</strong> can be used if the subsidiary
              corporations are wholly owned by the corporation they are amalgamating with.
            </p>
            <p>To complete a short-form amalgamation, the directors of each amalgamating corporation need to:</p>
            <ul>
              <li>Approve the amalgamation</li>
              <li>File articles of amalgamation</li>
            </ul>
            <br>
            <p>Shareholders of the amalgamating corporations do not need to approve the amalgamation.</p>
            <div class="btn-div">
              <v-btn
                id="vertical-short-form-btn"
                color="primary"
                large
                @click="startAmalgamation(AmalgamationTypes.VERTICAL)"
              >
                <strong>Start Vertical Short-form</strong>
              </v-btn>
            </div>
          </v-card>
        </v-col>

        <!-- Third Choice -->
        <v-col>
          <v-card
            id="start-regular-long-form-card"
            flat
            class="pa-8"
          >
            <h2>
              Regular long-form amalgamation
            </h2>
            <br>
            <p>
              If amalgamating corporations don't meet criteria for a short-form amalgamation, they
              need to complete the following steps to amalgamate:
            </p>
            <ul>
              <li>The shareholders of each amalgamating corporation must approve the amalgamation</li>
              <li>The amalgamating corporations must enter into an amalgamation agreement</li>
              <li>
                The amalgamating corporations then file articles of amalgamation along with the
                amalgamation agreement to obtain a certificate of amalgamation
              </li>
            </ul>
            <br>
            <p>
              When the amalgamation is complete, your company will be a
              <strong>{{ getRegularAmalgamationText() }}.</strong>
            </p>
            <div class="btn-div">
              <v-btn
                id="regular-long-form-btn"
                color="primary"
                large
                @click="startAmalgamation(AmalgamationTypes.REGULAR)"
              >
                <strong>Start Regular Long-form</strong>
              </v-btn>
            </div>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script lang="ts">
import { useAuthenticationStore, useBusinessStore, useConfigurationStore, useRootStore } from '@/stores'
import { Action, Getter } from 'pinia-class'
import { Component, Mixins } from 'vue-property-decorator'
import { CorpTypeCd } from '@bcrs-shared-components/corp-type-module'
import { AmalgamationTypes, CorrectNameOptions, FilingTypes } from '@bcrs-shared-components/enums'
import { AmlRoles, AmlTypes } from '@/enums'
import { LegalServices } from '@/services'
import { navigate } from '@/utils'
import { TechnicalErrorDialog } from '@/components/dialogs'
import { CommonMixin } from '@/mixins'

@Component({
  components: {
    TechnicalErrorDialog
  }
})
export default class AmalgamationSelection extends Mixins(CommonMixin) {
  @Getter(useAuthenticationStore) getAccountId!: string
  @Getter(useConfigurationStore) getCreateUrl!: string
  @Getter(useRootStore) getBusinessEmail!: string
  @Getter(useRootStore) getFullPhoneNumber!: string
  // @Getter(useBusinessStore) getIdentifier!: string
  @Getter(useBusinessStore) getLegalName!: string
  @Getter(useBusinessStore) getLegalType!: CorpTypeCd
  @Getter(useBusinessStore) isEntityBcCcc!: boolean
  @Getter(useBusinessStore) isEntityBcCompany!: boolean
  @Getter(useBusinessStore) isEntityBcUlcCompany!: boolean
  @Getter(useBusinessStore) isEntityBenContinueIn!: boolean
  @Getter(useBusinessStore) isEntityBenefitCompany!: boolean
  @Getter(useBusinessStore) isEntityCccContinueIn!: boolean
  @Getter(useBusinessStore) isEntityContinueIn!: boolean
  @Getter(useBusinessStore) isEntityUlcContinueIn!: boolean

  @Action(useRootStore) setStartingAmalgamationSpinner!: (x: boolean) => void

  // enum for template
  readonly AmalgamationTypes = AmalgamationTypes

  // local variable
  showErrorDialog = false

  /** Called when component is created. */
  created (): void {
    // if required data isn't set, go to dashboard
    if (!this.getIdentifier) {
      this.navigateToBusinessDashboard(this.getIdentifier)
    }
  }

  /** The amalmagated company result name depending on the type. */
  getRegularAmalgamationText (): string {
    switch (true) {
      case this.isEntityBcCompany:
      case this.isEntityBenContinueIn:
      case this.isEntityBenefitCompany:
      case this.isEntityContinueIn:
        return 'BC limited company'

      case this.isEntityBcCcc:
      case this.isEntityCccContinueIn:
        return 'BC community contribution company'

      case this.isEntityBcUlcCompany:
      case this.isEntityUlcContinueIn:
        return 'BC unlimited liability company'
    }
    return 'Unknown' // should never happen
  }

  /** Called when Start amalgamtion (regular, horizontal and vertical) button is clicked. */
  async startAmalgamation (amalgamationType: AmalgamationTypes): Promise<any> {
    // Create a draft amalgamation application then redirect to Create UI.
    try {
      // show spinner since this is a network call
      this.setStartingAmalgamationSpinner(true)
      const businessId = await this.createBusinessAA(amalgamationType)
      const route = this.isShortFormAmalgamation(amalgamationType)
        ? 'amalg-short-information'
        : 'amalg-reg-information'
      const amalgamationUrl = `${this.getCreateUrl}${route}?id=${businessId}`
      navigate(amalgamationUrl)
      return
    } catch (error) {
      console.log('Error - unable to amalgamate now =', error)
      this.setStartingAmalgamationSpinner(false)
      this.showErrorDialog = true
    }
  }

  /** The legal type of the new amalgamation filing. */
  get legalType (): CorpTypeCd {
    switch (true) {
      case this.isEntityBcCompany:
      case this.isEntityContinueIn:
        return CorpTypeCd.BC_COMPANY

      case this.isEntityBenefitCompany:
      case this.isEntityBenContinueIn:
        return CorpTypeCd.BENEFIT_COMPANY

      case this.isEntityBcCcc:
      case this.isEntityCccContinueIn:
        return CorpTypeCd.BC_CCC

      case this.isEntityBcUlcCompany:
      case this.isEntityUlcContinueIn:
        return CorpTypeCd.BC_ULC_COMPANY
    }
    throw new Error(`Invalid legal type = ${this.getLegalType}`) // should never happen
  }

  /**
   * Creates a draft amalgamation application.
   * @param type the type of amalgamation to create
   * @returns the business identifier of the newly created amalgamation application
   */
  private async createBusinessAA (type: AmalgamationTypes): Promise<string> {
    // short-form amalgamations use the legal name of the primary business,
    // otherwise don't set a legal name
    const legalName = this.isShortFormAmalgamation(type) ? this.getLegalName : undefined

    // short-form amalgamations adopt the name of the primary business,
    // otherwise don't set a correct name option
    const correctNameOption =
      this.isShortFormAmalgamation(type) ? CorrectNameOptions.CORRECT_AML_ADOPT : undefined

    // short-form amalgamations use the email and phone of the primary business (if they exist),
    // otherwise set empty initial values
    const email = (this.isShortFormAmalgamation(type) && this.getBusinessEmail) || ''
    const phone = (this.isShortFormAmalgamation(type) && this.getFullPhoneNumber) || ''

    const draftAmalgamationApplication = {
      filing: {
        header: {
          name: FilingTypes.AMALGAMATION_APPLICATION,
          accountId: this.getAccountId
        },
        business: {
          legalType: this.legalType
        },
        amalgamationApplication: {
          nameRequest: {
            legalName,
            legalType: this.legalType,
            correctNameOption
          },
          type,
          contactPoint: {
            email,
            phone
          }
        }
      }
    } as any

    // For Horizontal amalgamation, set current business as the Primary business.
    if (type === AmalgamationTypes.HORIZONTAL) {
      draftAmalgamationApplication.filing.amalgamationApplication.amalgamatingBusinesses = [
        {
          type: AmlTypes.LEAR,
          role: AmlRoles.PRIMARY,
          identifier: this.getIdentifier
        }
      ]
    }

    // For Vertical amalgamation, set current business as the Holding business.
    if (type === AmalgamationTypes.VERTICAL) {
      draftAmalgamationApplication.filing.amalgamationApplication.amalgamatingBusinesses = [
        {
          type: AmlTypes.LEAR,
          role: AmlRoles.HOLDING,
          identifier: this.getIdentifier
        }
      ]
    }

    // For Regular amalgamation, set the current business as an amalgamating business.
    if (type === AmalgamationTypes.REGULAR) {
      draftAmalgamationApplication.filing.amalgamationApplication.amalgamatingBusinesses = [
        {
          type: AmlTypes.LEAR,
          role: AmlRoles.AMALGAMATING,
          identifier: this.getIdentifier
        }
      ]
    }

    // create the draft business record
    // (throws an exception on error, which startAmalgamation() will handle)
    const filing = await LegalServices.createDraftBusiness(draftAmalgamationApplication)

    // validate and return the identifier
    const identifier = filing?.business?.identifier as string
    if (!identifier) throw new Error('Invalid business identifier')
    return identifier
  }

  /** Returns True if specified type is a short-form amalgamation. */
  private isShortFormAmalgamation (type: AmalgamationTypes): boolean {
    return (type === AmalgamationTypes.HORIZONTAL || type === AmalgamationTypes.VERTICAL)
  }
}
</script>

<style lang="scss" scoped>
@import '@/assets/styles/theme.scss';

p, ul {
  color: $gray9;
}

h1 {
  margin-bottom: 1.25rem;
  line-height: 2rem;
  letter-spacing: -0.01rem;
}

// Have all the 3 options (cards) with same height.
.v-card {
  height: 100%;
  margin-bottom: 4rem;
  position: relative;
}

// Center and push the buttons to the botton of the cards.
// Keep them centered in the cards regardless of screen size.
.btn-div {
  position: absolute;
  bottom: 2rem;
  margin-left: auto;
  margin-right: auto;
  left: 0;
  right: 0;
  text-align: center;
}
</style>
