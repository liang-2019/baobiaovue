<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="产线" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="line">
              <a-input v-model="model.line" placeholder="请输入产线"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="数据库地址" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="databaseaddr">
              <a-input v-model="model.databaseaddr" placeholder="请输入数据库地址"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="用户" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="user">
              <a-input v-model="model.user" placeholder="请输入用户"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="密码" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="password">
              <a-input v-model="model.password" placeholder="请输入密码"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="数据库名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="databasename">
              <a-input v-model="model.databasename" placeholder="请输入数据库名称"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="单班产能" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="cn">
              <a-input-number v-model="model.cn" placeholder="请输入单班产能" style="width: 100%" />
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </j-form-container>
  </a-spin>
</template>

<script>

  import { httpAction, getAction } from '@/api/manage'
  import { validateDuplicateValue } from '@/utils/util'

  export default {
    name: 'CnMaintainForm',
    components: {
    },
    props: {
      //表单禁用
      disabled: {
        type: Boolean,
        default: false,
        required: false
      }
    },
    data () {
      return {
        model:{
         },
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        confirmLoading: false,
        validatorRules: {
           line: [
              { required: true, message: '产线不能为空！'},
              { validator: (rule, value, callback) => validateDuplicateValue('cn_maintain', 'line', value, this.model.id, callback)},
           ],
           databaseaddr: [
              { required: true, message: '数据库地址不能为空！'},
           ],
           user: [
              { required: true, message: '用户名不能为空！'},
           ],
           databasename: [
              { required: true, message: '数据库名不能为空！'},
           ],
        },
        url: {
          add: "/system/cnMaintain/add",
          edit: "/system/cnMaintain/edit",
          queryById: "/system/cnMaintain/queryById"
        }
      }
    },
    computed: {
      formDisabled(){
        return this.disabled
      },
    },
    created () {
       //备份model原始值
      this.modelDefault = JSON.parse(JSON.stringify(this.model));
    },
    methods: {
      add () {
        this.edit(this.modelDefault);
      },
      edit (record) {
        this.model = Object.assign({}, record);
        this.visible = true;
      },
      submitForm () {
        const that = this;
        // 触发表单验证
        this.$refs.form.validate(valid => {
          if (valid) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            httpAction(httpurl,this.model,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
            })
          }
         
        })
      },
    }
  }
</script>