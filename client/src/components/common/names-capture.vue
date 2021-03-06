<template>
  <v-form @keydown="validate" id="send-to-examination-form">
    <v-container fluid class="pa-0" id="send-to-examination-container">
      <template v-if="editMode">
        <v-row justify="end" class="mt-n3 mb-n5">
          <v-col cols="6" class="font-weight-bold">I need a name to:</v-col>
          <v-col cols="6" style="display: flex; justify-content: flex-end">
            <button class="link-std"
                    id="help-me-choose-activator"
                    @change="activateHMCModal()">Help Me Choose
            </button>
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="5">
            <v-select :error-messages="errors.includes('request_action_cd') ? 'Please select a type' : ''"
                      :hide-details="!errors.includes('request_action_cd')"
                      :items="requestTypeOptions"
                      @change="clearErrors()"
                      filled
                      id="search-type-options-select"
                      v-model="request_action_cd" />
          </v-col>
          <v-col cols="2">
            <v-select :error-messages="errors.includes('location') ? 'Please select a location' : ''"
                      :hide-details="!errors.includes('location')"
                      :items="locationOptions"
                      filled
                      id="location-options-select"
                      v-model="location" />
          </v-col>
          <v-col cols="5">
            <v-select :error-messages="errors.includes('entity_type_cd') ? 'Please select a type' : ''"
                      :hide-details="!errors.includes('entity_type_cd')"
                      :items="entityTypeOptions"
                      @change="clearErrors()"
                      filled
                      id="entity-type-options-select"
                      v-model="entity_type_cd" />
          </v-col>
        </v-row>
      </template>

      <v-row :class="editMode ? '' : 'mt-3' ">
        <v-col cols="12"
               class="h4 mb-3 ml-n1"
               v-if="editMode">Name Choices
        </v-col>
        <v-col cols="12" v-if="!editMode && isAssumedName" class="text-body-3">
              Name in Home Jurisdiction: {{name}}
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="2" class="py-0 label-style" align-self="start" key="static-1">
          {{choicesLabelsAndHints[0].label}}
        </v-col>
        <transition name="fade" mode="out-in">
          <v-col :key="transitionKey(1)" class="ma-0 pa-0" cols="10">
            <v-row class="ma-0 pa-0" v-if="location === 'BC'">
              <v-col :cols="designationAtEnd ? 8 : 12" class="py-0" >
                <v-text-field :autofocus="autofocusField === 'name1'"
                              :error-messages="messages.name1"
                              :hide-details="hide"
                              :value="nameChoices.name1"
                              @blur="handleBlur()"
                              @input="editChoices('name1', $event, true)"
                              filled
                              id="choice-1-text-field"
                              :label="choicesLabelsAndHints[0].hint"/>
              </v-col>
              <v-col cols="4" class="py-0" v-if="designationAtEnd">
                <v-select :autofocus="autofocusField === 'des1'"
                          :error-messages="des1Message"
                          :hide-details="hide"
                          :items="items"
                          :menu-props="props"
                          :value="nameChoices.designation1"
                          @blur="handleBlur(); showDesignationErrors.des1 = true"
                          @input="editChoices('designation1', $event, true)"
                          filled
                          id="designation-1-select"
                          placeholder="Designation" />
              </v-col>
            </v-row>
            <v-row class="ma-0 pa-0" v-else>
              <v-col :cols="isAssumedName ? 8 : 12" class="py-0" >
                <v-text-field :autofocus="autofocusField === 'name1'"
                              :error-messages="messages.name1"
                              :hide-details="hide"
                              :value="xproNameWithoutConflict"
                              @blur="handleBlur()"
                              @input="editChoices('name1', $event, true)"
                              :filled="isAssumedName"
                              id="choice-1-text-field"
                              :label="choicesLabelsAndHints[0].hint"
                              :disabled="!isAssumedName"/>
              </v-col>
              <v-col cols="4" class="py-0" v-if="isAssumedName">
                <v-select :autofocus="autofocusField === 'des1'"
                          :error-messages="des1Message"
                          :hide-details="hide"
                          :items="items"
                          :menu-props="props"
                          :value="nameChoices.designation1"
                          @blur="handleBlur(); showDesignationErrors.des1 = true"
                          @input="editChoices('designation1', $event, true)"
                          filled
                          id="designation-1-select"
                          placeholder="Designation" />
              </v-col>
            </v-row>
          </v-col>
        </transition>
      </v-row>
      <v-row v-if="!editMode" class="my-1 py-0 colour-text mt-5">
        <v-col cols="2" class="py-0"></v-col>
        <v-col cols="10" class="py-0 text-body-3">
          <span v-if="location!=='BC'">
            <span v-if="isAssumedName">
              You may provide up to two additional assumed names which will be considered at no further cost,
              in the order provided, if your first choice cannot be approved. Be sure to follow
              all <a :href="buildNameURL" target='_blank'>guidelines for how to build a name.</a>
              <v-icon class="launch-icon">mdi-launch</v-icon>
            </span>
            <span v-else>
              You may provide up to two additional assumed names which will be considered at no further cost,
              in the order provided, if the name in the home jurisdiction cannot be approved. Be sure to follow
              all <a :href="buildNameURL" target='_blank'>for how to build a name.</a>
              <v-icon class="launch-icon">mdi-launch</v-icon>
            </span>
          </span>
          <span v-else>
            You may provide up to two additional names which will be considered at no further cost, in the
            order provided, only if your First Choice cannot be approved.`
          </span>
        </v-col>
      </v-row>
      <v-row class="mt-5">
        <v-col cols="2" class="py-0 label-style" align-self="start" key="static-2">
          {{choicesLabelsAndHints[1].label}}
        </v-col>
        <transition name="fade" mode="out-in">
          <v-col :key="transitionKey(2)" class="ma-0 pa-0" cols="10">
            <v-row class="ma-0 pa-0">
              <v-col :cols="designationAtEnd ? 8: 12" class="py-0">
                <v-text-field :autofocus="autofocusField === 'name2'"
                              :error-messages="messages.name2"
                              :hide-details="hide"
                              :value="nameChoices.name2"
                              @blur="handleBlur()"
                              @input="editChoices('name2', $event, true)"
                              filled
                              id="choice-2-text-field"
                              :label="choicesLabelsAndHints[1].hint" />
              </v-col>
              <v-col cols="4" class="py-0" v-if="designationAtEnd">
                <v-select :error-messages="des2Message"
                          :hide-details="hide"
                          :items="items"
                          :menu-props="props"
                          :value="nameChoices.designation2"
                          @blur="handleBlur(); showDesignationErrors.des2 = true"
                          @input="editChoices('designation2', $event, true)"
                          filled
                          id="designation-2-select"
                          placeholder="Designation" />
              </v-col>
            </v-row>
          </v-col>
        </transition>
      </v-row>
      <v-row no gutters class="mt-5" key="static-3">
        <v-col cols="2" class="py-0 label-style" align-self="start">
          {{choicesLabelsAndHints[2].label}}
        </v-col>
        <transition name="fade" mode="out-in">
          <v-col :key="transitionKey(3)" class="ma-0 pa-0" cols="10">
            <v-row class="ma-0 pa-0">
              <v-col :cols="designationAtEnd? 8: 12" class="py-0" style="height:60px">
                <v-text-field :error-messages="messages.name3"
                              :hide-details="hide"
                              :value="nameChoices.name3"
                              @blur="handleBlur()"
                              @input="editChoices('name3', $event)"
                              filled
                              id="choice-3-text-field"
                              :label="choicesLabelsAndHints[2].hint" />
              </v-col>
              <v-col cols="4" class="py-0" style="height: 60px" v-if="designationAtEnd">
                <v-select :error-messages="des3Message"
                          :hide-details="hide"
                          :items="items"
                          :value="nameChoices.designation3"
                          @blur="handleBlur(); showDesignationErrors.des3 = true"
                          @input="editChoices('designation3', $event, true)"
                          filled
                          id="designation-3-select"
                          placeholder="Designation" />
              </v-col>
            </v-row>
          </v-col>
        </transition>
      </v-row>

      <v-row class="mt-5">
        <v-col cols="7" class="py-0" />
        <ApplicantInfoNav :isValid="isValid" />
      </v-row>
    </v-container>
  </v-form>
</template>

<script lang="ts">
import ApplicantInfoNav from '@/components/common/applicant-info-nav.vue'
import { LocationT } from '@/models'
import { sanitizeName } from '@/plugins/utilities'
import newReqModule from '@/store/new-request-module'
import { Component, Vue, Watch } from 'vue-property-decorator'

@Component({
  components: {
    ApplicantInfoNav
  }
})
export default class NamesCapture extends Vue {
  hide: boolean | 'auto' = true
  messages = {
    des1: '',
    des2: '',
    des3: '',
    name1: '',
    name2: '',
    name3: ''
  }
  props = {
    disableKeys: false
  }
  showDesignationErrors = {
    des1: false,
    des2: false,
    des3: false
  }

  mounted () {
    this.$el.addEventListener('keydown', this.handleKeydown)
    newReqModule.mutateNameChoicesToInitialState()
    if (this.isAssumedName && !this.editMode) {
      this.$nextTick(function () { this.hide = true })
      return
    }
    this.$nextTick(function () {
      if (this.editMode) {
        this.populateNames()
        return
      }
      newReqModule.mutateSubmissionType('examination')
      if (this.designationAtEnd) {
        for (let item of this.items) {
          let { name } = this
          if ([' LTD', ' INC', ' CORP'].some(des => name.endsWith(des))) {
            name = name + '.'
          }
          if (item) {
            if (name.endsWith(item)) {
              newReqModule.mutateNameChoices({ key: 'designation1', value: item })
              let value = name.replace(item, '').trim()
              newReqModule.mutateNameChoices({ key: 'name1', value })
              return
            }
          }
        }
      }
      newReqModule.mutateNameChoices({ key: 'name1', value: this.name })
    })
  }
  destroyed () {
    this.$el.removeEventListener('keydown', this.handleKeydown)
  }

  @Watch('request_action_cd')
  updateLocationOnAssumedName (newVal, oldVal) {
    if (newVal === 'ASSUMED') {
      if (this.location === 'BC') this.location = 'CA'
    }
  }
  @Watch('entity_type_cd')
  handleEntityType (newVal, oldVal) {
    if (newVal === 'INFO') {
      newReqModule.mutatePickEntityModalVisible(true)
      this.entity_type_cd = oldVal
    }
  }
  @Watch('location')
  handleLocation (newVal, oldVal) {
    if (newVal !== oldVal) {
      newReqModule.mutateNRData({ key: 'xproJurisdiction', value: '' })
    }
    if (newVal === 'INFO') {
      let type = this.entity_type_cd
      newReqModule.mutateLocationInfoModalVisible(true)
      this.$nextTick(function () {
        this.location = oldVal
        this.entity_type_cd = type
      })
    } else {
      this.$nextTick(function () {
        if (this.editMode) {
          this.populateNames()
        }
      })
    }
  }

  get autofocusField () {
    if (this.isAssumedName) {
      return 'name1'
    }
    let output = 'name2'
    if (this.designationAtEnd) {
      if (!this.messages.name1 && !this.messages.des1) {
        return output
      }
      if (this.messages.des1) {
        output = 'des1'
      }
      if (this.messages.name1) {
        output = 'name1'
      }
      return output
    }
    if (this.messages.name1) {
      output = 'name1'
    }
    return output
  }
  get des1Message (): string {
    if (this.showDesignationErrors.des1) {
      return this.messages.des1
    }
    return ''
  }
  get des2Message (): string {
    if (this.showDesignationErrors.des2) {
      return this.messages.des2
    }
    return ''
  }
  get des3Message (): string {
    if (this.showDesignationErrors.des3) {
      return this.messages.des3
    }
    return ''
  }
  get designationAtEnd () {
    if (this.entity_type_cd && this.$designations[this.entity_type_cd]) {
      return this.$designations[this.entity_type_cd].end
    }
    return false
  }

  get choicesLabelsAndHints () {
    if (this.location === 'BC') {
      return [
        {
          'label': 'First Choice',
          'hint': ''
        },
        {
          'label': 'Second Choice',
          'hint': 'Second Alternate Name (Optional)'
        },
        {
          'label': 'Third Choice',
          'hint': 'Third Alternate Name (Optional)'
        }
      ]
    } else {
      if (this.isAssumedName) {
        return [
          {
            'label': 'Assumed Name First Choice',
            'hint': 'Enter your first choice for an assumed name'
          },
          {
            'label': 'Assumed Name Second Choice',
            'hint': 'Enter your second choice for an assumed name (Optional)'
          },
          {
            'label': 'Assumed Name Third Choice',
            'hint': 'Enter your third choice for an assumed name (Optional)'
          }
        ]
      } else {
        return [
          {
            'label': 'Name in Home Jurisdiction',
            'hint': ''
          },
          {
            'label': 'Assumed Name First Choice',
            'hint': 'Enter your first choice for an assumed name (Optional)'
          },
          {
            'label': 'Assumed Name Second Choice',
            'hint': 'Enter your second choice for an assumed name (Optional)'
          }
        ]
      }
    }
  }

  get editMode () {
    return newReqModule.editMode
  }
  get entityPhraseChoices () {
    if (!this.entity_type_cd || !this.$designations[this.entity_type_cd]) {
      return []
    }
    let basePhrases = this.$designations[this.entity_type_cd].words
    // these are the inner phrases for the CCC and CP types.  Filtering out CR designations from CPs has no effect
    // and CCC designations are a mix of CR-type ending designations and CCC specific inner phrases so filter out
    // the CR designations for the purposes of this getter
    return basePhrases.filter(phrase => !this.$designations['CR'].words.includes(phrase))
  }
  get entityPhraseRequired () {
    if (!this.entity_type_cd) return false
    return ['CC', 'CP'].includes(this.entity_type_cd)
  }
  get entityPhraseText () {
    return this.entityPhraseChoices.join(', ')
  }
  get entity_type_cd () {
    return newReqModule.entity_type_cd
  }
  get entityTypeText () {
    return (this.entity_type_cd === 'CC') ? 'Community Contribution Company' : 'Cooperative'
  }
  get entityTypeOptions () {
    return newReqModule.entityTypeOptions
  }
  get errors () {
    return newReqModule.errors
  }
  get isAssumedName () {
    return newReqModule.isAssumedName
  }
  get isValid () {
    let { nameChoices, messages, designationAtEnd, validatePhrases, location, isAssumedName } = this
    if (this.isAssumedName && this.editMode) {
      if (!nameChoices['name1']) {
        messages['name1'] = 'Please enter at least one name'
        return false
      }
      messages['name1'] = ''
      return true
    }
    if (this.isAssumedName) {
      if (!nameChoices['name1']) {
        if (!nameChoices['name2'] && !nameChoices['name3']) {
          return false
        }
        if (nameChoices['name2'] || nameChoices['name3']) {
          messages['name1'] = 'Please enter a first choice before any subsequent choices'
          return false
        }
      }
      if (designationAtEnd && !nameChoices.designation1) {
        return false
      }
      let outcome = true
      for (let choice of [1, 2, 3]) {
        if (nameChoices[`name${choice}`] === this.name) {
          messages[`name${choice}`] = 'This is identical to the name your originally entered.  Please enter a new name.'
          this.hide = 'auto'
          outcome = false
        }
      }
      return outcome
    }

    function name1 () {
      messages.name1 = ''
      messages.des1 = ''

      validatePhrases('name1')
      if (!nameChoices.name1) {
        if (nameChoices.name2 || nameChoices.name3) {
          messages.name1 = 'Please enter a first choice before any subsequent choices'
        } else {
          messages.name1 = 'Please enter at least one name'
        }
      }
      if (location === 'BC') {
        if (designationAtEnd && !nameChoices.designation1) {
          messages.des1 = 'Please choose a designation'
        }
      }
      if (messages.name1 || messages.des1) {
        return false
      }
      return true
    }
    function name2 () {
      messages.name2 = ''
      messages.des2 = ''

      if (!nameChoices.name2) {
        return true
      }
      validatePhrases('name2')
      if (nameChoices.name2 === nameChoices.name1) {
        messages.name2 = 'This is identical to your first name choice.  Please enter a unique name'
      }
      if (designationAtEnd && !nameChoices.designation2) {
        messages.des2 = 'Please choose a designation'
      }
      if (messages.name2 || messages.des2) {
        return false
      }
      return true
    }
    function name3 () {
      messages.name3 = ''
      messages.des3 = ''
      if (!nameChoices.name3) {
        return true
      }
      if (!nameChoices.name2) {
        messages.name3 = 'Please choose a second name before entering a third name'
        return false
      }
      validatePhrases('name3')
      if (nameChoices.name3 === nameChoices.name1) {
        messages.name3 = 'This is identical to your first name choice.  Please enter a unique name'
      }
      if (nameChoices.name3 === nameChoices.name2) {
        messages.name3 = 'This is identical to your second name choice.  Please enter a unique name'
      }
      if (designationAtEnd && !nameChoices.designation3) {
        messages.des3 = 'Please choose a designation'
      }
      if (messages.name3 || messages.des3) {
        return false
      }
      return true
    }

    if (this.editMode) {
      let outcome = true
      if (designationAtEnd) {
        if (!name1() || !name2() || !name3()) {
          outcome = false
        } else {
          for (let choice of [1, 2, 3]) {
            if (nameChoices[`name${choice}`]) {
              if (!nameChoices[`designation${choice}`]) {
                if (location === 'BC' || isAssumedName) {
                  messages[`des${choice}`] = 'Please choose a designation'
                  this.showDesignationErrors[`des${choice}`] = true
                  outcome = false
                }
              } else {
                this.messages[`des${choice}`] = ''
              }
            }
          }
        }
        if (!outcome) {
          this.hide = 'auto'
        }
      }
      return outcome
    }

    let step1 = name1()
    let step2 = name2()
    let step3 = name3()
    this.$nextTick(function () {
      if (!(step1 && step2 && step3)) {
        this.hide = 'auto'
      }
    })
    return (step1 && step2 && step3)
  }
  get items () {
    let output: string[] = this.$designations[this.entity_type_cd].words
    if (this.entity_type_cd === 'CC') {
      output = this.$designations['CR'].words
    }
    output.unshift('')
    return output
  }
  get location () {
    return newReqModule.location
  }
  get locationOptions () {
    return newReqModule.locationOptions
  }
  get name () {
    return newReqModule.name
  }
  get nameChoices () {
    return newReqModule.nameChoices
  }
  get nr () {
    return newReqModule.nr
  }
  get request_action_cd () {
    return newReqModule.request_action_cd
  }
  get requestTypeOptions () {
    return newReqModule.requestTypeOptions
  }
  get xproNameWithoutConflict () {
    var name = this.nameChoices.name1
    if (!this.isAssumedName && this.nameChoices.designation1) {
      name = `${name} ${this.nameChoices.designation1}`
    }
    return name
  }
  get buildNameURL () {
    return 'https://www2.gov.bc.ca/gov/content/employment-business/business/managing-a-business/' +
    'permits-licences/businesses-incorporated-companies/approval-business-name/how-to-name-business'
  }
  set entity_type_cd (type: string) {
    newReqModule.mutateEntityType(type)
  }
  set location (location: LocationT) {
    newReqModule.mutateLocation(location)
  }
  set request_action_cd (value: string) {
    newReqModule.mutateRequestAction(value)
    if (value === 'INFO') {
      newReqModule.mutatePickRequestTypeModalVisible(true)
    }
  }

  activateHMCModal () {
    newReqModule.mutateHelpMeChooseModalVisible(true)
  }
  activateNRRModal () {
    newReqModule.mutateNrRequiredModalVisible(true)
  }
  autoCapitalize (key: string) {
    let value = sanitizeName(this.nameChoices[key])
    newReqModule.mutateNameChoices({ key, value })
  }
  clearErrors () {
    newReqModule.clearErrors()
  }
  editChoices (key, value, userInitiated = false) {
    if (userInitiated) {
      this.hide = true
      for (let key in this.messages) {
        this.messages[key] = ''
      }
    }
    newReqModule.mutateNameChoices({ key, value })
  }
  handleBlur () {
    for (let key in this.nameChoices) {
      let value = this.nameChoices[key] ? sanitizeName(this.nameChoices[key]) : ''
      this.editChoices(key, value)
    }
    this.validate()
  }
  handleKeydown (event) {
    if (event.key === 'Enter') {
      this.props.disableKeys = true
      this.validateButton()
    }
  }
  populateNames () {
    let { nr } = this
    for (let name of nr.names) {
      let { choice } = name
      if (name.designation) {
        newReqModule.mutateNameChoices({ key: `designation${choice}`, value: name.designation })
      }
      newReqModule.mutateNameChoices({ key: `name${choice}`, value: name.name })
    }
    const { nameChoices } = this
    if (nr && nr.names && Array.isArray(nr.names)) {
      for (let choice of [1, 2, 3]) {
        if (nr.names.find(name => name.choice === choice)) {
          let { name } = nr.names.find(name => name.choice === choice)
          if (name.designation && name.name) {
            newReqModule.mutateNameChoices({ key: `name${choice}`, value: name.name })
            newReqModule.mutateNameChoices({ key: `designation${choice}`, value: name.designation })
            continue
          }
          if (this.designationAtEnd) {
            for (let item of this.items) {
              if ([' LTD', ' INC', ' CORP'].some(des => name.endsWith(des))) {
                name = name + '.'
              }
              if (item) {
                if (name.endsWith(item)) {
                  newReqModule.mutateNameChoices({ key: `designation${choice}`, value: item })
                  let value = name.replace(item, '').trim()
                  newReqModule.mutateNameChoices({ key: `name${choice}`, value })
                }
              }
            }
            if (!nameChoices[`name${choice}`]) {
              newReqModule.mutateNameChoices({ key: `name${choice}`, value: name })
            }
          } else {
            newReqModule.mutateNameChoices({ key: `name${choice}`, value: name })
          }
        }
        if (this.designationAtEnd) {
          if (nameChoices[`name${choice}`] && nameChoices[`designation${choice}`]) {
            if (nameChoices[`name${choice}`].endsWith(' ' + nameChoices[`designation${choice}`])) {
              let newName = nameChoices[`name${choice}`].replace(nameChoices[`designation${choice}`], '').trim()
              newReqModule.mutateNameChoices({ key: `name${choice}`, value: newName })
            }
          }
        } else {
          if (this.$designations[this.entity_type_cd].end && nameChoices[`designation${choice}`]) {
            if (!nameChoices[`name${choice}`].endsWith(nameChoices[`designation${choice}`])) {
              let newName = nameChoices[`name${choice}`] + ' ' + nameChoices[`designation${choice}`]
              newReqModule.mutateNameChoices({ key: `name${choice}`, value: newName })
            }
          }
        }
      }
    }
  }
  showNextTab () {
    newReqModule.mutateSubmissionTabComponent('ApplicantInfo1')
  }
  transitionKey (i) {
    if (this.designationAtEnd) {
      return `transition-state-1-${i}`
    }
    return `transition-state-2-${i}`
  }
  validate (next = false) {
    if (!this.isValid) {
      this.hide = 'auto'
      this.props.disableKeys = false
      return
    }
    this.hide = true
    if (next) {
      this.showNextTab()
    }
  }
  validateButton () {
    for (let key in this.showDesignationErrors) {
      this.showDesignationErrors[key] = true
    }
    this.validate(true)
  }
  validatePhrases (choice: string) {
    if (this.entityPhraseRequired) {
      const name = this.nameChoices[choice].toUpperCase()
      if (this.entityPhraseChoices.every(phrase => {
        phrase = phrase.replace(/[.*+\-?^${}()|[\]\\]/g, '\\$&')
        return (name.search(new RegExp('(\\s)' + phrase + '(\\s|$)')) === -1)
      })) {
        this.messages[choice] = `A ${this.entityTypeText} name must include (but not start with)
          one of the following phrases: ${this.entityPhraseText}`
      }
      if (this.entityPhraseChoices.some(phrase => name.startsWith(phrase))) {
        this.messages[choice] = `A ${this.entityTypeText} name must not begin with one of the
          following phrases: ${this.entityPhraseText}`
      }
    }
  }
}
</script>

<style lang="scss" scoped>
@import "@/assets/scss/theme.scss";

::v-deep .v-messages__message {
  line-height: 14px !important;
}

.label-style {
  font-size: 1rem;
  font-weight: bold;
  color: $gray9;
}

.launch-icon {
  font-size: 1rem;
  color: $app-blue;
}
</style>
