<template>
  <div>
    <div v-if="$loadingRouteData" class="data-loading">
      <mdl-spinner></mdl-spinner>
    </div>
    <div class="app-header-strip">
      <div class="mdl-grid">
        <div class="mdl-cell mdl-cell--12-col">
          <h2 class="app-title-text">
            <a v-link="{ path: '/app/exams' }">Exams</a> / {{ exam.name }}
          </h2>
        </div>
      </div>
    </div>
    <div class="padded-content" v-show="!$loadingRouteData">
      <div class="mdl-grid">
        <div class="mdl-cell mdl-cell--12-col mdl-color--white mdl-shadow--2dp">
          <div class="mdl-tabs mdl-js-tabs mdl-js-ripple-effect">
            <div class="mdl-tabs__tab-bar">
              <a href="#about-panel" class="mdl-tabs__tab is-active">Questions</a>
              <a href="#members-panel" class="mdl-tabs__tab">Exam Settings</a>
            </div>
            <div class="mdl-tabs__panel is-active" id="about-panel">
              <div class="exam-toolbar">
                <a class="toolbar-link" @click="displayForm">
                  <i class="material-icons">add</i>
                  Add question
                </a>
                <a class="toolbar-link">
                  <i class="material-icons">import_export</i>
                  Import from JSON
                </a>
              </div>
              <div class="exam-question" v-show="addingQuestion">
                <div class="mdl-grid">
                  <div class="mdl-cell mdl-cell--6-col">
                    <div class="padded-content compact">
                      <div class="flex align-middle-left">
                        <h4 class="fill">Create a new question</h4>
                        <div class="toggle-switch flex fill align-middle-right">
                          <span class="ms-label">Toggle Preview</span>
                          <mdl-switch :checked.sync="displayPreview"></mdl-switch>
                        </div>
                      </div>
                      <form action="#" class="flex column-layout">
                        <mdl-textfield label="Question Text (accepts markdown)" :value.sync="question.text" textarea rows="3"></mdl-textfield>
                        <p class="form-subheader-text">Options</p>
                        <div class="exam-options">
                          <div class="exam-option flex align-middle-left" v-for="i in numberOfOptions">
                            <label for="question-correct-{{$index}}" class="mdl-radio mdl-js-radio mdl-js-ripple-effect">
                              <input type="radio" id="question-correct-{{$index}}" name="question-correct" class="mdl-radio__button" v-model="question.is_correct" value="{{$index}}">
                            </label>
                            <mdl-textfield label="Enter option text" :value.sync="question.options[$index].text" class="fill"></mdl-textfield>
                          </div>
                          <mdl-anchor-button colored class="mdl-js-ripple-effect mdl-color-text--cyan" @click.prevent="addOption">Add Option</mdl-anchor-button>
                        </div>
                        <div class="actions">
                          <mdl-button raised colored class="mdl-js-ripple-effect mdl-color--cyan" @click.prevent="closeForm">Cancel</mdl-button>
                          <mdl-button raised colored class="mdl-js-ripple-effect mdl-color--cyan" @click.prevent="createQuestion">Create Question</mdl-button>
                        </div>
                      </form>
                    </div>
                  </div>
                  <div class="mdl-cell mdl-cell--6-col" v-show="displayPreview">
                    <div class="padded-content compact exam-preview">
                      <h4>Preview Question</h4>
                      <div class="question-text" v-html="question.text || '' | marked">{{ question.text }}</div>
                      <ol type="A">
                        <li v-for="option in question.options">{{ option.text }}</li>
                      </ol>
                    </div>
                  </div>
                </div>
              </div>
              <div class="padded-content spacious">
                <table class="question-items" v-show="questions.length" cellspacing=0 cellpadding=0>
                  <thead>
                    <tr>
                      <th width="40"></th>
                      <th width="70%">Question Text</th>
                      <th></th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="question in questions">
                      <td>#{{ $index + 1 }}</td>
                      <td>{{ question.text }}</td>
                      <td class="actions">
                        <a href="#" id="question-visibility"><i class="material-icons">visibility</i></a>
                        <a href="#" id="question-move-up"><i class="material-icons">arrow_upward</i></a>
                        <a href="#" id="question-move-down"><i class="material-icons">arrow_downward</i></a>
                        <a href="#" id="question-edit"><i class="material-icons">edit</i></a>
                        <a id="question-delete" @click.prevent="deleteQuestion(question.__id, $index, $event)"><i class="material-icons">delete</i></a>
                      </td>
                    </tr>
                  </tbody>
                </table>
                <div class="empty-message regular flex column-layout align-middle-center" v-show="!questions.length">
                  <div class="empty-icon">
                    <img src="/build/img/folder.png" />
                  </div>
                  <div class="empty-text">
                    <h2>There are currently no questions here</h2>
                  </div>
                </div>
              </div>
            </div>
            <div class="mdl-tabs__panel" id="members-panel">
              <div class="padded-content spacious">
                <p class="form-subheader-text">Public URL</p>
                <a v-link="{ path: '/runner/' + exam.__id }" target="_blank">View Exam</a>
                <p class="form-subheader-text spacer">Timing</p>
                <mdl-checkbox :checked.sync="exam.timed" class="mdl-js-ripple-effect">Timed</mdl-checkbox>
                <mdl-textfield label="Enter exam duration in seconds" :value.sync="exam.duration"></mdl-textfield>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import _ from 'lodash';
import marked from 'marked';

import { notify, loadUI, idfyObj, objectToArray } from './utils'

import User from '../models/user';
import Exam from '../models/exam';
import Question from '../models/question';

export default {
  data() {
    return {
      questions: [],
      question: {
        text: '',
        is_correct: 0,
        options: []
      },
      addingQuestion: false,
      numberOfOptions: 2,
      displayPreview: true
    }
  },
  route: {
    activate() {
      loadUI()
    },
    data(transition) {
      let examId = this.$route.params.examId;
      Exam.get(examId, (exam) => {
        Question.all(examId, (questions) => {
          transition.next({
            exam: idfyObj(examId, exam),
            questions: objectToArray(questions)
          })
        })
      })
    },
  },
  methods: {
    displayForm() {
      this.addingQuestion = true;
    },
    resetQuestion() {
      this.question = {
        text: '',
        is_correct: 0,
        options: []
      };
      this.numberOfOptions = 2;
    },
    addOption() {
      if(this.numberOfOptions < 5) {
        this.numberOfOptions += 1;
        loadUI();
      }
    },
    closeForm() {
      this.addingQuestion = false;
      this.resetQuestion();
    },
    createQuestion(event) {
      let uid = this.$parent.user.uid;
      Question.create(uid, this.exam.__id, this.question, (err, key) => {
        if(err) {
          console.log("Error occured while trying to create question.")
        } else {
          let _fb_q = idfyObj(key, this.question);
          this.questions.push(_fb_q);
          this.closeForm();
          notify("Successfully added the question");
        }
      })
    },
    deleteQuestion(id, idx, event) {
      swal({
        title: "Are you sure?",
        text: "You are about to delete this question from this exam!",
        type: "warning",
        showCancelButton: true,
        confirmButtonText: "Yes, delete it!",
        closeOnConfirm: true
      }, () => {
        Question.delete(this.exam.__id, id, (err) => {
          if(err) {
            console.log("Error occured while trying to delete question.")
          } else {
            this.questions.splice(idx, 1);
            notify("Successfully deleted the question");
          }
        })
      });
    }
  },
  filters: {
    marked: marked
  }
}
</script>