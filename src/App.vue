<template>
    <div id="app">
        <el-row type="flex" justify="center">
            <el-col :span="10" style="text-align: left">
                <el-select size="mini" filterable v-model="source">
                    <el-option-group v-for="lang in languages" :key="lang.label" :label="lang.label">
                        <el-option v-for="item in lang.options" :key="item.value" :label="item.label"
                                   :value="item.value">
                        </el-option>
                    </el-option-group>
                </el-select>
            </el-col>
            <el-col :span="4" style="text-align: center">
                <el-button size="mini" @click="change" type="success" icon="el-icon-refresh"></el-button>
            </el-col>
            <el-col :span="10" style="text-align: right">
                <el-select size="mini" filterable v-model="target">
                    <el-option-group v-for="lang in languages" :key="lang.label" :label="lang.label">
                        <el-option v-for="item in lang.options" :key="item.value" :label="item.label"
                                   :value="item.value">
                        </el-option>
                    </el-option-group>
                </el-select>
            </el-col>
        </el-row>

        <hr>

        <el-form :model="ruleForm" label-position="top" ref="ruleForm">
            <el-row>
                <el-col :span="24">

                    <el-form-item :label="templateData.labelSource" prop="source">
                        <el-input tabindex="1" id="source" :placeholder="templateData.textareatPlaceholder"
                                  type="textarea" v-model="ruleForm.source"></el-input>
                    </el-form-item>
                </el-col>
            </el-row>

            <el-row type="flex" justify="space-between">
                <el-col :span="16">
                    <el-button @click="translate" style="width: 100%" size="mini" type="primary"
                               icon="el-icon-edit">
                        {{ templateData.translate }}
                    </el-button>
                </el-col>
                <el-col :span="7">
                    <el-button @click="clear" style="width: 100%" size="mini">
                        {{ templateData.clear }}
                    </el-button>
                </el-col>
            </el-row>

            <el-row>
                <el-col :span="24">
                    <el-form-item :label="templateData.labelTarget" prop="target">
                        <el-input type="textarea" id="target" v-model="ruleForm.target"></el-input>
                    </el-form-item>
                </el-col>
            </el-row>
        </el-form>

        <el-row id="footer">
            <el-col style="font-size: 12px;" :span="12">
                &copy; 2017 "Simple Translation"
            </el-col>
            <el-col :span="12" style="font-size: 12px; text-align: right">
                Open Source Project - <a target="_blank" href="https://github.com/kamaliev/simple-translate">GitHub</a>
            </el-col>
        </el-row>
    </div>
</template>

<script>
  import axios from 'axios'
  import qs from 'qs'
  import {API_KEY} from './config'

  export default {
    name: 'app',
    data () {
      return {
        ruleForm: {
          source: '',
          target: ''
        },
        templateData: {
          textareatPlaceholder: window.chrome.i18n.getMessage('textareatPlaceholder'),
          translate: window.chrome.i18n.getMessage('translate'),
          clear: window.chrome.i18n.getMessage('clear'),
          labelSource: window.chrome.i18n.getMessage('labelSource'),
          labelTarget: window.chrome.i18n.getMessage('labelTarget')
        },
        languages: [{
          label: window.chrome.i18n.getMessage('popularLanguage'),
          options: [{
            value: 'en',
            label: window.chrome.i18n.getMessage('english')
          }, {
            value: 'ru',
            label: window.chrome.i18n.getMessage('russian')
          }]
        }, {
          label: window.chrome.i18n.getMessage('otherLanguages'),
          options: [{
            value: 'de',
            label: window.chrome.i18n.getMessage('german')
          }]
        }],
        source: 'en',
        target: 'ru'
      }
    },
    mounted: function () {
      let source = document.getElementById('source')
      source.value = ''
      source.select()

      if (document.execCommand('paste')) {
        this.translate()
        this.translate()
      }

      source.value = ''
    },
    methods: {
      translate: function () {
        let _this = this
        let data = {
          'q': this.ruleForm.source,
          'source': this.source,
          'target': this.target,
          'key': API_KEY,
          'format': 'text',
          'model': 'nmt'
        }

        axios
          .post('https://translation.googleapis.com/language/translate/v2', qs.stringify(data))
          .then(function ({data}) {
            _this.ruleForm.target = data.data.translations[0].translatedText
          })
          .catch(function (response) {
            _this.$message.error(window.chrome.i18n.getMessage('apiError'))

            console.error(response)
          })
      },
      clear: function () {
        this.$refs.ruleForm.resetFields()
      },
      change: function () {
        let gap = this.target
        this.target = this.source
        this.source = gap
      }
    }
  }
</script>