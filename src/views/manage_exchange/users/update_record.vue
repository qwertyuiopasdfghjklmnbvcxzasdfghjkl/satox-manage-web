<template>
    <div class="update_record">
        <Card style="width:800px;">
            <p slot="title">
                操作日志
                <i class="ivu-icon ivu-icon-close" style="float:right;cursor:pointer;" @click="closeDialog"></i>
            </p>
            <Row style="height:40px;line-height:40px;margin-bottom:20px;">
                <Col span="3">业务领域</Col>
                <Col span="5">
                    <Input v-model="domain"></Input>
                </Col>
                <Col span="3">操作类型</Col>
                <Col span="5">
                    <Input v-model="operate"></Input>
                </Col>
                <Col span="2">
                    <Button type="primary" @click="getChangeRecord" style="margin-left:10px;">查询</Button>
                </Col>
            </Row>
            <Table :columns="columns1" :data="data1"></Table>
            <Page :current="curPage" :total="total" :page-size="pageSize" @on-change="changePage" style="text-align:center;margin-top:20px;"></Page> 
        </Card>
    </div>
</template>

<script>
import currenyApi from '../../../api/currency'
export default {
    props: ['userId'],
    data () {
        return {
            domain: '',
            operate: '',
            curPage: 1,
            total: 0,
            pageSize: 5,
            columns1: [
                {title: '业务领域', key: 'domain'},
                {title: '操作类型', key: 'operate'},
                {title: '操作详细说明', key: 'operateData', width: 250, 
                 render: (h, params) => {
                            return h('div', [
                                h('div', {
                                    style: {height: '100px', overflow: 'auto'},
                                }, params.row.operateData)
                            ]);
                        }
                },
                {title: '操作时间', key: 'operateTime'},
                // {title: '操作人员姓名', key: 'operatorName'}
            ],
            data1: []
        }
    },
    created () {
        this.getChangeRecord()
    },
    methods: {
        closeDialog () {
            this.$emit('removeDialog');
        },
        getChangeRecord() {
            currenyApi.findUserOperateLogList(this.pageSize, this.curPage, {
                operateUser: this.userId,
                operate: this.operate || '',
                domain: this.domain || ''
            }, (res, total) => {
                this.total = total
                this.data1 = res
            }, (msg) => {
             this.$Message.error({content: msg})
         })
        },
        changePage (page) {
          this.curPage = page
          this.getChangeRecord()
        }
    }
}
</script>

<style scoped>
.update_record{width: 800px;height: 600px;overflow: auto;}
</style>
