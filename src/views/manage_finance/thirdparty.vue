<template>
    <Row style="margin-top:10px;">
        <Card>
            <p slot="title">{{$t('exchange.bzcztj')}}</p>
            <Table ref="test2" :columns="columns1" :data="data1"></Table>
            <Page :current="curPage1" :total="total1" @on-change="changePage1"
                  :page-size="size" style="text-align:center;margin-top:20px;"></Page>
        </Card>
        <Card style="margin-top: 10px">
            <p slot="title">{{$t('exchange.dsfczlb')}}
                <Button type="primary" @click="downloadList()">{{$t('systemlog.dc')}}</Button>
            </p>
            <row class="selWp">
                <Col :md='8' :lg='6'>
                <label>{{$t('common.bz')}}：</label>
                <Select v-model="formData.symbol" style="width: 154px;" :clearable="true">
                    <Option value="0">{{$t('common.qb')}}</Option>
                    <Option v-for="item in symbolList" :value="item.symbol" :key="item.symbol">{{ item.symbol }}</Option>
                </Select>
                </Col>
                <Col :md='8' :lg='6'>
                <label>{{$t('common.yhm')}}：</label>
                <Input v-model.trim="formData.username" clearable style="width: 154px;"
                :placeholder="$t('common.qsryhm')"></Input>
                </Col>
                <Col :md='8' :lg='6'>
                <label>{{$t('finance.hzid')}}：</label>
                <Input v-model.trim="formData.transferId" clearable style="width: 154px;"
                :placeholder="$t('finance.hzid')"></Input>
                </Col>
                <Col :lg='12' :md='16'>
                <label>{{$t('common.cjsj')}}：</label>
                <DatePicker type="datetime" v-model="formData.createdStart" :placeholder="$t('common.kssj')"
                format="yyyy-MM-dd HH:mm:ss"
                style="width: 154px;"></DatePicker>
                <DatePicker type="datetime" v-model="formData.createdEnd" :placeholder="$t('common.jssj')"
                format="yyyy-MM-dd HH:mm:ss"
                style="width: 154px;"></DatePicker>
                </Col>
                <Col :md='8' :lg='6'>
                <label>{{$t('common.sl')}}：</label>
                <Select v-model="amount" style="width: 154px" :clearable="true">
                    <Option value="0">{{$t('common.qb')}}</Option>
                    <Option value="1">{{$t('common.xy1')}}</Option>
                    <Option value="2">{{$t('common.dy1xy1000')}}</Option>
                    <Option value="3">{{$t('common.dy1000xy10000')}}</Option>
                    <Option value="4">{{$t('common.dy10000')}}</Option>
                </Select>
                </Col>
                <Col :md='3' :lg='3'>
                <Button type="primary" @click="curPage=1;getfindUser()">{{$t('common.cx')}}</Button>
                </Col>
            </row>
            <Table ref="test2" :columns="columns" :data="data"></Table>
            <Page :current="curPage" :total="total" @on-change="changePage"
                  :page-size="size" style="text-align:center;margin-top:20px;"></Page>
        </Card>
    </Row>
</template>

<script>
    import financeApi from '../../api/finance';
    import currenyApi from '../../api/currency';
    import util from '../../libs/util';

    export default {
        data () {
            return {
                curPage: 1,
                total: 0,
                curPage1: 1,
                total1: 0,
                size: 10,
                columns: [
                    {title: this.$t('common.cjsj'), key: 'createdAt'},
                    {title: this.$t('finance.hzid'), key: 'transferId'},
                    {title: this.$t('common.yhm'), key: 'username'},
                    {title: this.$t('common.bz'), key: 'symbol'},
                    {title: this.$t('exchange.czsl'), key: 'quantity'},
                    {
                        title: this.$t('common.zt'), key: 'status',
                        render: (h, params) => {
                            return h('div', this.state('' + params.row.status));
                        }
                    }
                ],
                columns1: [
                    {title: this.$t('common.bz'), key: 'symbol'},
                    {title: this.$t('finance.jrczyhs'), key: 'currentRechargeUserCount'},
                    {title: this.$t('finance.jrczbs'), key: 'currentRechargeCount'},
                    {title: this.$t('finance.jryczsl'), key: 'currentRechargeSum'},
                    {title: this.$t('finance.qrnczyhs'), key: 'rechargeUserCount7d'},
                    {title: this.$t('finance.qrnczbs'), key: 'rechargeCount7d'},
                    {title: this.$t('finance.qrnyczsl'), key: 'rechargeSum7d'},
                ],
                data: [],
                data1: [],
                formData: {
                    username: '',
                    max: null,
                    min: null,
                    createdStart: null,
                    createdEnd: null,
                    symbol: '0'
                },
                amount: '0',
                symbolList:[],
                downloadparmes: {}
            };
        },
        created () {
            this.getdataSymbol();
            this.getfindUser();
            this.getStatisticList();
        },
        methods: {
            getdataSymbol () {
                currenyApi.findAllValidSymbolList((res) => {
                    this.symbolList = res;
                });
            },
            getStatisticList () {
                let data = {
                    page: this.curPage1,
                    size: this.size
                };
                financeApi.statisticList(data, (res, total) => {
                    this.data1 = res;
                    this.total1 = total;
                }, (msg) => {
                    this.$Message.error({content: msg});
                });
            },
            getfindUser () {
                switch (this.amount) {
                    case '1':
                        this.formData.min = null;
                        this.formData.max = 1;
                        break;
                    case '2':
                        this.formData.max = 1000;
                        this.formData.min = 1;
                        break;
                    case '3':
                        this.formData.max = 10000;
                        this.formData.min = 1000;
                        break;
                    case '4':
                        this.formData.min = 10000;
                        this.formData.max = null;
                        break;
                    case '0':
                        this.formData.max = null;
                        this.formData.min = null;
                        break;
                }
                this.formData.page = this.curPage;
                this.formData.size = this.size;
                let D = JSON.stringify(this.formData);
                let data = JSON.parse(D);
                data.createdStart = data.createdStart ? util.dateToStr(new Date(data.createdStart)) : null;
                data.createdEnd = data.createdEnd ? util.dateToStr(new Date(data.createdEnd)) : null;
                data.symbol = data.symbol === '0' ? null : data.symbol;
                this.downloadparmes = JSON.parse(JSON.stringify(data));
                financeApi.virtualRechargeRecordList(data, (res, total) => {
                    this.total = total;
                    this.data = res;
                });
            },
            state (i) {
                switch (i) { // 1：进行中，2：已完成，3：已取消，4：已拒绝
                    case '1':
                        return this.$t('common.jxz');
                    case '2':
                        return this.$t('common.ywc');
                    case '3':
                        return this.$t('common.yqx');
                    case '4':
                        return this.$t('common.yjj');
                }
            },
            changePage (page) {
                this.curPage = page;
                this.getfindUser();
            },
            changePage1 (page) {
                this.curPage1 = page;
                this.getfindUser();
            },
            downloadList() {
                let arr = []
                for (let i in this.downloadparmes){
                    if(this.downloadparmes[i]!== null && this.downloadparmes[i]!== ''){
                        let v = this.downloadparmes[i]
                        arr.push(i+'='+v)
                    }
                }
                console.log(arr)
                window.location.href = `${util.baseURL}api/bm/account/transfer/virtualRechargeRecordExport?export=1&${arr.join('&')}`
            },
        }
    };
</script>

<style scoped lang='less'>
.selWp{
    margin-bottom: 10px;
    label{
        display: inline-block;
        width: 70px;
    }
}
.selWp>div{
    margin-bottom: 10px;
}
</style>