<template>
  <Card style="width:450px;">
      <p slot="title">
        创建分发
        <i class="ivu-icon ivu-icon-close" style="float:right;cursor:pointer;" @click="closeDialog"></i>
      </p>
      <Form ref="formValidate" :model="formValidate" :rules="ruleValidate" >
             <FormItem label="编号" prop="serialNumber">
                <Input v-model="formValidate.serialNumber" name="serialNumber"></Input>
            </FormItem> 
            <!-- <FormItem label="创建时间" prop="creatTime">
                <DatePicker type="date" placeholder="请选择时间" name="creatTime" v-model="formValidate.creatTime" width="100%"></DatePicker>
            </FormItem> -->
            <FormItem label="币种" prop="symbol">
                <Select v-model="formValidate.symbol" name="symbol">
                    <Option :value="data.symbol" v-for="data in symbolData" :key="data.id">{{data.symbol}}</Option>
                </Select>
            </FormItem>
            <FormItem label="数量" prop="quantity">
                <Input v-model="formValidate.quantity" name="quantity"></Input>
            </FormItem>
            <FormItem label="目标用户ID" prop="userId">
                <Input v-model="formValidate.userId" name="userId"></Input>
            </FormItem>
            <FormItem label="目标账户ID" prop="accountId">
                <Input v-model="formValidate.accountId" name="accountId"></Input>
            </FormItem>
             <FormItem label="来源账户ID" prop="sourceAccountId">
                <Input v-model="formValidate.sourceAccountId" name="sourceAccountId"></Input>
            </FormItem>
             <FormItem label="备注" prop="remarks">
                <Input v-model="formValidate.remarks" name="remarks" :maxlength="255"></Input>
            </FormItem>
            <div style="text-align:center;margin-top:15px;">
                <Button type="primary" style="margin-right:10px;" @click="creat">创建</Button>
                <Button type="ghost" @click="cancel()">取消</Button>
            </div>
      </Form>
  </Card> 
</template>
<script>
import extendApi from '../../api/extend'
export default {
    data () {
        return {
            formValidate: {
                serialNumber: null,
                creatTime: '',
                symbol: '',
                quantity: null,
                userId: null,
                accountId: null,
                sourceAccountId: null,
                remarks: null
            },
            ruleValidate: {
                serialNumber: [
                    { required: true, message: '请输入编号', trigger: 'blur' }
                ],
                creatTime: [
                    { required: true, type: 'date', message: '请输入时间', trigger: 'change' }
                ],
                symbol: [
                    { required: true, message: '请输入币种', trigger: 'blur' }
                ],
                quantity: [
                    { required: true, message: '请输入数量', trigger: 'blur' }
                ],
                userId: [
                    { required: true, message: '请输入目标用户ID', trigger: 'blur' }
                ],
                accountId: [
                    { required: true, message: '请输入目标账户ID', trigger: 'blur' }
                ],
                sourceAccountId: [
                    { required: true, message: '请输入来源账户ID', trigger: 'blur' }
                ],
                remarks: [
                    { required: true, message: '请输入备注', trigger: 'blur' }
                ],
            },
            symbolData: []
        }
   },
   created () {
       this.findSymbolInfo()
   },
   methods: {
       findSymbolInfo () {
           extendApi.findSymbolInfo((res) => {
               this.symbolData = res
           })
       },
      cancel () {
          this.$emit('removeDialog')
      },
      creat () {
           this.$refs.formValidate.validate((valid) => {
                if (valid) {
                    let formData = new FormData(this.$refs.formValidate.$el);
                    extendApi.createSingleDistribute(formData, (res) => {
                        this.$Message.success({content: '创建成功'})
                        // this.findSymbolInfo()
                        this.$emit('okCallback')
                        this.$emit('removeDialog')
                    })
                }
            })
      },
      closeDialog () {
        this.$emit('removeDialog')
      }
    }  
}
</script>

