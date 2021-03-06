<template>
  <MainContainer id="analyze-results-container">
    <template v-slot:container-header>
      <v-col cols="auto" class="font-weight-bold h6 py-0 mt-1">
        You are searching for a name for a
        {{ entityText === ' BC Corporation' && location.text === ' BC' ? '' : ' ' + location.text }}
        {{ entityText }}
      </v-col>
    </template>
    <template v-slot:content>
      <v-row class="mt-5">
        <v-col cols="12" class="pb-0" @click="clickNameField">
          <quill-editor :contents="contents"
                        :options="config"
                        :disabled="!!finalName || isApproved"
                        @change="handleChange($event)"
                        @keydown.native.capture="handleEnterKey"
                        id="name-search-bar"
                        ref="quill" />
          <div class="search-icon">
            <v-tooltip bottom
                       content-class="bottom-tooltip search-tooltip"
                       transition="fade-transition">
              <template v-slot:activator="scope">
                <v-icon @click.capture.stop="handleSubmit"
                        class="name-search-icon"
                        color="primary"
                        id="name-input-icon"
                        :disabled="isApproved"
                        v-on="scope.on">mdi-magnify</v-icon>
              </template>
              Search Again
            </v-tooltip>
          </div>
        </v-col>
      </v-row>

      <transition name="fade" mode="out-in" :duration="{ enter: 200, leave: 200 }">
        <v-row no-gutters :key="issueIndex+'vcol'">
          <v-col>

            <!--"FURTHER ACTION REQUIRED" OR "APPROVABLE" TEXT + ICON-->
            <transition name="fade" mode="out-in" >
              <v-row no-gutters justify="center"
                     class="mt-n4"
                     :key="headerProps.text">
                <v-col cols="auto" :class="headerProps.class" class="h4">
                  <v-icon :class="headerProps.class">
                    {{ headerProps.icon }}
                  </v-icon>
                  {{ headerProps.text }}
                </v-col>
              </v-row>
            </transition>

            <!--ISSUE_TYPE: FURTHER ACTION REQUIRED-->
            <template v-if="issue && issue.issue_type">
              <!--MAIN HEADINGS / INFO: LINE 1 & LINE 2-->
              <transition name="fade" mode="out-in">
                <v-row no-gutters justify="center"
                       class="py-1"
                       v-if="headerProps.showNextLines"
                       :key="headerProps.text">
                  <v-col class="pt-2 pb-4 mt-n1 text-center"
                         cols="12"
                         v-html="issue.line1"
                         v-if="issue.line1" />
                  <v-col class="mt-n3 pb-4 text-center"
                         cols="12"
                         v-html="issue.line2"
                         v-if="issue.line2" />
                </v-row>
                <v-row v-else>
                  <v-col class="pt-2 pb-4 mt-n1 text-center"
                         cols="12">
                    You have fixed all the issues.
                  </v-col>
                </v-row>
              </transition>

              <!--CORP CONFLICT TABLE-->
              <v-row no-gutters justify="center" v-if="conflicts.length > 0" class="mt-n3 pb-4 text-center">
                <v-col cols="auto">
                  <div v-for="(corp, n) in conflicts" :key="'conflict-' + n">
                    <b>{{ corp.name }}</b><br>
                    <span v-if="conflictDate && conflictId.startsWith('NR ')">
                      <b>Submitted Date:</b> {{ conflictDate + ', ' }}</span>
                    <span v-if="conflictDate && !conflictId.startsWith('NR ')">
                      <b>Incorporation Date:</b> {{ conflictDate + ', ' }}</span>
                    <span v-if="conflictId && !conflictId.startsWith('NR ')">
                      <b>Corporation No.:</b> {{ conflictId }}</span>
                  </div>
                </v-col>
              </v-row>

              <!--GREY BOXES-->
              <transition name="fade" mode="out-in" >
                <v-row :key="'grey-box-row' + showGreyBoxes"
                       class="colour-p-blue-text justify-center"
                       dense
                       v-if="showGreyBoxes">
                  <v-col :key="issue.issue_type + '-' + option.header + '-' + optionIndex"
                         v-for="(option, optionIndex) of issue.setup">
                    <GreyBox :changesInBaseName="changesInBaseName"
                             :designationIsFixed="designationIsFixed"
                             :finalName="finalName"
                             :i="optionIndex"
                             :issueIndex="issueIndex"
                             :option="option"
                             :originalName="originalName" />
                  </v-col>
                </v-row>
              </transition>

              <!--SUBMISSION BUTTON-->
              <v-row justify="center" v-if="issue.show_examination_button || issue.show_reserve_button">
                <v-col cols="auto" class="pb-0">
                  <ReserveSubmit
                    :id="issue.show_examination_button ? 'reserve-submit-examine' : 'reserve-submit-normal'"
                    :setup="issue.show_examination_button ? 'examine' : ''"
                  />
                </v-col>
              </v-row>

              <!--ERROR MESSAGE / NEXT - PREVIOUS BUTTONS-->
              <v-row v-if="json.issues.length > 1" no-gutters class="mt-5">
                <!-- left third -->
                <v-col cols="4" class="text-left align-self-center">
                  <div v-if="highlightCheckboxes" class="copy-small error-message">
                    Select an option above to continue
                  </div>
                </v-col>
                <!-- middle third -->
                <v-col cols="4" class="text-center align-self-center">
                  <span class="copy-small">Viewing {{ issueIndex + 1 }} of {{ json.issues.length }} Issues</span>
                </v-col>
                <!-- right third -->
                <v-col cols="4" class="text-right align-self-center">
                  <v-btn @click="issueIndex--"
                         class="rnd-wht-btn mr-0"
                         color="#1669bb"
                         id="previous-issue-btn"
                         large
                         outlined
                         v-if="issueIndex > 0">Previous Issue
                  </v-btn>
                  <v-btn :class="nextButtonDisabled ? 'disabled-issue-btn' : 'active-issue-btn'"
                         @click="clickNext"
                         class="mr-0"
                         id="next-issue-btn"
                         large
                         outlined
                         v-if="(json.issues.length - 1) > issueIndex">Next Issue
                  </v-btn>
                </v-col>
              </v-row>
            </template>

            <!--APPROVABLE NAME, NO ISSUES-->
            <template v-else>

              <!--APPROVED TEXT-->
              <v-row no-gutters justify="center">
                <v-col cols="12" class="copy-normal pt-2">
                  <v-row justify="center">
                    <v-col cols="auto">
                     This name has passed initial checks and is ready to send to staff for a final decision.
                    </v-col>
                  </v-row>
                  <v-row justify="center">
                    <v-col cols="auto" class="pb-0">
                      <ReserveSubmit id="reserve-submit-normal" />
                    </v-col>
                  </v-row>
                </v-col>
              </v-row>
            </template>
          </v-col>
        </v-row>
      </transition>
    </template>
  </MainContainer>
</template>

<script lang="ts">
import GreyBox from '@/components/new-request/grey-box.vue'
import MainContainer from '@/components/new-request/main-container.vue'
import allDesignations, { allDesignationsList } from '@/store/list-data/designations'
import Moment from 'moment'
import newReqModule from '@/store/new-request-module'
import ReserveSubmit from '@/components/new-request/submit-request/reserve-submit.vue'
import { Component, Vue, Watch } from 'vue-property-decorator'
import { IssueI, QuillOpsI, SelectionI } from '@/models'
import { quillEditor } from 'vue-quill-editor'
import { matchWord, removeExcessSpaces, replaceWord } from '@/plugins/utilities'

@Component({
  components: { GreyBox, MainContainer, quillEditor, ReserveSubmit }
})
export default class AnalyzeResults extends Vue {
  config = {
    modules: {
      toolbar: false
    },
    placeholder: '',
    scrollingContainer: false
  }
  contents: string = ''
  finalName: string = ''
  highlightCheckboxes: boolean = false
  issueIndex: number = 0
  originalName: string = ''
  originalOps = []

  created () {
    this.originalName = newReqModule.name
  }
  mounted () {
    this.$root.$on('updatecontents', (name) => { this.updateContents(name) })
    this.$root.$on('show-original-name', () => { this.name = this.originalName })
    document.addEventListener('keydown', this.handleEnterKey)
    this.$nextTick(function () {
      this.quill.setContents([])
      let ops: QuillOpsI[] = this.getBaseNameOps()
      if (!this.hasNameActions) {
        this.quill.setContents(ops)
        this.originalOps = ops
        return
      }
      let { length } = this.chunkedName
      this.nameActions.forEach(action => {
        if (action.index < length) {
          if (action.type === 'brackets') {
            let op: QuillOpsI = { insert: ` [${action.message}]`, attributes: { color: 'red' } }
            if (action.position === 'start') {
              ops.splice(action.index, 0, op)
            } else if (action.position === 'end') {
              ops.splice(action.index + 1, 0, op)
            }
          }
          if (action.type === 'highlight') {
            ops[action.index] = { ...ops[action.index], attributes: { color: 'red' } }
          }
          if (action.type === 'strike') {
            ops[action.index] = { ...ops[action.index], attributes: { color: 'red', strike: true } }
          }
        }
      })
      this.quill.setContents(ops)
      this.originalOps = ops
    })
  }
  beforeDestroy () {
    document.removeEventListener('keydown', this.handleEnterKey)
  }

  @Watch('issueIndex')
  resetShowActualInput (newVal, oldVal) {
    if (newVal !== oldVal) {
      if (this.name !== this.finalName) {
        this.showActualInput = false
      }
    }
    if (newVal === this.issueLength - 1) {
      let keys = Object.keys(this.requestExaminationOrProvideConsent[newVal])
      keys.forEach(key => {
        newReqModule.mutateRequestExaminationOrProvideConsent({
          value: false,
          type: key,
          index: newVal
        })
      })
    }
  }
  @Watch('nameIsFixed')
  updateFinalName (newVal) {
    if (newVal) {
      this.finalName = this.name
      this.showActualInput = true
    }
  }

  get allDesignationsStripped () {
    return this.stripAllDesignations(this.originalName)
  }
  get allowProceed () {
    /* newReqModule.designationIssueTypes is a list of all the designation-related issue types.  designationIsFixed
     will not be true unless one of these issues was solved.  some sets of issues will not include a designation-related
     issue and this will need to return true in these cases even tho designationIsFixed will be false */
    if (!this.hasDesignationIssue && this.isLastIndex) {
      return true
    }
    return this.designationIsFixed
  }
  get baseWordsAreUnchanged () {
    let nameTest = this.stripAllDesignations(this.name)
    let { allDesignationsStripped } = this
    if (this.nameActionWords.length > 0) {
      for (let word of this.nameActionWords) {
        nameTest = replaceWord(nameTest, word)
        allDesignationsStripped = replaceWord(allDesignationsStripped, word)
      }
    }
    return (allDesignationsStripped === nameTest)
  }
  get changesInBaseName () {
    if (this.finalName && this.name === this.finalName) return false
    if (this.name === this.originalName) {
      return false
    }
    return !this.baseWordsAreUnchanged
  }
  get chunkedName () {
    return this.name.split(' ')
  }
  get conflictDate () {
    if (Array.isArray(this.issue.conflicts) && this.issue.conflicts.length >= 1) {
      if (this.issue.conflicts[0].source === 'corp') {
        return Moment(this.issue.conflicts[0].start_date).utc().local().format('MMMM Do YYYY')
      }
    }
    return null
  }
  get conflictId () {
    if (Array.isArray(this.issue.conflicts) && this.issue.conflicts.length >= 1) {
      return this.issue.conflicts[0].id
    }
    return null
  }
  get conflicts () {
    if (Array.isArray((this.issue as IssueI).conflicts)) {
      return (this.issue as IssueI).conflicts
    }
    return []
  }
  get designations () {
    if (this.issue && this.issue.designations) {
      return this.issue.designations
    }
    return null
  }
  get designationIsFixed () {
    try {
      if (this.userCancelled) return false
      if (this.finalName && this.name === this.finalName) return true
      if (!this.changesInBaseName) {
        // creating a new version of allDesignationsList because we are going to modify it but still need the original
        let AllDesignationsList = allDesignationsList
        // allDesignations is an array of objects containing {words, end} keys, set designationEntityTypes to the
        // relevant object
        const designationEntityTypes = allDesignations[this.entity_type_cd]
        /* designationEntityTypes.words.length === 0 is true for types 'PAR', 'FI', 'PA', and the proprietorships so
         the designation issue is fixed as long as there are no designations present and no nameAction words which in
         this situation would only be designation-like words to be removed */
        if (designationEntityTypes && designationEntityTypes.words.length === 0) {
          if (this.nameActionWords.length > 0) {
            for (let word of this.nameActionWords) {
              // so we add any nameActionWords onto the allDesignationsList that were not already part of it
              if (!AllDesignationsList.includes(word)) {
                AllDesignationsList = AllDesignationsList.concat(word)
              }
            }
          }
          // and then fail the test if this.name includes any of the consolodated AllDesignationsList
          for (let designation of AllDesignationsList) {
            if (matchWord(this.name, designation)) {
              return false
            }
          }
          return true
        }
        let end: string
        // here we determine which valid end designation is being used in this.name
        AllDesignationsList.forEach(designation => {
          if (this.name.endsWith(' ' + designation)) {
            end = designation
          }
        })
        /* all the cases where a valid designation was not required are covered by this point, so end should contain
         a match, and we fail the test if it does not */
        if (!end) {
          return false
        }
        if (this.nameActionWords.length > 0) {
          for (let word of this.nameActionWords) {
            if (!AllDesignationsList.includes(word)) {
              AllDesignationsList = AllDesignationsList.concat(word)
            }
          }
        }
        let matches = []
        for (let designation of AllDesignationsList) {
          if (matches.includes(designation)) {
            continue
          }
          let matchedWords = matchWord(this.name, designation)
          if (matchedWords) {
            if (matchedWords.length > 1) {
              return false
            }
            if (matchedWords.length === 1) {
              matches.push(designation)
            }
          }
        }
        // checks for a designation_misplaced issue type that is not the last issue... ie. the name won't be fixed yet
        // but need to pass to proceed to the next issue.
        if (this.issueType === 'designation_misplaced') {
          return !!this.name.endsWith(this.nameActionWords[0])
        }
        if (this.isMisplacedPrecedingMismatch) {
          let nextWords = []
          if (Array.isArray(newReqModule.analysisJSON.issues[this.issueIndex + 1].name_actions)) {
            nextWords = newReqModule.analysisJSON.issues[this.issueIndex + 1].name_actions.map(
              action => action.word.toUpperCase()
            )
            nextWords = nextWords.filter(word => word !== end)
            matches = matches.filter(match => !nextWords.includes(match))
          }
        }
        if (matches.length > 1) {
          return false
        }
        if (Array.isArray(this.designations) && this.designations.length > 0) {
          if (this.designations.some(designation => this.name.endsWith(designation))) {
            return true
          }
        } else {
          if (designationEntityTypes && designationEntityTypes.words.some(word => this.name.endsWith(word))) {
            return true
          }
        }
      }
      return false
    } catch (err) {
      // Catch designation errors and log to console, as it will prevent this component from rendering properly
      // eslint-disable-next-line no-console
      console.warn(err)
      // If something fails in this method, return false, as getters are expected to return a value
      // We don't want to prevent the component from rendering, log the error and continue
      return false
    }
  }
  get hasDesignationIssue () {
    return (this.json.issues.some(issue => newReqModule.designationIssueTypes.includes(issue.issue_type)))
  }
  get enableNextForAssumedName () {
    if (this.issue && ['corp_conflict', 'queue_conflict'].includes(this.issue.issue_type)) {
      return (this.issue.setup[0].type === 'assumed_name' && !this.isLastIndex)
    }
    return false
  }
  get entity_type_cd () {
    return newReqModule.entity_type_cd
  }
  get entityText () {
    return newReqModule.entityTextFromValue
  }
  get examinationOrConsentCompleted () {
    let types = ['send_to_examiner', 'obtain_consent', 'conflict_self_consent']

    if (!this.json.issues.some(issue => issue.setup.some(set => types.includes(set.type)))) return true

    let result = true
    let index = 0

    for (let issue of this.json.issues) {
      for (let set of issue.setup) {
        if (types.includes(set.type)) {
          if (types.every(type => !this.requestExaminationOrProvideConsent[index][type])) {
            result = false
          }
        }
      }
      index++
    }
    return result
  }
  get hasNameActions () {
    if (this.issue && this.issue.name_actions && Array.isArray(this.issue.name_actions)) {
      if (this.issue.name_actions.length > 0) {
        return true
      }
    }
    return false
  }
  get headerProps () {
    if (this.json.status === 'Available') {
      return {
        class: 'approved',
        icon: 'mdi-check-circle',
        text: 'Name Ready for Review',
        showNextLines: true
      }
    }
    let { length } = this.json.issues
    if (length >= 1) {
      return {
        class: 'rejected',
        icon: 'mdi-star-circle',
        text: 'Further Action Required',
        showNextLines: true
      }
    }
    if (this.isLastIndex && this.issue.issue_type !== 'word_to_avoid') {
      if (!this.changesInBaseName && this.allowProceed && this.examinationOrConsentCompleted) {
        return {
          class: 'approved',
          icon: 'mdi-check-circle',
          text: 'Name Ready for Review',
          showNextLines: false
        }
      }
    }
    return {
      class: 'rejected',
      icon: 'mdi-star-circle',
      text: 'Further Action Required',
      showNextLines: true
    }
  }
  get isApproved () {
    return (this.json.status === 'Available')
  }
  get isDesignationIssue () {
    if (this.issue && this.issue.issue_type) {
      return (newReqModule.designationIssueTypes.some(issue => issue.issue_type === this.issue.issue_type))
    }
    return false
  }
  get isLastIndex () {
    return (this.issueIndex === this.issueLength - 1)
  }
  get isMismatchFollowingMisplaced () {
    if (this.issueIndex > 0 &&
    this.issueType === 'designation_mismatch' &&
    newReqModule.analysisJSON.issues[this.issueIndex - 1].issue_type === 'designation_misplaced') {
      return true
    }
    return false
  }
  get isMisplacedPrecedingMismatch () {
    if (this.issueLength > 1 && this.issueIndex < this.issueLength) {
      if (['designation_misplaced', 'end_designation_more_than_once'].includes(this.issueType)) {
        if (newReqModule.analysisJSON.issues[this.issueIndex + 1]) {
          return (newReqModule.analysisJSON.issues[this.issueIndex + 1].issue_type === 'designation_mismatch')
        }
      }
    }
    return false
  }
  get issue () {
    if (Array.isArray(this.json.issues)) {
      return this.json.issues[this.issueIndex]
    }
    return {}
  }
  get issueLength () {
    if (Array.isArray(this.json.issues)) {
      return this.json.issues.length
    }
    return 1
  }
  get issueType () {
    if (this?.issue?.issue_type) {
      return this.issue.issue_type
    }
    return ''
  }
  get json () {
    return newReqModule.analysisJSON
  }
  get location () {
    let value = newReqModule.location
    let options = newReqModule.locationOptions
    return options.find((opt: any) => opt.value === value)
  }
  get name () {
    return newReqModule.name
  }
  get nameActions () {
    if ((this.issue as IssueI) && (this.issue as IssueI).name_actions) {
      return (this.issue as IssueI).name_actions
    }
    return null
  }
  get nameActionWords () {
    if (Array.isArray(this.nameActions) && this.nameActions.length > 0) {
      return this.nameActions.map(action => action.word.toUpperCase())
    }
    return []
  }
  get nameIsFixed () {
    return (this.hasDesignationIssue && this.isLastIndex && this.designationIsFixed && !this.changesInBaseName)
  }
  get nextButtonDisabled () {
    if (this.enableNextForAssumedName) {
      return false
    }
    if (['designation_misplaced', 'end_designation_more_than_once'].includes(this.issue.issue_type)) {
      if (this.allowProceed && this.issueIndex < this.json.issues.length) {
        return false
      }
    }
    for (let type of ['obtain_consent', 'conflict_self_consent', 'send_to_examiner']) {
      if (this.requestExaminationOrProvideConsent[this.issueIndex][type]) {
        return false
      }
    }
    return true
  }
  get quill (): any {
    return (this.$refs as any).quill.quill
  }
  get requestExaminationOrProvideConsent () {
    return newReqModule.requestExaminationOrProvideConsent
  }
  get showActualInput () {
    return newReqModule.showActualInput
  }
  get showGreyBoxes () {
    if (!this.isLastIndex || this.changesInBaseName || !this.hasDesignationIssue) {
      return true
    }
    return !(this.examinationOrConsentCompleted && this.designationIsFixed)
  }
  get word () {
    if (Array.isArray(this.issue.name_actions) && this.issue.name_actions[0]) {
      return this.issue.name_actions[0].word.toUpperCase()
    }
    return ''
  }
  set name (name: string) {
    newReqModule.mutateName(name)
  }
  set showActualInput (value) {
    newReqModule.mutateShowActualInput(value)
  }
  get userCancelled () {
    return newReqModule.userCancelledAnalysis
  }

  cancelAnalyzeName () {
    newReqModule.cancelAnalyzeName('Tabs')
  }
  clickNameField () {
    if (!this.showActualInput) {
      this.showActualInput = true
      if (this.issue && this.issue.issue_type) {
        let selection: SelectionI = this.quill.getSelection()
        let inserts = this.originalOps.map(op => op.insert)
        let indexMap = {}
        let unmappedIndex = 0
        let mappedIndex = 0
        if (!inserts.some(insert => insert.match(/\[.+\]/))) {
          let ops = this.getBaseNameOps()
          this.quill.setContents(ops)
          this.quill.setSelection(selection)
          return
        }
        inserts.forEach(insert => {
          let { length } = insert
          // checking for brackets-type name_actions in the displayed name.  brackets-type name_actions disappear when
          // the user clicks the text-field so if the text field is clicked somewhere after the message, the carat will
          // be inserted at that index but the length of the text-field contents will change.  if there are such
          // messages, map the clicked carat index to the final carat index and insert the carat at the final index
          if (!insert.match(/\[.+\]/)) {
            for (let n = 0; n < length; n++) {
              indexMap[unmappedIndex + n] = mappedIndex + n
            }
            mappedIndex = mappedIndex + length
            unmappedIndex = unmappedIndex + length
          } else {
            for (let n = 0; n < length; n++) {
              indexMap[unmappedIndex + n] = mappedIndex
            }
            unmappedIndex = unmappedIndex + length
          }
        })
        let lastIndex = Object.keys(indexMap).length
        indexMap[lastIndex] = indexMap[lastIndex - 1] + 1
        let ops = this.getBaseNameOps()
        this.quill.setContents(ops)
        let { index } = selection
        selection.index = indexMap[index]
        this.quill.setSelection(selection)
      }
    }
  }
  clickNext () {
    let reset = () => {
      this.highlightCheckboxes = false
    }
    if (!this.nextButtonDisabled) {
      this.issueIndex++
      return
    }
    if (this.nextButtonDisabled) {
      this.highlightCheckboxes = true
      // reset () => highlightedCheckboxes = true
      setTimeout(() => { reset() }, 4000)
    }
  }
  getBaseNameOps () {
    let nameWords = this.originalName.split(' ')
    return nameWords.map((word, i) => (
      { insert: (i > 0) ? (' ' + word) : (word) }
    ))
  }
  handleChange ({ html, text }) {
    // prevents quill from adding a newline to what is supposed to be a single-line input field
    text = text.replace('\n', '')
    this.name = text
    if (html.includes('</p><p>')) {
      html = html.replace('</p><p>', '')
    }
    this.contents = html
  }
  handleEnterKey (event) {
    if (this.isApproved) {
      event.preventDefault()
      this.quill.setText(this.originalName)
      return
    }
    if (event.key === 'Enter') {
      event.stopPropagation()
      event.stopImmediatePropagation()
      event.preventDefault()
      this.name = this.quill.getText()
      let Action = this.nameActions.find(action => action.type === 'brackets')
      if (Action && Action.message) {
        let replace = '[' + Action.message + ']'
        this.name = removeExcessSpaces(this.name.replace(replace, ''))
      }
      newReqModule.startAnalyzeName()
    }
  }
  handleSubmit (event: Event) {
    event.preventDefault()
    this.name = this.quill.getText()
    newReqModule.startAnalyzeName()
  }
  stripAllDesignations (name) {
    for (let word of allDesignationsList) {
      name = replaceWord(name, word)
    }
    return name
  }
  updateContents (text: string) {
    try {
      (this.quill as any).setContents([
        { insert: text }
      ])
    } catch {
      return
    }
  }
}

</script>

<style scoped lang="scss">
@import '@/assets/scss/theme.scss';

#analyze-results-container {
  max-width: 1140px;
}
#name-search-bar {
  margin: -20px 0 20px 0;
  max-height: 40px;
  min-height: 40px;
}
.modal-activator {
  background-color: unset !important;
  color: $app-blue !important;
  cursor: pointer !important;
  letter-spacing: unset !important;
  text-decoration: underline;
  text-transform: none !important;
}
.strike {
  text-decoration-line: line-through;
}
.search-icon {
  position: relative;
  left: 96.5%;
  top: -45px;
  z-index: 1000;
}
.search-tooltip {
  max-width: 100px !important;
  text-align: center;
  padding: 10px !important;
}
#next-issue-btn.disabled-issue-btn {
  color: $app-blue !important;
  border-color: $app-blue !important;
  opacity: .4 !important;
}
</style>
