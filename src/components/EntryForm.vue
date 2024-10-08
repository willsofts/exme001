<template>
  <DialogForm ref="dialogForm">
    <template v-slot:header>
      <h4 class="modal-title" v-if="insertMode">{{ labels.title_new }}</h4>
      <h4 class="modal-title" v-if="updateMode">{{ labels.title_edit }}</h4>
    </template>
    <template v-slot:default>
        <div class="row row-height">
          <div class="col-height col-md-4">
            <label for="fieldchar">{{ labels.fieldchar_label }}</label>
            <div class="input-group has-validation" :class="{'has-error': v$.fieldchar.$error}">
              <InputMask ref="fieldchar" v-model="localData.fieldchar" id="fieldchar" name="fieldchar" picture="XXXXXXXXXXXX" :disabled="disabledKeyField"/> 
              <label class="required">*</label>
            </div>
            <span v-if="v$.fieldchar.$error" class="has-error">{{ v$.fieldchar.$errors[0].$message }}</span>
          </div>
        </div>
        <div class="row row-height">
          <div class="col-height col-md-7">
            <label for="fieldtext">{{ labels.fieldtext_label }}</label>
            <div class="input-group" :class="{'has-error': v$.fieldtext.$error}">
              <input ref="fieldtext" type="text" v-model="localData.fieldtext" id="fieldtext" class="form-control input-md" maxlength="100" />
            </div>
            <span v-if="v$.fieldtext.$error" class="has-error">{{ v$.fieldtext.$errors[0].$message }}</span>
          </div>
        </div>
        <div class="row row-height">
          <div class="col-height col-md-4">
            <label for="fielddate">{{ labels.fielddate_label }}</label>
            <div class="input-group" :class="{'has-error': v$.fielddate.$error}">
              <InputDate ref="fielddate" v-model="localData.fielddate" id="fielddate" name="fielddate" /> 
            </div>
            <span v-if="v$.fielddate.$error" class="has-error">{{ v$.fielddate.$errors[0].$message }}</span>
          </div>
          <div class="col-height col-md-3">
            <label for="fieldtime">{{labels.fieldtime_label}}</label>
            <div class="input-group" :class="{'has-error': v$.fieldtime.$error}">
              <InputTime ref="fieldtime" v-model="localData.fieldtime" id="fieldtime" name="fieldtime" /> 
            </div>
            <span v-if="v$.fieldtime.$error" class="has-error">{{ v$.fieldtime.$errors[0].$message }}</span>
          </div>
        </div>
        <div class="row row-height">
          <div class="col-height col-md-4">
            <label for="fielddecimal">{{ labels.fielddecimal_label }}</label>
            <div class="input-group" :class="{'has-error': v$.fielddecimal.$error}">
              <InputMoney ref="fielddecimal" v-model="localData.fielddecimal" id="fielddecimal" name="fielddecimal" decimal="2" /> 
            </div>
            <span v-if="v$.fielddecimal.$error" class="has-error">{{ v$.fielddecimal.$errors[0].$message }}</span>
          </div>
          <div class="col-height col-md-3">
            <label for="fieldinteger">{{ labels.fieldinteger_label }}</label>
            <div class="input-group" :class="{'has-error': v$.fieldinteger.$error}">
              <InputNumber ref="fieldinteger" v-model="localData.fieldinteger" id="fieldinteger" name="fieldinteger" /> 
            </div>
            <span v-if="v$.fieldinteger.$error" class="has-error">{{ v$.fieldinteger.$errors[0].$message }}</span>
          </div>
        </div>
        <div class="row row-height">
          <div class="col-height col-md-3">
            <label>{{ labels.fieldvarchar_label }}</label>
            <div class="input-group" :class="{'has-error': v$.fieldvarchar.$error}">
              <select ref="fieldvarchar" v-model="localData.fieldvarchar" class="form-control input-md">
                <option value=""></option>          
                <option v-for="item in dataCategory.tkprogtype" :key="item.id" :value="item.id">{{item.text}}</option>
              </select>
            </div>
            <span v-if="v$.fieldvarchar.$error" class="has-error">{{ v$.fieldvarchar.$errors[0].$message }}</span>
          </div>
          <div class="col-height col-md-3">
            <label>{{ labels.fieldflag_label }}</label>
            <div class="input-group" :class="{'has-error': v$.fieldflag.$error}">
              <select ref="fieldflag" v-model="localData.fieldflag" class="form-control input-md">
                <option v-for="item in dataCategory.tkactive" :key="item.id" :value="item.id">{{item.text}}</option>
              </select>
            </div>
            <span v-if="v$.fieldflag.$error" class="has-error">{{ v$.fieldflag.$errors[0].$message }}</span>
          </div>
        </div>
    </template>
    <template v-slot:footer>
      <button class="btn btn-dark btn-sm" @click="saveClick" v-if="insertMode"><em class="fa fa-save fa-btn-icon"></em>{{ labels.save_button }}</button>
      <button class="btn btn-dark btn-sm" @click="updateClick" v-if="updateMode"><em class="fa fa-save fa-btn-icon"></em>{{ labels.update_button }}</button>
      <button class="btn btn-dark btn-sm" data-dismiss="modal"><em class="fa fa-close fa-btn-icon"></em>{{ labels.cancel_button }}</button>
    </template>
  </DialogForm>
</template>
<script>
import { ref, computed, watch } from 'vue';
import { useVuelidate } from '@vuelidate/core';
import { required, helpers } from '@vuelidate/validators';
import $ from "jquery";
import { DEFAULT_CONTENT_TYPE, getApiUrl }  from '@willsofts/will-app';
import { startWaiting, stopWaiting, submitFailure, detectErrorResponse }  from '@willsofts/will-app';
import { confirmUpdate, confirmSave, confirmDelete, successbox, serializeParameters } from '@willsofts/will-app';
import { InputDate, InputTime, InputNumber, InputMoney, InputMask } from '@willsofts/will-control';
import DialogForm from './DialogForm.vue';

const defaultData = {
  fieldchar: '',
  fieldtext: '',
  fielddate: "",
  fieldtime: "",
  fieldecimal: 0.00,
  fieldinteger: 0,
  fieldvarchar: "F",
  fieldflag: "0",
};

export default {
  components: {
    DialogForm, InputDate, InputTime, InputNumber, InputMoney, InputMask
  },
  props: {
    modes: Object,
    labels: Object,
    dataCategory: Object
  },
  emits: ["data-saved","data-updated","data-deleted"],
  setup(props) {
    const mode = ref({action: "new", ...props.modes});
    const localData = ref({ ...defaultData }); 
    const disabledKeyField = ref(false);
    const reqalert = ref(props.labels.empty_alert);
    const agealert = ref(props.labels.age_alert);
    const requiredMessage = () => {
      return helpers.withMessage(reqalert, required);
    }
    const validateRules = computed(() => { 
      return {
        fieldchar: { required: requiredMessage() },
        fieldtext: { required: requiredMessage() },
        fielddate: { required: requiredMessage() },
        fieldtime: { required: requiredMessage() },
        fielddecimal: { required: requiredMessage() },
        fieldinteger: { required: requiredMessage() },
        fieldvarchar: { required: requiredMessage() },
        fieldflag: { required: requiredMessage() },
      } 
    });
    const v$ = useVuelidate(validateRules, localData, { $lazy: true, $autoDirty: true });
    return { mode, v$, localData, disabledKeyField, reqalert, agealert };
  },
  created() {
    watch(this.$props, (newProps) => {      
      this.reqalert = newProps.labels.empty_alert;
    });
  },
  computed: {
    insertMode() {
      return this.mode.action=="insert" || this.mode.action=="new";
    },
    updateMode() {
      return this.mode.action=="update" || this.mode.action=="edit";
    },
  },
  mounted() {
    this.$nextTick(function () {
      $("#modaldialog_layer").find(".modal-dialog").draggable();
    });
  },
  methods: {
    reset(newData,newMode) {
      if(newData) this.localData = {...newData};
      if(newMode) this.mode = {...newMode};
    },
    submit() {      
      this.$emit('update-formData', this.localData);
    },
    async saveClick() {
      console.log("click: save");
      let valid = await this.validateForm();
      if(!valid) return;
      this.startSaveRecord();
    },
    async updateClick() {
      console.log("click: update");
      let valid = await this.validateForm();
      if(!valid) return;
      this.startUpdateRecord();
    },
    async validateForm(focusing=true) {
      const valid = await this.v$.$validate();
      console.log("validate form: valid",valid);
      console.log("error:",this.v$.$errors);
      if(!valid) {
        if(focusing) {
          this.focusFirstError();
        }
        return false;
      }
      return true;
    },
    focusFirstError() {
      if(this.v$.$errors && this.v$.$errors.length > 0) {
        let input = this.v$.$errors[0].$property;
        let el = this.$refs[input];
        if(el) el.focus(); //if using ref
        else $("#"+input).trigger("focus"); //if using id
      }
    },
    showDialog() {
      //$("#modaldialog_layer").modal("show");
      $(this.$refs.dialogForm.$el).modal("show");
    },  
    hideDialog() {
      //$("#modaldialog_layer").modal("hide");
      $(this.$refs.dialogForm.$el).modal("hide");
    },
    formData() { return this.localData; },
    resetRecord() {
      //reset to default data 
      this.reset(defaultData,{action:"insert"});
      //reset validator
      this.v$.$reset();
      //enable key field
      this.disabledKeyField = false;
    },
    startInsertRecord() {
      this.resetRecord();
      this.showDialog();
    },
    startSaveRecord() {
      confirmSave(() => {
        this.saveRecord(this.localData);
      });
    },
    startUpdateRecord() {
      confirmUpdate(() => { 
        this.updateRecord(this.localData);
      });
    },
    startDeleteRecord(dataKeys) {
      confirmDelete(Object.values(dataKeys),() => {
        this.deleteRecord(dataKeys);
      });
    },
    saveRecord(dataRecord) {
        let jsondata = {ajax: true};
        let formdata = serializeParameters(jsondata,dataRecord,true);
        startWaiting();
        $.ajax({
          url: getApiUrl()+"/api/exme001/insert",
          data: formdata.jsondata,
          headers : formdata.headers,
          type: "POST",
          dataType: "json",
          contentType: DEFAULT_CONTENT_TYPE,
          error : function(transport,status,errorThrown) {
            console.error("error: status",status,"errorThrown",errorThrown);
            submitFailure(transport,status,errorThrown);
          },
          success: (data) => {
            stopWaiting();
            console.log("success",data);
            if(detectErrorResponse(data)) return;
            successbox(() => {
              //reset data for new record insert
              this.resetRecord();
              this.$refs.fieldchar.focus();
            });
            this.$emit('data-saved',dataRecord,data);
          }
      });
    },
    updateRecord(dataRecord) {
        let jsondata = {ajax: true};
        //Object.assign(jsondata,dataRecord);
        let formdata = serializeParameters(jsondata,dataRecord);
        startWaiting();
        $.ajax({
          url: getApiUrl()+"/api/exme001/update",
          data: formdata.jsondata,
          headers : formdata.headers,
          type: "POST",
          dataType: "json",
          contentType: DEFAULT_CONTENT_TYPE,
          error : function(transport,status,errorThrown) {
            console.error("error: status",status,"errorThrown",errorThrown);
            submitFailure(transport,status,errorThrown);
          },
          success: (data) => {
            stopWaiting();
            console.log("success",data);
            if(detectErrorResponse(data)) return;
            successbox(() => {
              this.hideDialog();
            });
            this.$emit('data-updated',dataRecord,data);
          }
      });
    },
    retrieveRecord(dataKeys) {
      let jsondata = {ajax: true};
      //Object.assign(jsondata,dataKeys);
      let formdata = serializeParameters(jsondata,dataKeys);
      startWaiting();
      $.ajax({
        url: getApiUrl()+"/api/exme001/retrieve",
        data: formdata.jsondata,
        headers : formdata.headers,
        type: "POST",
        dataType: "json",
        contentType: DEFAULT_CONTENT_TYPE,
        error : function(transport,status,errorThrown) {
          console.error("retrieveRecord: error: status",status,"errorThrown",errorThrown);
          submitFailure(transport,status,errorThrown);
        },
        success: (data) => {
          stopWaiting();
          console.log("retrieveRecord: success",data);
          if(data.body.dataset) {
            this.reset(data.body.dataset,{action:"edit"});
            this.v$.$reset();
            this.disabledKeyField = true;
            this.showDialog();
          }
        }
      });	
    },
    deleteRecord(dataKeys) {
      let jsondata = {ajax: true};
      //Object.assign(jsondata,dataKeys);
      let formdata = serializeParameters(jsondata,dataKeys);
      startWaiting();
      $.ajax({
        url: getApiUrl()+"/api/exme001/remove",
        data: formdata.jsondata,
        headers : formdata.headers,
        type: "POST",
        dataType: "json",
        contentType: DEFAULT_CONTENT_TYPE,
        error : function(transport,status,errorThrown) {
          console.error("deleteRecord: error: status",status,"errorThrown",errorThrown);
          submitFailure(transport,status,errorThrown);
        },
        success: (data) => {
          stopWaiting();
          console.log("deleteRecord: success",data);
          if(detectErrorResponse(data)) return;
          successbox();
          this.$emit('data-deleted',dataKeys,data);
        }
      });	
    },
  }
};
</script>
