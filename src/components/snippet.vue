<template>
  <div class="" >
    <h2> Markdown or Text Editor?</h2>
    <hr>
    <b-row>
     
      <b-col class="custom-editor-col" cols="6" >
        <b-btn v-show="selectedMdEditor" variant="custom" class="custom-btn-select" @click="selectMdEditor"><i class="icon-frame"></i></b-btn>
        <b-btn v-show="!selectedMdEditor" variant="custom" class="" style="position: absolute;background:  transparent;top: 0;left: 1%;" @click="selectMdEditor"><i class="icon-frame"></i></b-btn>
        <div class="md-editor" :style="mdEditorOnSelectProperty">
          <h2>Markdown Editor</h2>
          <b-form >
            <b-form-group id="exampleInputGroup1"
                          label=""
                          label-for="exampleInput1">
              <b-form-input :disabled="selectedMdEditor" id="exampleInput1"
              class="mb-3"
                            type="text"
                            v-model="title"
                            required
                            placeholder="Enter name">
              </b-form-input>

              <b-form-input :disabled="selectedMdEditor" id="exampleInput2"
              class="mb-3"
                            type="text"
                            v-model="languageName"
                            required
                            placeholder="Enter language">
              </b-form-input>

            </b-form-group>
            <label><b-btn :disabled="selectedMdEditor" id="popoverButton-sync" variant="success">Guide to Markdown</b-btn></label>
            <b-form-group class="mb-3">
              
              <textarea :disabled="selectedMdEditor" style="width:100%;" rows="10" v-model="code" placeholder="Enter code snippet"></textarea>
            </b-form-group>
      
            <b-button :disabled="selectedMdEditor" variant="primary" @click="onSubmit">Update</b-button>
          </b-form>
        </div>
        
      </b-col>

      <b-col class="custom-editor-col" cols="6">
        <b-btn v-show="!selectedTextEditor" variant="custom" class="" style="position: absolute;background:  transparent;top: 0;left: 1%;" @click="selectTextEditor"><i class="icon-frame"></i></b-btn>
        <b-btn v-show="selectedTextEditor" variant="custom" class="custom-btn-select-te"  @click="selectTextEditor"><i class="icon-frame"></i></b-btn>
        <div class="text-editor-select" :style="textEditorOnSelectProperty">
          <h2>Text Editor</h2>
          <b-form-group id="exampleInputGroup1"
                          label=""
                          label-for="exampleInput1">
              <b-form-input :disabled="selectedTextEditor" id="exampleInput1"
              class="mb-3"
                            type="text"
                            v-model="title"
                            required
                            placeholder="Enter name">
              </b-form-input>

            </b-form-group>
          <quill-editor :disabled="selectedTextEditor" v-model="content"></quill-editor>
          <b-button :disabled="selectedTextEditor" class="custom-text-editor-btn" variant="primary" @click="onSubmit">Update</b-button>
        </div>
        
      </b-col>

    </b-row>
    <b-popover target="popoverButton-sync" title="Guide">
      <div class="mb-3">
        Using `(single backtick) or ```(triple backtick) You can call out code or a command within a sentence with single backticks. The text within the backticks will not be formatted.
          <br/>For example - ```var foo = bar;```
      </div>
      <div class="mb-3">
        Use # for h1 tag, ## for h2 and ### for h3
      </div>
       <a href="https://github.com/showdownjs/showdown/wiki/Showdown's-Markdown-syntax#headings">More Help</a>
    </b-popover>
  </div>
</template>

<script>
import API,{ ApiCall } from '../api/api';
import showdown from 'showdown';
import moment from 'moment'
import 'quill/dist/quill.core.css'
import 'quill/dist/quill.snow.css'
import 'quill/dist/quill.bubble.css'
import { quillEditor } from 'vue-quill-editor'

export default {
  data () {
    return {
      code : null,
      title: '',
      content :null,
      languageList : [],
      selectedMdEditor: true,
      selectedTextEditor : true,
      language : null,
      show: true,
      languageName : '',
      fileType : null,
      mdEditorOnSelectProperty : 'opacity:0.4;border : 2px dotted #333;',
      textEditorOnSelectProperty : 'opacity:0.4;border : 2px dotted #333;',
    }
  },
  components:{
    quillEditor
  },
  created(){
    
  },
  mounted () {
      const id = this.$route.params.userId;
      var token = localStorage.getItem('token');
      ApiCall(API.getCodeById + id,'GET',{id:id},{
          'Content-Type': 'application/json',
          'Authorization': 'Bearer ' + token
        })
      .then(responseJson => {
        console.log('Response',responseJson);
        var data = responseJson;
        if(data.fileType === 'markdown'){
          this.selectedMdEditor = !this.selectedMdEditor;
          this.code = data.raw_cont;
          this.languageName = data.language;
          this.title = data.name;
          this.mdEditorOnSelectProperty = 'opacity:1;border:none;';
          this.fileType = data.fileType;
        } else if(data.fileType === 'textnote') {
          this.selectedTextEditor = !this.selectedTextEditor;
          this.content = data.content;
          this.title = data.name;
          this.textEditorOnSelectProperty = 'opacity:1;border:none;';
          this.fileType = data.fileType;
        } else{
          this.selectedMdEditor = !this.selectedMdEditor;
          this.code = data.raw_cont;
          this.languageName = data.language;
          this.title = data.name;
          this.mdEditorOnSelectProperty = 'opacity:1;border:none;';
        }
      })
      ApiCall(API.getLanguageList,'GET')
      .then((response) => {
        this.languageList = response;
      })
  },
  methods: {
    selectMdEditor(){
      this.mdEditorOnSelectProperty === 'opacity:1;border:none;' 
      ? this.mdEditorOnSelectProperty = 'opacity:0.4;border : 2px dotted #333;' 
      : this.mdEditorOnSelectProperty = 'opacity:1;border:none;';
      

      this.selectedMdEditor = !this.selectedMdEditor;
    },
    selectTextEditor(){
      this.textEditorOnSelectProperty === 'opacity:1;border:none;' 
      ? this.textEditorOnSelectProperty = 'opacity:0.4;border : 2px dotted #333;' 
      : this.textEditorOnSelectProperty = 'opacity:1;border:none;';
      
      this.selectedTextEditor = !this.selectedTextEditor;
    },
    onSubmit () {
      let converter = new showdown.Converter();
      var name = this.title;
      var content = '';
      var fileType = this.fileType;
      const id = this.$route.params.userId;
      if(this.code){
        content = this.code;
        fileType = 'markdown';
      } else if(this.content){
        content = this.content;
        fileType = 'textnote';
      } else{
        content = this.code;
        fileType = 'markdown';
      }
      var token = localStorage.getItem('token');

      ApiCall(API.updateCodeById + id,'PUT',{name,content,fileType},{
        'Content-Type': 'application/json',
        'Authorization': 'Bearer ' + token
      })
      .then((response) => {
        console.log('Response',response);
        this.$toasted.show('Submitted successfully');
      })

    },
    
  }
}
</script>

<style >
.custom-editor-col{
  position: relative;
}

.custom-editor-col .custom-btn-select{
  font-size: 6rem;
  position: absolute;
  top: 32%;
  left: 41%;
  background: transparent;
  z-index: 1111;
}

.custom-btn-select-te{
  font-size: 6rem;
  position: absolute;
  top: 32%;
  left: 41%;
  background: transparent;
  z-index: 1111;
}

.md-editor{
  transition: all 0.5s ease;
  padding: 10px 15px;
}

.text-editor-select{
  height: 527px;
  transition: all 0.5s ease;
  padding: 10px 15px;
}

.custom-text-editor-btn{
  margin-top: 2%;
}
</style>
