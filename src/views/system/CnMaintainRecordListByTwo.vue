<template>
  <a-card :bordered="false" >
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
<!--      <a-form layout="inline" @keyup.enter.native="searchQuery">-->
<!--        <a-row :gutter="24">-->

<!--          <a-col :xl="10" :lg="11" :md="12" :sm="24">-->
<!--            <a-form-item label="日期">-->
<!--              <j-date placeholder="请选择日期" class="query-group-cust" dateFormat="YYYY-MM-DD" v-model="printDate"></j-date>-->
<!--            </a-form-item>-->
<!--          </a-col>-->
<!--          <a-col :xl="6" :lg="7" :md="8" :sm="24">-->
<!--            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">-->
<!--              <a-button type="primary" @click="getList()" icon="search">查询</a-button>-->
<!--            </span>-->
<!--          </a-col>-->

<!--        </a-row>-->
<!--      </a-form>-->
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator" style="text-align: center">
      <h1 style="font-size: 30px">产能报表</h1>
<!--      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>-->
<!--      <a-button type="primary" icon="download" @click="handleExportXls('产能维护')">导出</a-button>-->
<!--      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">-->
<!--        <a-button type="primary" icon="import">导入</a-button>-->
<!--      </a-upload>-->
<!--      &lt;!&ndash; 高级查询区域 &ndash;&gt;-->
<!--      <j-super-query :fieldList="superFieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query>-->
<!--      <a-dropdown v-if="selectedRowKeys.length > 0">-->
<!--        <a-menu slot="overlay">-->
<!--          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>-->
<!--        </a-menu>-->
<!--        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>-->
<!--      </a-dropdown>-->

<!--      {{day}}-->
    </div>

    <div class="table-operator" style="text-align: left;display: flex;justify-content: space-between">
      <h1 style="font-size: 25px">班次：{{day}}</h1>
      <h1 style="font-size: 25px;color: black;text-align: end">{{time}}</h1>
    </div>


    <!-- table区域-begin -->
    <div>

      <a-table
        ref="table"
        size="middle"
        :scroll="{x:true}"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="false"
        :loading="loading"
        class="j-table-force-nowrap"
        @change="handleTableChange">

        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img v-else :src="getImgView(text)" height="25px" alt="" style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无文件</span>
          <a-button
            v-else
            :ghost="true"
            type="primary"
            icon="download"
            size="small"
            @click="downloadFile(text)">
            下载
          </a-button>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a @click="handleDetail(record)">详情</a>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>

    <cn-maintain-modal ref="modalForm" @ok="modalFormOk"></cn-maintain-modal>
  </a-card>
</template>

<script>
  import { getAction, deleteAction, putAction, postAction, httpAction } from '@/api/manage'
  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import CnMaintainModal from './modules/CnMaintainModal'
  import moment from "moment";

  export default {
    name: 'CnMaintainRecordListByTwo',
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      CnMaintainModal
    },
    data () {
      return {
        time:undefined,
        printDate:"",
        day:"",
        description: '产能报表',
        // 表头
        columns: [
          {
            title:'项目',
            align:"center",
            dataIndex: 'name',
          },
          {
            title:'一号线',
            align:"center",
            dataIndex: 'sjcn1',
          },
          {
            title:'二号线',
            align:"center",
            dataIndex: 'sjcn2'
          },
          {
            title:'三号线',
            align:"center",
            dataIndex: 'sjcn3'
          },
          {
            title:'四号线',
            align:"center",
            dataIndex: 'sjcn4'
          },
          {
            title:'五号线',
            align:"center",
            dataIndex: 'sjcn5'
          },
          {
            title:'六号线',
            align:"center",
            dataIndex: 'sjcn6'
          },

        ],
        url: {
          list: "/system/cnMaintain/SHlistTHree",
          delete: "/system/cnMaintain/delete",
          deleteBatch: "/system/cnMaintain/deleteBatch",
          exportXlsUrl: "/system/cnMaintain/exportXls",
          importExcelUrl: "system/cnMaintain/importExcel",

        },
        dictOptions:{},
        superFieldList:[],
      }
    },
    created() {
      this.getList();
      this.detailtime();
    this.detail();
    },
    computed: {

    },
    methods: {
      detailtime () {
        setInterval(() => {
          this.time = moment().format('YYYY-MM-DD HH:mm')
        }, 1000)
      },
      getList(){
        getAction(this.url.list, {
          'printDate': this.printDate,
        }).then(res => {
          this.dataSource = res.result.cnMaintainResultVo;
          this.day = res.result.day;
        })
      },
      detail () {
        setInterval(() => {
          this.getList()
        }, 90000)
      },

      loadData(){},
      initDictConfig(){
      },
      getSuperFieldList(){
        let fieldList=[];
        fieldList.push({type:'string',value:'line',text:'产线名称',dictCode:''})
        fieldList.push({type:'string',value:'cn',text:'实际产能',dictCode:''})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';


</style>

<style  lang="less"  scoped>
  /deep/ .ant-table-tbody > tr > td {
    font-size: 20px;
  }
  /deep/ .ant-table-thead > tr > th {
    font-size: 25px;
  }
  </style>